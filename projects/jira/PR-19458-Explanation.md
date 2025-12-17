# PR #19458: Fix Authorization in AgencyHuman GraphQL Queries - Explanation

## Overview

This PR fixes how **authorization** (permission checking) works in GraphQL queries that fetch AgencyHuman relationships. The main change is making authorization more **explicit** and getting **better error messages** when users don't have permission.

---

## What is Authorization?

**Authorization** = "Does this user have permission to do this action?"

Think of it like a bouncer at a club:
- They check your ID (authentication - "who are you?")
- They check if you're allowed in (authorization - "are you allowed?")

In this codebase:
- **Authentication**: "Is this a valid user?" (already logged in)
- **Authorization**: "Can this user view this person's relationships?" (permission check)

---

## The Problem (Before the PR)

### Old Code Approach:
```ruby
agency_human = find_resource(::AgencyHuman, agency_human_id)
```

**What `find_resource` does:**
- Finds the record AND checks permissions in one step
- If the user doesn't have permission, it raises a generic error: `"AgencyHuman not found"`
- This is **confusing** because:
  - The record might actually exist in the database
  - But the error says "not found" (misleading!)
  - You can't tell if it's a permission issue or if the record truly doesn't exist

**Example Problem:**
- User tries to view relationships for person ID 123
- Person 123 exists in the database
- User doesn't have permission to view person 123
- Error: "AgencyHuman not found" ❌ (confusing - the person DOES exist!)

---

## The Solution (After the PR)

### New Code Approach:
```ruby
# Don't use find_resource here because we're auth'ing the agency
# human directly below
agency_human = ::AgencyHuman.find(agency_human_id)

authorize(agency_human, :view_relationships?)
```

**What this does:**
1. **`.find(agency_human_id)`** - Finds the record (no permission check yet)
   - If record doesn't exist → raises `ActiveRecord::RecordNotFound` (clear error!)
   - If record exists → continues to step 2

2. **`authorize(agency_human, :view_relationships?)`** - Checks permission explicitly
   - If user has permission → continues
   - If user doesn't have permission → raises `Pundit::NotAuthorizedError` (clear error!)

**Why this is better:**
- **Clear error messages**: You know exactly what went wrong
  - "Record not found" = the person doesn't exist
  - "Not authorized" = the person exists but you can't see it
- **Separation of concerns**: Finding the record is separate from checking permissions
- **Easier debugging**: You can tell if it's a data issue or a permission issue

---

## Key Ruby/Rails Concepts Explained

### 1. `find_resource` vs `.find`

