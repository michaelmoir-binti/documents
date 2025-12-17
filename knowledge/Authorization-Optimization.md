# Authorization Optimization: Early Checks and Fail-Fast Patterns

## Your Excellent Observation

You're absolutely right! The current pattern of "fetch first, then authorize" is **not optimal** in many cases. Let's explore how to optimize it.

---

## The Current Pattern (Suboptimal)

```ruby
# Current approach
agency_human = AgencyHuman.find(agency_human_id)  # ← Fetches full record
authorize(agency_human, :view_relationships?)      # ← Then checks permission
```

**Problems:**
1. **Performance**: Fetches full record even if user will be denied
2. **Security**: Touches sensitive data before checking permission
3. **Wasteful**: Loads related data (addresses, relationships) that might not be needed

---

## The Optimized Pattern (Your Suggestion)

### Pattern 1: Role-Based Early Rejection

```ruby
# Early check: Is user a visitor? Reject immediately
return raise Pundit::NotAuthorizedError if user.visitor?

# Early check: Is user an admin? Allow immediately
return agency_human if user.admin?

# Otherwise, do detailed check
authorize(agency_human, :view_relationships?)
```

**Benefits:**
- ✅ **Fast rejection** for visitors (no database query)
- ✅ **Fast approval** for admins (no permission logic needed)
- ✅ **Only detailed checks** for nurses (the complex case)

### Pattern 2: Lightweight Pre-Check

```ruby
# Lightweight check: Just get the agency_id (not full record)
agency_id = AgencyHuman.where(id: agency_human_id).pluck(:agency_id).first

# Early rejection: Different agency?
return raise Pundit::NotAuthorizedError if agency_id != user.agency_id

# If same agency, now fetch full record
agency_human = AgencyHuman.find(agency_human_id)
authorize(agency_human, :view_relationships?)
```

**Benefits:**
- ✅ **Cheap query**: Only fetches `agency_id` (one column)
- ✅ **Early rejection**: Fails fast if different agency
- ✅ **Full fetch only when needed**: Only loads full record if likely authorized

---

## Implementation Examples

### Example 1: Optimized AgencyHuman Query

```ruby
class AgencyHumanRelationships < BaseQuery
  query do |agency_human_id:, ...|
    # Step 1: Early role-based checks
    if user.visitor?
      raise Pundit::NotAuthorizedError, "Visitors cannot view relationships"
    end
    
    if user.admin?
      # Admins can see everything - skip detailed checks
      return AgencyHuman.find(agency_human_id)
    end
    
    # Step 2: Lightweight pre-check (for workers/nurses)
    agency_id = AgencyHuman
      .where(id: agency_human_id)
      .pluck(:agency_id)
      .first
    
    unless agency_id
      raise ActiveRecord::RecordNotFound, "AgencyHuman not found"
    end
    
    # Step 3: Early rejection if different agency
    if agency_id != user.agency_id
      raise Pundit::NotAuthorizedError, 
        "Cannot view relationships for different agency"
    end
    
    # Step 4: Now safe to fetch full record (likely authorized)
    agency_human = AgencyHuman.find(agency_human_id)
    
    # Step 5: Final detailed check (for edge cases)
    authorize(agency_human, :view_relationships?)
    
    # Return data
    agency_human
  end
end
```

### Example 2: Using Scopes for Early Filtering

```ruby
class AgencyHumanRelationships < BaseQuery
  query do |agency_human_id:, ...|
    # Step 1: Role-based early checks
    return early_admin_check(agency_human_id) if user.admin?
    return early_rejection if user.visitor?
    
    # Step 2: Use scope to filter early
    # This scope only returns records the user can potentially see
    agency_human = AgencyHuman
      .accessible_by_user(user)  # ← Custom scope
      .find_by(id: agency_human_id)
    
    unless agency_human
      # Could be "not found" OR "not authorized"
      # But we already filtered, so it's likely "not authorized"
      raise Pundit::NotAuthorizedError
    end
    
    # Step 3: Final check for specific permission
    authorize(agency_human, :view_relationships?)
    
    agency_human
  end
  
  private
  
  def early_admin_check(id)
    AgencyHuman.find(id)  # Admins can see everything
  end
  
  def early_rejection
    raise Pundit::NotAuthorizedError, "Visitors cannot view relationships"
  end
end
```

