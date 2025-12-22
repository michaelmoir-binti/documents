# ENG-24262: PKOS Creation Flow Investigation

## Investigation Summary

After reverting the agency_humans component changes (as they're not the root cause), I investigated the PKOS creation flow to identify where null data might be introduced when creating relationships from Potential Kin Online Search results.

## PKOS Creation Flow

### Path: `/family_finding/potential_kin_searches/new` → Link Connection → Create Relationship

1. **User searches for potential kin** via PKOS
2. **User clicks "Link Connection"** → Opens `LinkPotentialConnectionModal`
3. **User selects data to include** and clicks "Create New Potential Connections"
4. **Frontend calls `CreateOrUpdateAgencyHuman` mutation** with:
   - `skipInvalidDataErrors: true` (line 292 in `LinkPotentialConnectionModal.js`)
   - Selected addresses, phone numbers, email addresses
   - `childAgencyHumanIds` (the children to link to)
5. **Backend mutation** (`create_or_update_agency_human.rb`):
   - Calls `Services::AgencyHumans::Create.call` with `skip_invalid_data_errors: true`
   - If `skip_invalid_data_errors` is true, filters attributes using `FilterValidAttributes`
   - Creates agency human with only valid attributes
   - Creates relationships via `Services::Relationships::Create.call`

## Key Code Paths

### 1. Agency Human Creation with Invalid Data Skipping

**File**: `app/lib/services/agency_humans/create.rb`

```ruby
if skip_invalid_data_errors
  valid_attrs = Services::AgencyHumans::FilterValidAttributes.call(
    attrs: agency_human_attrs,
    agency: agency,
  ).body
else
  valid_attrs = agency_human_attrs
end

agency_human = AgencyHuman.create!(human: Human.new, agency: agency, **valid_attrs)
```

**File**: `app/lib/services/agency_humans/filter_valid_attributes.rb`

This service:
- Creates a test `AgencyHuman` with all attributes
- If invalid, iterates through each attribute individually
- Only includes attributes that don't cause validation errors
- **Result**: Agency human is created but may be missing some data

### 2. Relationship Creation

**File**: `app/graphql/mutations/create_or_update_agency_human.rb` (lines 171-187)

```ruby
unless child_agency_humans.empty?
  relationships_to_children =
    child_agency_humans.map do |cah|
      authorize(cah, :create_relationship?)
      Services::Relationships::Create.call(
        associated_child_agency_human_ids: [cah.id],
        agency_human_id_1: agency_human.id,  # The newly created agency human
        agency_human_id_2: cah.id,           # The child
        ...
      ).body[:relationship]
    end
end
```

**File**: `app/lib/services/relationships/create.rb`

```ruby
def find_or_create_relationship(agency_human_id_1, agency_human_id_2)
  relationship = Relationship.find_relationship_between_agency_humans(
    agency_human_id_1,
    agency_human_id_2,
  )
  if relationship.nil?
    agency_id = AgencyHuman.find_by(id: agency_human_id_1)&.agency_id
    relationship = Relationship.create!(
      data_owner_id: agency_id,
      source_agency_human_id: agency_human_id_1,
      destination_agency_human_id: agency_human_id_2,
    )
  end
  relationship
end
```

## Potential Issues

### Issue 1: Agency Human Created But Incomplete

**Scenario**: When `skipInvalidDataErrors: true` is used:
- Some attributes might be filtered out if they cause validation errors
- Agency human is created successfully but may be missing:
  - Name fields (firstName, lastName, middleName)
  - Other required fields that were invalid

**Impact**: 
- Agency human exists in database
- Relationship is created successfully
- But when queried later, the agency human might not have a `fullName` or `linkToView`
- This causes `determineName` to return undefined values

### Issue 2: Soft-Deleted Agency Humans

**Scenario**: 
- Agency human is created via PKOS
- Relationship is created
- Later, the agency human is soft-deleted (if using `acts_as_paranoid`)
- When relationship is queried, the association might return `null`

**Investigation Needed**:
- Does `AgencyHuman` use `acts_as_paranoid`?
- Does the relationship query use `with_deleted` scope?
- Can soft-deleted agency humans cause null associations?

### Issue 3: Authorization Filtering

**Scenario**:
- Agency human is created via PKOS
- Relationship is created
- When relationships are queried, `policy_scope` filters relationships
- If the agency human is not visible to the current user (due to authorization), the association might not be loaded

**File**: `app/services/relationships/compute_agency_human_relationships.rb`

```ruby
relationships.concat(
  policy_scope(
    agency_human.relationships_as_source_agency_human.includes(
      Relationship::INCLUDES_FOR_QUERIES,
    ),
  ),
)
```

**Investigation Needed**:
- Does `RelationshipPolicy` filter based on agency human visibility?
- Can authorization cause agency humans to be excluded from eager loading?
- Does `INCLUDES_FOR_QUERIES` respect authorization scopes?

### Issue 4: GraphQL Field Resolution

**File**: `app/graphql/types/relationship.rb`

```ruby
field(:source_agency_human, Types::AgencyHuman, null: false)
field(:destination_agency_human, Types::AgencyHuman, null: false)
```

**Issue**: 
- Fields are marked as `null: false` but no custom resolver
- Uses default ActiveRecord association
- If association returns `null`, GraphQL will still return `null` (despite `null: false` constraint)
- Frontend expects these fields to always be present

## Questions to Investigate

1. **Can an agency human be created successfully but have null `fullName`?**
   - Check if `fullName` is a required field
   - Check if `FilterValidAttributes` can filter out name fields
   - Check if PKOS data can have missing names

2. **Can relationships be created with null agency human associations?**
   - Check database foreign key constraints
   - Check if `Relationship.create!` can succeed with invalid IDs
   - Check if soft-deletes can cause null associations

3. **Can authorization cause agency humans to be null in relationships?**
   - Check `RelationshipPolicy` scope
   - Check if `policy_scope` filters out agency humans
   - Check if eager loading respects authorization

4. **What happens when GraphQL tries to resolve a null association?**
   - Does it return `null` despite `null: false`?
   - Does it raise an error?
   - How does the frontend handle this?

## Recommended Next Steps

1. **Check database for existing relationships with null agency humans**
   ```sql
   SELECT r.id, r.source_agency_human_id, r.destination_agency_human_id
   FROM relationships r
   LEFT JOIN agency_humans sah ON r.source_agency_human_id = sah.id
   LEFT JOIN agency_humans dah ON r.destination_agency_human_id = dah.id
   WHERE sah.id IS NULL OR dah.id IS NULL;
   ```

2. **Check for relationships where agency humans have null names**
   ```sql
   SELECT r.id, sah.full_name, dah.full_name
   FROM relationships r
   JOIN agency_humans sah ON r.source_agency_human_id = sah.id
   JOIN agency_humans dah ON r.destination_agency_human_id = dah.id
   WHERE sah.full_name IS NULL OR dah.full_name IS NULL;
   ```

3. **Test PKOS creation flow with invalid data**
   - Create a PKOS search with missing/invalid data
   - Link connection with `skipInvalidDataErrors: true`
   - Check what data is actually stored
   - Check if relationship is created successfully
   - Check if agency human has all required fields

4. **Add validation/defensive checks in relationship creation**
   - Ensure agency human has required fields before creating relationship
   - Add validation in `Services::Relationships::Create` to check agency humans exist
   - Add null checks in GraphQL resolvers

5. **Fix GraphQL type to handle null associations**
   - Add custom resolvers for `source_agency_human` and `destination_agency_human`
   - Handle null cases gracefully
   - Or change `null: false` to `null: true` and handle in frontend

## Current Defensive Fixes (Already Implemented)

The following files have been updated to handle null agency humans gracefully:

1. **`sortRelationships.js`**: 
   - `determineName` now checks for null `destinationAgencyHuman`/`sourceAgencyHuman`
   - Returns undefined values instead of crashing
   - `sortByName` handles null names in sorting

2. **`relationshipsColumns.js`**:
   - Handles null `name` from `determineName`
   - Uses empty string as fallback

3. **`RelationshipActions.js`**:
   - Handles null `name` and `keystoneName` in translation strings

These fixes prevent crashes but don't address the root cause of why null data is being stored.

## Conclusion

The root cause is likely in the **PKOS creation flow** where:
1. `skipInvalidDataErrors: true` allows incomplete agency humans to be created
2. Relationships are created even if agency humans are missing required data
3. When relationships are queried later, the incomplete data causes null associations or missing fields

**Recommended Fix**: 
- Add validation before creating relationships to ensure agency humans have required fields
- Add defensive checks in GraphQL resolvers
- Consider not using `skipInvalidDataErrors` for critical fields like names
- Add database-level constraints if possible