**`find_resource` (old way):**
```ruby
agency_human = find_resource(::AgencyHuman, agency_human_id)
```
- Custom method that combines finding + authorization
- Returns the record OR raises "not found" error (even if it's a permission issue)
- Less clear what went wrong

**`.find` (new way):**
```ruby
agency_human = ::AgencyHuman.find(agency_human_id)
```
- Standard Rails method
- Only finds the record (no permission check)
- Raises `ActiveRecord::RecordNotFound` if record doesn't exist
- Clear and explicit

### 2. `authorize` - Pundit Authorization

```ruby
authorize(agency_human, :view_relationships?)
```

**What this does:**
- Uses the **Pundit** gem (authorization library)
- Checks if the current user can perform `:view_relationships?` on this `agency_human`
- Looks for a method in `AgencyHumanPolicy` class:
  ```ruby
  class AgencyHumanPolicy
    def view_relationships?
      # permission logic here
      user.can_view_relationships?(agency_human)
    end
  end
  ```

**If permission denied:**
- Raises `Pundit::NotAuthorizedError`
- This is a **specific error** that clearly indicates a permission problem

### 3. `::AgencyHuman` - Namespace Resolution

```ruby
::AgencyHuman.find(...)
```

- The `::` at the start means "start from the root namespace"
- Ensures we're using the `AgencyHuman` class from the global scope
- Prevents confusion if there's a local variable or method with the same name

---

## Changes in Detail

### Change 1: `agency_human_relationships.rb`

**Before:**
```ruby
query do |agency_human_id:, ...|
  agency_human = find_resource(::AgencyHuman, agency_human_id)
  # ... rest of query
end
```

**After:**
```ruby
query do |agency_human_id:, ...|
  # Don't use find_resource here because we're auth'ing the agency
  # human directly below
  agency_human = ::AgencyHuman.find(agency_human_id)
  
  authorize(agency_human, :view_relationships?)
  # ... rest of query
end
```

**What changed:**
- Separated finding the record from checking permissions
- Made authorization explicit with `authorize` call
- Added comment explaining why we don't use `find_resource`

### Change 2: `related_agency_humans.rb`

**Before:**
```ruby
query do |agency_human_id:, ...|
  authorized_via_policy_scope  # Old authorization method
  
  agency_human = find_resource(::AgencyHuman, agency_human_id)
  # ... rest of query
end
```

**After:**
```ruby
query do |agency_human_id:, ...|
  agency_human = ::AgencyHuman.find(agency_human_id)
  
  authorize(agency_human, :view_relationships?)
  # ... rest of query
end
```

**What changed:**
- Removed `authorized_via_policy_scope` (old way)
- Replaced `find_resource` with explicit `.find` + `authorize`
- Same pattern as Change 1 for consistency

---

## Test Changes Explained

### Change 1: `agency_human_relationships_spec.rb`

**Before:**
```ruby
it("rejects as the policy_scope fails to find the agency human") do
  expect(execute_query.to_h["errors"][0]["message"]).to(
    eq("AgencyHuman not found"),
  )
end
```
- Test expected a generic "not found" error message
- This was misleading because the record might exist

**After:**
```ruby
it(
  "raises a Pundit::NotAuthorizedError because the user " \
    "does not have permission to view the agency human",
) { expect { execute_query }.to(raise_error(Pundit::NotAuthorizedError)) }
```
- Test now expects the specific `Pundit::NotAuthorizedError`
- Clearer test description explaining it's a permission issue
- More accurate - this is what actually happens now

### Change 2: `related_agency_humans_spec.rb`

**Before:**
```ruby
it("raises an error because the keystone AH cannot be found") do
  expect(
    graphql_execute(...).to_h["errors"][0]["message"],
  ).to(eq("AgencyHuman not found"))
end
```
- Similar issue - expected "not found" error
- Test description was misleading

**After:**
```ruby
it(
  "raises a Pundit::NotAuthorizedError because the user " \
    "does not have permission to view the agency human",
) do
  expect do
    graphql_execute(...)
  end.to(raise_error(Pundit::NotAuthorizedError))
end
```
- Now expects the correct authorization error
- Clearer test description
- Uses `raise_error` matcher (more standard RSpec pattern)

---

## Why This Matters

### 1. Better Error Messages
**Before:**
```
Error: "AgencyHuman not found"
```
- User thinks: "Maybe I typed the wrong ID?"
- Developer thinks: "Does the record exist or is it permissions?"

**After:**
```
Error: Pundit::NotAuthorizedError
```
- User thinks: "I don't have permission to see this"
- Developer thinks: "This is a permission issue, not a data issue"

### 2. Easier Debugging
- Can distinguish between:
  - **Data problem**: Record doesn't exist → `ActiveRecord::RecordNotFound`
  - **Permission problem**: Record exists but no access → `Pundit::NotAuthorizedError`

### 3. More Explicit Code
- Clear separation: find the record, then check permissions
- Easier to understand what's happening
- Follows the principle: "Make it obvious"

### 4. Consistency
- Both queries now use the same authorization pattern
- Easier to maintain and understand

---

## Connection to ENG-24262

This PR is **related** to our blank screen issue because:

1. **Error Handling**: Better error messages help identify when issues are permission-related vs data-related
2. **Data Integrity**: Explicit authorization ensures we only return data the user is allowed to see
3. **Debugging**: Clear errors make it easier to diagnose issues like blank screens

However, this PR doesn't directly fix the null data issue - it's more about improving authorization patterns and error messages.

---

## Summary

**What changed:**
- Replaced `find_resource` with explicit `.find` + `authorize`
- Changed error from generic "not found" to specific `Pundit::NotAuthorizedError`
- Updated tests to expect the correct error type

**Why it matters:**
- Clearer error messages (permission vs data issues)
- Easier debugging
- More explicit and maintainable code
- Better user experience (clearer error messages)

**Ruby/Rails concepts learned:**
- `find_resource` vs `.find` - different ways to find records
- `authorize` - explicit permission checking with Pundit
- `Pundit::NotAuthorizedError` - specific error for permission issues
- `::` namespace resolution - ensuring correct class reference
- RSpec `raise_error` matcher - testing for exceptions

---

## Key Takeaway

**The main principle**: Separate concerns
- **Finding data** (does the record exist?) is separate from
- **Checking permissions** (can the user see it?)

This makes code clearer, errors more specific, and debugging easier.
