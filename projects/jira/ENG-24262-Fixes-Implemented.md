# ENG-24262: Fixes to Prevent Null Data Creation

## Summary

Implemented comprehensive fixes to prevent null/incomplete data from being created when linking potential connections via PKOS. These changes address the root cause by adding validation at multiple layers.

## Changes Implemented

### 1. Validation in CreateOrUpdateAgencyHuman Mutation

**File**: `app/graphql/mutations/create_or_update_agency_human.rb`

**Changes**:
- Added validation to ensure agency human has at least a first name or last name before creating relationships
- Prevents creating relationships with incomplete agency humans from PKOS
- Returns a clear error message if validation fails

**Code**:
```ruby
# Reload to ensure we have the latest data after creation/update
agency_human.reload

# Validate that agency human has required fields before creating relationships
# This prevents creating relationships with incomplete agency humans from PKOS
unless child_agency_humans.empty?
  # Ensure agency human has at least a first name or last name for display
  if agency_human.first_name.blank? && agency_human.last_name.blank?
    skip_authorization
    return(
      {
        errors:
          generate_errors(
            "Agency human must have at least a first name or last name before creating relationships",
          ),
      }
    )
  end
  # ... create relationships
end
```

### 2. Preserve Critical Fields in FilterValidAttributes

**File**: `app/lib/services/agency_humans/filter_valid_attributes.rb`

**Changes**:
- Added `CRITICAL_FIELDS` constant for fields that should never be filtered out
- Always includes `first_name` and `last_name` if provided, even if they cause validation errors
- Prevents creating agency humans without names which would cause blank pages

**Code**:
```ruby
# Critical fields that should never be filtered out, as they are required
# for relationships to function properly (e.g., displaying names in UI)
CRITICAL_FIELDS = %i[first_name last_name].freeze

# Always include critical fields if they were provided, even if they cause
# validation errors. This prevents creating agency humans without names
# which would cause blank pages when viewing relationships.
CRITICAL_FIELDS.each do |critical_field|
  if attrs.key?(critical_field) && attrs[critical_field].present?
    valid_attrs[critical_field] = attrs[critical_field]
  end
end
```

### 3. Validation in Relationship Creation Service

**File**: `app/lib/services/relationships/create.rb`

**Changes**:
- Added validation to ensure both agency humans exist before creating relationship
- Raises `ActiveRecord::RecordNotFound` with clear error messages if agency humans are missing
- Prevents creating relationships with null or missing agency humans

**Code**:
```ruby
def find_or_create_relationship(agency_human_id_1, agency_human_id_2)
  # Validate that both agency humans exist before creating relationship
  # This prevents creating relationships with null or missing agency humans
  agency_human_1 = AgencyHuman.find_by(id: agency_human_id_1)
  agency_human_2 = AgencyHuman.find_by(id: agency_human_id_2)

  if agency_human_1.nil?
    raise(
      ActiveRecord::RecordNotFound,
      "Source agency human with ID #{agency_human_id_1} not found",
    )
  end

  if agency_human_2.nil?
    raise(
      ActiveRecord::RecordNotFound,
      "Destination agency human with ID #{agency_human_id_2} not found",
    )
  end
  # ... create relationship
end
```

### 4. GraphQL Type Made Nullable

**File**: `app/graphql/types/relationship.rb`

**Changes**:
- Changed `source_agency_human` and `destination_agency_human` fields from `null: false` to `null: true`
- Added custom resolvers (though they currently just return the association)
- Allows GraphQL to return null values gracefully instead of raising errors
- Frontend already has defensive code to handle null values

**Code**:
```ruby
field(:source_agency_human, Types::AgencyHuman, null: true)
field(:destination_agency_human, Types::AgencyHuman, null: true)

# Custom resolvers to handle null agency humans gracefully
def source_agency_human
  object.source_agency_human
end

def destination_agency_human
  object.destination_agency_human
end
```

### 5. Frontend Defensive Fixes (Already Implemented)

**Files**:
- `app/javascript/components/family_finding/relationships/sortRelationships.js`
- `app/javascript/components/family_finding/relationships/relationshipsColumns.js`
- `app/javascript/components/family_finding/relationships/RelationshipActions.js`

**Purpose**: These changes handle existing bad data gracefully, preventing crashes when null values are encountered.

## Defense in Depth Strategy

These fixes implement a **defense in depth** approach:

1. **Prevention Layer**: `FilterValidAttributes` ensures critical fields are never filtered out
2. **Validation Layer**: `CreateOrUpdateAgencyHuman` mutation validates required fields before creating relationships
3. **Service Layer**: `Services::Relationships::Create` validates agency humans exist before creating relationships
4. **GraphQL Layer**: Relationship type allows null values and handles them gracefully
5. **Frontend Layer**: Components handle null values defensively (already implemented)

## Testing Recommendations

1. **Test PKOS creation with missing names**:
   - Try to link a connection with only invalid data
   - Should fail with clear error message before creating relationship

2. **Test PKOS creation with partial names**:
   - Try to link a connection with only first_name or only last_name
   - Should succeed and create relationship

3. **Test relationship creation with missing agency humans**:
   - Should raise `ActiveRecord::RecordNotFound` with clear error

4. **Test existing relationships with null agency humans**:
   - Frontend should handle gracefully (already tested)

## Impact

- **Prevents new bad data**: New relationships cannot be created with incomplete agency humans
- **Clear error messages**: Users get helpful error messages instead of silent failures
- **Backward compatible**: Existing bad data is handled gracefully by frontend
- **No breaking changes**: GraphQL schema change is backward compatible (fields are now nullable)

## Files Modified

1. `app/graphql/mutations/create_or_update_agency_human.rb` - Added validation
2. `app/lib/services/agency_humans/filter_valid_attributes.rb` - Preserve critical fields
3. `app/lib/services/relationships/create.rb` - Validate agency humans exist
4. `app/graphql/types/relationship.rb` - Made fields nullable
5. `app/javascript/components/family_finding/relationships/sortRelationships.js` - Defensive handling
6. `app/javascript/components/family_finding/relationships/relationshipsColumns.js` - Defensive handling
7. `app/javascript/components/family_finding/relationships/RelationshipActions.js` - Defensive handling

## Next Steps

1. Run tests to ensure all changes work correctly
2. Test PKOS flow end-to-end with various data scenarios
3. Monitor for any new errors in production
4. Consider adding database-level constraints if needed


