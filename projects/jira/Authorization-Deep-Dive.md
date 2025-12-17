# Authorization Deep Dive - Understanding Permission Checks

## Your Questions Answered

### 1. Is this to authorize every connection to the DB?
**No!** Authorization is **NOT** about database connections.

- **Database connections** = technical infrastructure (how the app talks to the database)
- **Authorization** = business logic (what data can this user see?)

Think of it like this:
- **Database connection** = "Can I talk to the database?" (technical - always yes if the app is running)
- **Authorization** = "Can this user see this person's data?" (business rule - depends on who the user is)

### 2. Is it meant to auth the actions the user can take on the DB?
**Not exactly!** It's about **what data the user can see**, not what actions they can perform.

Authorization in this context checks:
- ✅ "Can User A view Person X's relationships?"
- ✅ "Can User B see Person Y's details?"
- ❌ NOT "Can User A INSERT into the database?" (that's a different concern)

### 3. Must it be called on every query?
**No, but it should be called on queries that return sensitive data.**

- **Public data** (like public blog posts) → No authorization needed
- **Private data** (like person records, relationships) → Authorization required

In this codebase, authorization is needed for queries that return:
- Person/agency human data
- Relationships between people
- Family finding information
- Any data that's not public

---

## What is Authorization? (Simple Explanation)

**Authorization** = "Does this user have permission to see/do this?"

### Real-World Analogy

Imagine a hospital:

1. **Authentication** (Who are you?)
   - "Show me your ID badge"
   - "Are you a doctor, nurse, or visitor?"

2. **Authorization** (What can you see?)
   - Doctor: Can see all patient records
   - Nurse: Can see assigned patients only
   - Visitor: Can see no patient records

In our codebase:
- **Authentication**: "Is this a logged-in user?" (handled by login system)
- **Authorization**: "Can this user see this person's relationships?" (handled by `authorize`)

---

## How Authorization Works in This Codebase

### The Flow

```
User makes request
    ↓
GraphQL query runs
    ↓
Find the record (e.g., AgencyHuman.find(id))
    ↓
Check authorization (e.g., authorize(agency_human, :view_relationships?))
    ↓
    ├─ If authorized → Return data
    └─ If not authorized → Raise Pundit::NotAuthorizedError
```

### Example: Viewing Relationships

```ruby
# Step 1: Find the person (no permission check yet)
agency_human = ::AgencyHuman.find(agency_human_id)
# This just gets the record from the database
# Doesn't check if user can see it

# Step 2: Check if user has permission
authorize(agency_human, :view_relationships?)
# This checks: "Can the current user view relationships for this person?"
# 
# Behind the scenes, it calls:
# AgencyHumanPolicy.new(current_user, agency_human).view_relationships?
#
# If returns true → Continue
# If returns false → Raise Pundit::NotAuthorizedError

# Step 3: If authorized, return the data
return agency_human.relationships
```

---

## What Does `authorize` Actually Check?

### The Pundit Policy System

When you call:
```ruby
authorize(agency_human, :view_relationships?)
```

Pundit looks for a policy class and method:

```ruby
# app/policies/agency_human_policy.rb
class AgencyHumanPolicy
  def initialize(user, agency_human)
    @user = user
    @agency_human = agency_human
  end

  def view_relationships?
    # Business logic here:
    # - Is the user an admin? → Yes
    # - Is the user a worker at the same agency? → Yes
    # - Is the user a random person? → No
    
    return true if @user.admin?
    return true if @user.agency_id == @agency_human.agency_id
    return false
  end
end
```

### Real Example Logic

```ruby
def view_relationships?
  # Rule 1: Admins can see everything
  return true if user.binti_admin?
  
  # Rule 2: Workers can see people in their agency
  return true if user.agency_id == agency_human.agency_id
  
  # Rule 3: Workers can see people they're assigned to
  return true if user.assigned_to?(agency_human)
  
  # Rule 4: Otherwise, no access
  return false
end
```

---

## When is Authorization Needed?

### ✅ Queries That Need Authorization

```ruby
# Getting person details - SENSITIVE DATA
query AgencyHumanDetails($ids: [ID!]!) {
  agencyHumanDetails(ids: $ids) {
    fullName
    ssn              # ← Sensitive!
    addresses         # ← Sensitive!
    relationships     # ← Sensitive!
  }
}

# Getting relationships - SENSITIVE DATA
query AgencyHumanRelationships($agencyHumanId: ID!) {
  agencyHumanRelationships(agencyHumanId: $agencyHumanId) {
    # Relationships are private information
  }
}
```

**Why?** Because this data is:
- Private (not public)
- Regulated (HIPAA, child welfare laws)
- Agency-specific (workers shouldn't see other agencies' data)

### ❌ Queries That Don't Need Authorization

```ruby
# Public information - NO AUTHORIZATION NEEDED
query PublicAnnouncements {
  announcements(public: true) {
    title
    content
  }
}

# User's own data - NO AUTHORIZATION NEEDED (they're already authenticated)
query CurrentUser {
  currentUser {
    email
    name
  }
}
```

**Why?** Because:
- Public data is meant to be seen by everyone
- User's own data is always accessible to them

---

## Why Not Check Authorization in the Database?

### Database-Level vs Application-Level

**Database-level security:**
```sql
-- This is about technical access
GRANT SELECT ON agency_humans TO app_user;
-- "Can the application read from this table?"
```

**Application-level authorization:**
```ruby
# This is about business rules
authorize(agency_human, :view_relationships?)
# "Can THIS specific user see THIS specific person?"
```

### Why Application-Level is Better

1. **More Flexible**
   - Database: "Can read table?" (all or nothing)
   - Application: "Can see this specific record?" (per-record rules)

2. **Business Logic**
   - Database doesn't know about "agencies", "assignments", "roles"
   - Application knows: "Worker A can see Person X because they're in the same agency"

3. **Dynamic Rules**
   - Database rules are static
   - Application rules can change: "If it's after hours, only admins can see this"

---

## The Pattern in This PR

### Before (Implicit Authorization)

```ruby
agency_human = find_resource(::AgencyHuman, agency_human_id)
# find_resource does authorization internally
# But if it fails, error is: "AgencyHuman not found"
# Unclear if it's a data issue or permission issue
```

**Problem:**
- Authorization is hidden inside `find_resource`
- Error message is misleading
- Hard to debug permission issues

### After (Explicit Authorization)

```ruby
agency_human = ::AgencyHuman.find(agency_human_id)
# Step 1: Just get the record (no permission check)
# If record doesn't exist → ActiveRecord::RecordNotFound (clear!)

authorize(agency_human, :view_relationships?)
# Step 2: Explicitly check permission
# If no permission → Pundit::NotAuthorizedError (clear!)
```

**Benefits:**
- Clear separation: data vs permissions
- Specific error messages
- Easier to debug
- More maintainable

---

## Common Authorization Patterns

### Pattern 1: Check Before Returning Data

```ruby
query do |id:|
  record = Model.find(id)
  authorize(record, :show?)  # ← Check permission
  record                     # ← Return if authorized
end
```

### Pattern 2: Filter Results by Permission

```ruby
query do
  records = Model.all
  records.select do |record|
    policy(record).show?  # ← Check each one
  end
end
```

### Pattern 3: Check Permission for Actions

```ruby
mutation do |id:, data:|
  record = Model.find(id)
  authorize(record, :update?)  # ← Check permission to update
  record.update(data)          # ← Update if authorized
end
```

---

## Security Considerations

### Why Authorization Matters

**Without authorization:**
```ruby
# BAD - No permission check!
query do |id:|
  AgencyHuman.find(id)  # Anyone can see anyone's data!
end
```

**Attack scenario:**
1. User A (Worker at Agency 1) requests person ID 999
2. Person 999 belongs to Agency 2 (different agency)
3. Without authorization check → User A sees Agency 2's data ❌
4. **Privacy violation!**

**With authorization:**
```ruby
# GOOD - Permission check!
query do |id:|
  agency_human = AgencyHuman.find(id)
  authorize(agency_human, :show?)  # ← Blocks unauthorized access
  agency_human
end
```

**Same scenario:**
1. User A requests person ID 999
2. Person 999 belongs to Agency 2
3. Authorization check fails → `Pundit::NotAuthorizedError` ✅
4. **Privacy protected!**

---

## Summary: Your Questions Answered

### Q1: Is this to authorize every connection to the DB?
**A:** No. Authorization is about **what data users can see**, not database connections. Database connections are a technical infrastructure concern.

### Q2: Is it meant to auth the actions the user can take on the DB?
**A:** Not exactly. It's about **what data the user can view**, not database operations. The database operations (SELECT, INSERT, etc.) are handled by the application code, and authorization determines **which records** those operations can access.

### Q3: Must it be called on every query?
**A:** No, but it **should** be called on queries that return sensitive/private data. In this codebase, that means most queries involving people, relationships, and family finding data.

---

## Key Takeaways

1. **Authorization ≠ Database Access**
   - Authorization is about business rules (who can see what)
   - Database access is about technical infrastructure

2. **Authorization is Explicit**
   - The PR makes authorization explicit with `authorize()` calls
   - This makes code clearer and errors more specific

3. **Authorization Protects Privacy**
   - Without it, users could see data they shouldn't
   - With it, access is controlled by business rules

4. **Not Every Query Needs It**
   - Public data → No authorization
   - Private data → Authorization required

5. **It's About Data, Not Actions**
   - Checks: "Can user see this record?"
   - Not: "Can user perform database operations?"

---

## Real-World Example

**Scenario:** Worker at Agency A tries to view relationships for a person at Agency B

**Without proper authorization:**
```
Worker A: "Show me relationships for Person 123"
System: "Here's all the data!" ❌ (Privacy violation!)
```

**With proper authorization:**
```
Worker A: "Show me relationships for Person 123"
System: Checks permission...
System: "Pundit::NotAuthorizedError - You don't have permission" ✅ (Privacy protected!)
```

This is why authorization is critical in applications handling sensitive data like child welfare information.