### Example 3: Database-Level Filtering

```ruby
# In AgencyHuman model
scope :accessible_by_user, ->(user) {
  if user.admin?
    all  # Admins see everything
  elsif user.worker?
    where(agency_id: user.agency_id)  # Workers see their agency only
  else
    none  # Visitors see nothing
  end
}

# In query
query do |agency_human_id:, ...|
  agency_human = AgencyHuman
    .accessible_by_user(current_user)  # ← Filtered at DB level
    .find_by(id: agency_human_id)
  
  unless agency_human
    raise Pundit::NotAuthorizedError
  end
  
  authorize(agency_human, :view_relationships?)
  agency_human
end
```

**Benefits:**
- ✅ **Database does the filtering** (very efficient)
- ✅ **Only returns records user can see**
- ✅ **No unnecessary data loaded**

---

## Performance Comparison

### Current Approach (Fetch First)

```ruby
# Query 1: Fetch full record with all associations
agency_human = AgencyHuman
  .includes(:addresses, :phone_numbers, :relationships)
  .find(agency_human_id)
# Time: ~50ms, Loads: Full record + all associations

# Then check permission
authorize(agency_human, :view_relationships?)
# Time: ~5ms

# Total: ~55ms, even if user will be denied
```

### Optimized Approach (Early Checks)

```ruby
# Early check: Role-based
return if user.visitor?  # Time: ~0.1ms (no DB query!)

# Lightweight check: Just agency_id
agency_id = AgencyHuman
  .where(id: agency_human_id)
  .pluck(:agency_id)
  .first
# Time: ~5ms, Loads: Just one column

# Early rejection if different agency
return if agency_id != user.agency_id  # Time: ~0.1ms

# Only now fetch full record (likely authorized)
agency_human = AgencyHuman.find(agency_human_id)
# Time: ~50ms

# Total: ~55ms if authorized, ~5ms if denied (10x faster rejection!)
```

**Performance Improvement:**
- **Authorized users**: Same speed (~55ms)
- **Denied users**: 10x faster (~5ms vs ~55ms)
- **Visitors**: Instant rejection (~0.1ms)

---

## Security Benefits

### Principle: Fail Fast, Fail Secure

**Current approach:**
```
1. Fetch full record (with sensitive data)
2. Check permission
3. If denied, data was already loaded in memory
```

**Risk:** Even though data isn't returned, it was:
- Loaded from database
- Parsed into objects
- Stored in memory (briefly)
- Could be logged or cached

**Optimized approach:**
```
1. Check role (visitor? → reject immediately)
2. Check agency_id (different? → reject immediately)
3. Only fetch full record if likely authorized
```

**Benefit:** Sensitive data never leaves the database for unauthorized users.

---

## When to Use Each Pattern

### Use Early Role Checks When:
- ✅ Roles have clear permission boundaries (admin, visitor, worker)
- ✅ Most users fall into one role category
- ✅ Role checks are cheap (no database query)

### Use Lightweight Pre-Checks When:
- ✅ You can determine authorization from a single field (like `agency_id`)
- ✅ That field is indexed (fast query)
- ✅ Full record is expensive to load (many associations)

### Use Full Fetch + Authorize When:
- ✅ Authorization logic is complex (many conditions)
- ✅ Need full record anyway (for the response)
- ✅ Pre-checks would be as expensive as full fetch

---

## Real-World Example: Hospital System

### Current Pattern (Inefficient)
```ruby
def view_patient_record(patient_id)
  # Fetch full record (expensive!)
  patient = Patient.find(patient_id)
  # Loads: name, SSN, medical history, prescriptions, etc.
  
  # Then check permission
  authorize(patient, :view?)
  # If visitor → denied, but we already loaded all the data!
end
```

### Optimized Pattern (Efficient)
```ruby
def view_patient_record(patient_id)
  # Early check 1: Role-based
  if current_user.visitor?
    raise Pundit::NotAuthorizedError
  end
  
  if current_user.admin?
    return Patient.find(patient_id)  # Admins see everything
  end
  
  # Early check 2: Lightweight (just department_id)
  department_id = Patient
    .where(id: patient_id)
    .pluck(:department_id)
    .first
  
  # Early rejection
  if department_id != current_user.department_id
    raise Pundit::NotAuthorizedError
  end
  
  # Now safe to fetch full record
  patient = Patient.find(patient_id)
  authorize(patient, :view?)  # Final check for edge cases
  
  patient
end
```

