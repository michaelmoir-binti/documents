# PR #19400: Optimize AgencyHumanDetails GraphQL - Explanation

## Overview

This PR optimizes a GraphQL query that fetches AgencyHuman (person) details. The main goal was to make it **faster** by changing how permissions are checked.

---

## What is a GraphQL Query?

Think of a GraphQL query like asking a database: "Give me information about these specific people (by their IDs)". The query needs to:
1. Find the people in the database
2. Check if the current user has permission to see them
3. Return only the people the user is allowed to see

---

## The Problem (Before the PR)

### Old Code Approach:
```ruby
AgencyHumanPolicy::Scope
  .new(current_user, ::AgencyHuman)
  .resolve(filter_for_children:)
```

**What this does:**
- Uses a "scope" (a pre-built filtering system) to check permissions
- The scope checks permissions for ALL agency humans, then filters
- This is **slow** because it processes many records even when you only need one or two

**Why it's slow:**
- Imagine you have 10,000 people in the database
- You only want to see 1 person
- The old way checks permissions for all 10,000, then filters down to 1
- That's a lot of unnecessary work!

---

## The Solution (After the PR)

### New Code Approach:
```ruby
::AgencyHuman
  .includes(AgencyHuman::INCLUDES)  # Step 1: Load related data efficiently
  .find(ids)                        # Step 2: Find only the records we need
  .where(id: ids)                   # Step 3: Filter to just these IDs (redundant but safe)
  .select do |agency_human|         # Step 4: Manually check each one
    # Check permission for THIS specific person
    next false unless policy(agency_human).show?
    
    # If filtering for children, only return children
    next true unless filter_for_children
    agency_human.child?
  end
```

**What this does:**
1. **`.includes(...)`** - Pre-loads related data (like addresses, phone numbers) to avoid multiple database queries
2. **`.find(ids)`** - Gets ONLY the specific records we need (not all 10,000!)
3. **`.select`** - Manually checks each record:
   - Does the user have permission? (`policy(agency_human).show?`)
   - If filtering for children, is this person a child? (`agency_human.child?`)

**Why it's faster:**
- Only processes the records you actually need (usually just 1-2)
- Checks permissions one-by-one instead of using the slow scope
- Uses eager loading (`.includes`) to prevent multiple database queries

---

## Key Ruby Concepts Explained

### 1. `.includes()` - Eager Loading
```ruby
.includes(AgencyHuman::INCLUDES)
```
- **Problem it solves**: Without this, if you load a person and then access their addresses, Ruby makes separate database queries
- **Solution**: Loads all related data in one query
- **Example**: Instead of 5 queries (1 for person, 4 for addresses), it does 1 query

### 2. `.find(ids)` - Find Specific Records
```ruby
.find(ids)  # ids is an array like [123, 456]
```
- Gets records by their IDs
- Much faster than loading all records and filtering

### 3. `.select` - Filter Array Elements
```ruby
.select do |agency_human|
  # Return true to keep this element, false to remove it
  policy(agency_human).show?
end
```
- Goes through each record one by one
- Keeps records where the block returns `true`
- Removes records where the block returns `false`

### 4. `next false` / `next true` - Skip or Keep
```ruby
next false unless policy(agency_human).show?  # Skip this record if no permission
next true unless filter_for_children         # Keep this record if not filtering
```
- `next false` = "skip this record, don't include it"
- `next true` = "keep this record, include it"
- `unless` = "if NOT"

---

## Test Changes Explained

### Change 1: Error Handling

**Before:**
```ruby
it("rejects as unauthorized") do
  expect { execute_query }.to(raise_error(ActiveRecord::RecordNotFound))
end
```
- Expected the query to **crash** with an error if user has no permission

**After:**
```ruby
it("returns an empty array") do
  expect(execute_query["data"]["agencyHumanDetails"]).to(be_empty)
end
```
- Now expects an **empty array** instead of crashing
- This is better UX - no error, just "no results"

### Change 2: Variable Naming

**Before:**
```ruby
let(:zion) do
  create(:agency_human, :with_child, agency: family_finding_agency)
end
```

**After:**
```ruby
let(:child_agency_human) do
  create(:agency_human, :with_child, agency: family_finding_agency)
end
```
- Changed variable name from `zion` (a specific name) to `child_agency_human` (descriptive)
- Makes the test clearer - you know it's a child agency human, not just "zion"

### Change 3: Test Expectations

**Before:**
```ruby
it("raises an error") do
  expect { execute_query }.to(raise_error(ActiveRecord::RecordNotFound))
end
```

**After:**
```ruby
it("only includes child agency humans") do
  expect(execute_query["data"]["agencyHumanDetails"]).to(
    match([
      hash_including({
        id: child_agency_human.id.to_s,
        fullName: child_agency_human.full_name,
      }.stringify_keys),
    ])
  )
end
```
- Now tests that it **returns the correct data** instead of testing for an error
- Verifies that when filtering for children, only children are returned
- Uses `hash_including` to check that the result contains the expected fields

---

## Why This Matters for ENG-24262

This PR is **related** to our blank screen issue because:

1. **Performance**: Faster queries mean less chance of timeouts or incomplete data
2. **Error Handling**: The change from raising errors to returning empty arrays is similar to our null handling fixes
3. **Data Integrity**: The manual permission checking ensures we only return valid, accessible records

However, this PR doesn't directly fix the null data issue - it's more about performance and error handling patterns.

---

## Summary

**What changed:**
- Switched from slow scope-based permission checking to fast manual checking
- Changed error behavior from "crash" to "return empty array"
- Improved test clarity with better variable names

**Why it matters:**
- Much faster when querying 1-2 records (common case)
- Better user experience (no crashes, just empty results)
- More maintainable code (clearer what's happening)

**Ruby concepts learned:**
- `.includes()` for eager loading
- `.find()` for getting specific records
- `.select` for filtering arrays
- `next` for early returns in blocks
- `unless` for negative conditions