**Result:**
- **Visitors**: Instant rejection (no DB query)
- **Different department**: Fast rejection (cheap query)
- **Same department**: Full fetch only when likely authorized

---

## Implementation in This Codebase

### Suggested Refactor

```ruby
class AgencyHumanRelationships < BaseQuery
  query do |agency_human_id:, ...|
    # Step 1: Early role-based checks
    return early_rejection("Visitors cannot view relationships") if user.visitor?
    return early_admin_approval(agency_human_id) if user.admin?
    
    # Step 2: Lightweight pre-check
    agency_id = AgencyHuman
      .where(id: agency_human_id)
      .pluck(:agency_id)
      .first
    
    unless agency_id
      raise ActiveRecord::RecordNotFound, "AgencyHuman not found"
    end
    
    # Step 3: Early rejection if different agency
    if agency_id != user.agency_id
      raise Pundit::NotAuthorizedError,
        "Cannot view relationships for different agency"
    end
    
    # Step 4: Fetch full record (likely authorized)
    agency_human = ::AgencyHuman
      .includes(AgencyHuman::INCLUDES)
      .find(agency_human_id)
    
    # Step 5: Final detailed check (for complex rules)
    authorize(agency_human, :view_relationships?)
    
    # Return data
    agency_human
  end
  
  private
  
  def early_admin_approval(id)
    # Admins can see everything - no need for detailed checks
    ::AgencyHuman
      .includes(AgencyHuman::INCLUDES)
      .find(id)
  end
  
  def early_rejection(message)
    raise Pundit::NotAuthorizedError, message
  end
end
```

---

## Trade-offs to Consider

### Pros of Early Checks:
- ✅ **Faster rejection** for unauthorized users
- ✅ **Better security** (data never loaded)
- ✅ **Reduced database load** (fewer queries)
- ✅ **Better user experience** (faster error messages)

### Cons of Early Checks:
- ⚠️ **More complex code** (multiple checks)
- ⚠️ **Potential duplication** (checking same thing twice)
- ⚠️ **Maintenance burden** (more code paths)

### When NOT to Optimize:
- ❌ Authorization logic is very simple (one check)
- ❌ Full record is needed anyway (for response)
- ❌ Pre-checks are as expensive as full fetch
- ❌ Code complexity outweighs performance gain

---

## Best Practices

### 1. Layer Checks from Cheapest to Most Expensive

```ruby
# 1. Cheapest: Role check (in-memory)
return if user.visitor?

# 2. Cheap: Single column query
agency_id = AgencyHuman.where(id: id).pluck(:agency_id).first

# 3. Expensive: Full record with associations
agency_human = AgencyHuman.includes(...).find(id)
```

### 2. Use Database Indexes

```ruby
# Make sure agency_id is indexed for fast lookups
add_index :agency_humans, :agency_id
```

### 3. Cache Role Checks

```ruby
# Cache user role (doesn't change often)
def user_role
  @user_role ||= user.role
end
```

### 4. Measure Performance

```ruby
# Add timing to see if optimization helps
start_time = Time.now
# ... authorization checks ...
Rails.logger.info "Authorization took: #{Time.now - start_time}ms"
```

---

## Summary

**Your observation is correct!** The current "fetch first, then authorize" pattern can be optimized:

1. **Early role checks**: Reject visitors immediately (no DB query)
2. **Early admin approval**: Approve admins immediately (no detailed checks)
3. **Lightweight pre-checks**: Check `agency_id` before fetching full record
4. **Full fetch only when needed**: Only load expensive data if likely authorized

**Benefits:**
- 🚀 **Performance**: 10x faster rejection for unauthorized users
- 🔒 **Security**: Sensitive data never loaded for unauthorized users
- 💰 **Cost**: Reduced database load and memory usage

**Implementation:**
- Start with role-based checks (cheapest)
- Add lightweight pre-checks (single column queries)
- Only fetch full records when likely authorized
- Keep final authorization check for edge cases

This is a **best practice** pattern called "fail fast" or "defense in depth" - check early, check often, only load what you need!
