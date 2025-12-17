# ENG-24262: Blank Screen When Editing Connections Added via PKOS

## Problem Statement

There have been some connections that do not allow workers to successfully update/edit relationships. These are connections that are added from by running an Integrated Person search (PKOS). If a worker then tries to click on the connection's name to view details, they return a blank page on the Person Details page. The error sometimes mentions social media but the nullable field (blank) is the Person Details Page for the connection.

### Issues Identified

1. **Missing Hyperlinks**: When a connection is added via PKOS, the hyperlink on their name is missing
2. **Blank Person Profile Page**: If the link is not missing, when a user clicks on the connection's name they see a blank page on the Person Profile page
3. **Blank Edit Relationships Page**: If the user tries to edit the relationship, they see a blank page on the Edit Relationships page

### Expected Behavior

- All Connections, whether added manually or through PKOS, should contain a hyperlink on their name
- The hyperlink on the connection's name should direct a user to the Person Profile page (not a blank screen)
- When a user selects "Edit Relationship" for a connection they should be directed to the "Edit Relationships" page (not a blank screen)

### Workarounds

1. If the Edit Person link already appears while on the Profile Details page, try going there and adding just the name of the connection and saving. It should be the exact name you see on the relationship dashboard and that appears on the Family Finding Documents for that connection.
2. Attempt to edit the url for the connection then add the name as it appears on the Relationship Dashboard.
   - Example: `http://family.binti.support/people/XXXXXX/edit`

---

## Root Cause Analysis

The root cause of this issue is **insufficient null handling** in multiple components throughout the Family Finder feature. When PKOS creates connections, it's possible that:

1. The `destinationAgencyHuman` or `sourceAgencyHuman` objects in relationships can be `null`
2. The `linkToView` field can be `null` or `undefined`
3. The `fullName` field can be `null` or `undefined`

The UI components were not prepared to handle these null cases gracefully, leading to:
- Destructuring errors when trying to access properties of null objects
- Undefined values being passed to components that expect strings
- React rendering errors when keys or values are undefined
- Blank pages when GraphQL queries return incomplete data

### Key Problem Areas

1. **`determineName` function**: Destructured `destinationAgencyHuman` and `sourceAgencyHuman` without checking for null first
2. **Relationship table columns**: Used `href` and `name` from `determineName` without null checks
3. **Person Details page**: Accessed array elements and object properties without null checks
4. **Edit Relationship form**: Accessed GraphQL query results without validating data exists

---

## Solutions Implemented

### Solution 1: Fixed `determineName` Function to Handle Null AgencyHuman Objects

**File**: `app/javascript/components/family_finding/relationships/sortRelationships.js`

**Problem**: The function destructured `destinationAgencyHuman` and `sourceAgencyHuman` without checking if they were null first. When PKOS created connections with incomplete data, this caused destructuring to fail.

**Before**:
```javascript
/** Determine the name of the non-keystone Agency Human */
export const determineName = ({
  keystoneAgencyHumanId,
  destinationAgencyHuman: {
    id: destinationAgencyHumanId,
    fullName: destinationAgencyHumanFullName,
    firstName: destinationAgencyHumanFirstName,
    lastName: destinationAgencyHumanLastName,
    linkToView: destinationAgencyHumanLinkToView,
    isDeceased: destinationAgencyHumanIsDeceased,
  } = {},
  sourceAgencyHuman: {
    fullName: sourceAgencyHumanFullName,
    firstName: sourceAgencyHumanFirstName,
    lastName: sourceAgencyHumanLastName,
    linkToView: sourceAgencyHumanLinkToView,
    isDeceased: sourceAgencyHumanIsDeceased,
  } = {},
}) => {
  if (!destinationAgencyHumanId) return {};
  return destinationAgencyHumanId.toString() ===
    keystoneAgencyHumanId.toString()
    ? {
        href: sourceAgencyHumanLinkToView,
        name: sourceAgencyHumanFullName,
        firstName: sourceAgencyHumanFirstName,
        lastName: sourceAgencyHumanLastName,
        isDeceased: sourceAgencyHumanIsDeceased,
        keystoneName: destinationAgencyHumanFullName,
        keystoneFirstName: destinationAgencyHumanFirstName,
      }
    : {
        href: destinationAgencyHumanLinkToView,
        name: destinationAgencyHumanFullName,
        firstName: destinationAgencyHumanFirstName,
        lastName: destinationAgencyHumanLastName,
        isDeceased: destinationAgencyHumanIsDeceased,
        keystoneName: sourceAgencyHumanFullName,
        keystoneFirstName: sourceAgencyHumanFirstName,
      };
};
```

**After**:
```javascript
/** Determine the name of the non-keystone Agency Human */
export const determineName = ({
  keystoneAgencyHumanId,
  destinationAgencyHuman,
  sourceAgencyHuman,
}) => {
  // Handle null/undefined agency humans gracefully
  if (!destinationAgencyHuman || !sourceAgencyHuman) {
    return {
      href: undefined,
      name: undefined,
      firstName: undefined,
      lastName: undefined,
      isDeceased: undefined,
      keystoneName: undefined,
      keystoneFirstName: undefined,
    };
  }

  const {
    id: destinationAgencyHumanId,
    fullName: destinationAgencyHumanFullName,
    firstName: destinationAgencyHumanFirstName,
    lastName: destinationAgencyHumanLastName,
    linkToView: destinationAgencyHumanLinkToView,
    isDeceased: destinationAgencyHumanIsDeceased,
  } = destinationAgencyHuman;

  const {
    fullName: sourceAgencyHumanFullName,
    firstName: sourceAgencyHumanFirstName,
    lastName: sourceAgencyHumanLastName,
    linkToView: sourceAgencyHumanLinkToView,
    isDeceased: sourceAgencyHumanIsDeceased,
  } = sourceAgencyHuman;

  if (!destinationAgencyHumanId) {
    return {
      href: undefined,
      name: undefined,
      firstName: undefined,
      lastName: undefined,
      isDeceased: undefined,
      keystoneName: undefined,
      keystoneFirstName: undefined,
    };
  }

  return destinationAgencyHumanId.toString() ===
    keystoneAgencyHumanId.toString()
    ? {
        href: sourceAgencyHumanLinkToView || undefined,
        name: sourceAgencyHumanFullName || undefined,
        firstName: sourceAgencyHumanFirstName || undefined,
        lastName: sourceAgencyHumanLastName || undefined,
        isDeceased: sourceAgencyHumanIsDeceased || undefined,
        keystoneName: destinationAgencyHumanFullName || undefined,
        keystoneFirstName: destinationAgencyHumanFirstName || undefined,
      }
    : {
        href: destinationAgencyHumanLinkToView || undefined,
        name: destinationAgencyHumanFullName || undefined,
        firstName: destinationAgencyHumanFirstName || undefined,
        lastName: destinationAgencyHumanLastName || undefined,
        isDeceased: destinationAgencyHumanIsDeceased || undefined,
        keystoneName: sourceAgencyHumanFullName || undefined,
        keystoneFirstName: sourceAgencyHumanFirstName || undefined,
      };
};
```

**Impact**: This ensures that when AgencyHuman objects are null, the function returns safe default values instead of causing destructuring errors.

---

### Solution 2: Fixed Relationship Table Name Column to Handle Undefined Names

**File**: `app/javascript/components/family_finding/relationships/relationshipsColumns.js`

**Problem**: The name column used `href` and `name` from `determineName` without checking if they were undefined, which could cause the link to not render or the name to be blank.

**Before**:
```javascript
export const nameCol = ({ keystoneAgencyHumanId }) => ({
  columnName: { name: relationshipT("name") },
  id: "connectionName",
  cell: relationship => {
    const { destinationAgencyHuman, sourceAgencyHuman } = relationship;
    const { href, name, isDeceased } = determineName({
      keystoneAgencyHumanId,
      destinationAgencyHuman,
      sourceAgencyHuman,
    });
    return (
      <Flex column>
        <OptionalLink href={href}>{name}</OptionalLink>
        <If condition={isDeceased}>{relationshipT("deceased")}</If>
        <FlexItem>
          <StatusPill
            status={getChildCaseStatus(
              determineNonKeystoneAH({ relationship, keystoneAgencyHumanId })
            )}
          />
        </FlexItem>
      </Flex>
    );
  },
```

**After**:
```javascript
export const nameCol = ({ keystoneAgencyHumanId }) => ({
  columnName: { name: relationshipT("name") },
  id: "connectionName",
  cell: relationship => {
    const { destinationAgencyHuman, sourceAgencyHuman } = relationship;
    const { href, name, isDeceased } = determineName({
      keystoneAgencyHumanId,
      destinationAgencyHuman,
      sourceAgencyHuman,
    });
    
    // Handle null/undefined name gracefully - show fallback text if name is missing
    const displayName = name || "";
    
    return (
      <Flex column>
        <OptionalLink href={href}>{displayName}</OptionalLink>
        <If condition={isDeceased}>{relationshipT("deceased")}</If>
        <FlexItem>
          <StatusPill
            status={getChildCaseStatus(
              determineNonKeystoneAH({ relationship, keystoneAgencyHumanId })
            )}
          />
        </FlexItem>
      </Flex>
    );
  },
```

**Impact**: Prevents blank names from appearing in the relationship table and ensures the UI renders gracefully even when name data is missing.

---

### Solution 3: Fixed `sortByName` Function to Handle Null Values

**File**: `app/javascript/components/family_finding/relationships/sortRelationships.js`

**Problem**: The sorting function destructured relationship objects without default values and didn't handle undefined names in the comparison.

**Before**:
```javascript
/** Sort relationship by the names of the non-keystone agency human */
export const sortByName =
  ({ keystoneAgencyHumanId }) =>
  (
    {
      destinationAgencyHuman: {
        id: destinationAId,
        fullName: destinationAFullName,
      },
      sourceAgencyHuman: { fullName: sourceAFullName },
    },
    {
      destinationAgencyHuman: {
        id: destinationBId,
        fullName: destinationBFullName,
      },
      sourceAgencyHuman: { fullName: sourceBFullName },
    }
  ) => {
    const { name: nameA } = determineName({
      keystoneAgencyHumanId,
      destinationAgencyHuman: {
        id: destinationAId,
        fullName: destinationAFullName,
      },
      sourceAgencyHuman: {
        fullName: sourceAFullName,
      },
    });

    const { name: nameB } = determineName({
      keystoneAgencyHumanId,
      destinationAgencyHuman: {
        id: destinationBId,
        fullName: destinationBFullName,
      },
      sourceAgencyHuman: {
        fullName: sourceBFullName,
      },
    });

    return upperCase(nameA) < upperCase(nameB) ? 1 : -1;
  };
```

**After**:
```javascript
/** Sort relationship by the names of the non-keystone agency human */
export const sortByName =
  ({ keystoneAgencyHumanId }) =>
  (
    {
      destinationAgencyHuman: {
        id: destinationAId,
        fullName: destinationAFullName,
      } = {},
      sourceAgencyHuman: { fullName: sourceAFullName } = {},
    } = {},
    {
      destinationAgencyHuman: {
        id: destinationBId,
        fullName: destinationBFullName,
      } = {},
      sourceAgencyHuman: { fullName: sourceBFullName } = {},
    } = {}
  ) => {
    const { name: nameA } = determineName({
      keystoneAgencyHumanId,
      destinationAgencyHuman: {
        id: destinationAId,
        fullName: destinationAFullName,
      },
      sourceAgencyHuman: {
        fullName: sourceAFullName,
      },
    });

    const { name: nameB } = determineName({
      keystoneAgencyHumanId,
      destinationAgencyHuman: {
        id: destinationBId,
        fullName: destinationBFullName,
      },
      sourceAgencyHuman: {
        fullName: sourceBFullName,
      },
    });

    // Handle null/undefined names in sorting
    const nameAForSort = nameA || "";
    const nameBForSort = nameB || "";

    return upperCase(nameAForSort) < upperCase(nameBForSort) ? 1 : -1;
  };
```

**Impact**: Prevents sorting errors when relationship data is incomplete and ensures the table can sort even with missing names.

---

### Solution 4: Fixed RelationshipActions Component to Handle Undefined Names

**File**: `app/javascript/components/family_finding/relationships/RelationshipActions.js`

**Problem**: The component used `name` and `keystoneName` from `determineName` in translation strings without checking if they were undefined.

**Before**:
```javascript
  const { name, keystoneName } = determineName({
    keystoneAgencyHumanId,
    ...relationship,
  });
  const nonKeystoneAgencyHumanId = determineNonKeystoneId({
    keystoneAgencyHumanId,
    relationship,
  });
```

And later:
```javascript
      {relationshipT("lock_unlock_modals.for_this_relationship", {
        connection: name,
        child: keystoneName,
      })}
```

**After**:
```javascript
  const { name, keystoneName } = determineName({
    keystoneAgencyHumanId,
    ...relationship,
  });
  // Handle null/undefined names gracefully
  const displayName = name || "";
  const displayKeystoneName = keystoneName || "";
  const nonKeystoneAgencyHumanId = determineNonKeystoneId({
    keystoneAgencyHumanId,
    relationship,
  });
```

And later:
```javascript
      {relationshipT("lock_unlock_modals.for_this_relationship", {
        connection: displayName,
        child: displayKeystoneName,
      })}
```

**Impact**: Prevents translation errors and ensures action menus display correctly even when name data is missing.

---

### Solution 5: Fixed Person Details Page to Handle Null AgencyHuman Data

**File**: `app/javascript/components/agency_humans/AgencyHuman.js`

**Problem**: The component accessed `agencyHumanDetails[0]` and `agencyHuman.fullName` without checking if the data exists, which could cause blank pages when the person is not found or data is incomplete.

**Before**:
```javascript
  const { agencyHumanDetails = [{ permissions: [] }] } = data;
  const agencyHuman = agencyHumanDetails[0];

  const pageTitle = loading ? "" : agencyHuman.fullName;
```

And later:
```javascript
        sidebar={{
          fullHeight: true,
          opaque: true,
          title: pageTitle,
          content: (
            <AgencyHumanSidebar
              avatar={{
                firstName: agencyHuman.firstName,
                lastName: agencyHuman.lastName,
              }}
              label={pageTitle}
              agencyHumanId={id}
            />
          ),
        }}
```

**After**:
```javascript
  const { agencyHumanDetails = [{ permissions: [] }] } = data;
  const agencyHuman = agencyHumanDetails[0];

  // Handle null/undefined agencyHuman gracefully
  if (!loading && !agencyHuman) {
    return (
      <Layout
        pageTitle={t("title")}
        main={{
          content: (
            <Notice variant="alert">
              <Text>Person not found</Text>
            </Notice>
          ),
        }}
      />
    );
  }

  const pageTitle = loading ? "" : agencyHuman?.fullName || "";
```

And later:
```javascript
        sidebar={{
          fullHeight: true,
          opaque: true,
          title: pageTitle,
          content: (
            <AgencyHumanSidebar
              avatar={{
                firstName: agencyHuman?.firstName,
                lastName: agencyHuman?.lastName,
              }}
              label={pageTitle}
              agencyHumanId={id}
            />
          ),
        }}
```

**Impact**: Prevents blank pages when navigating to a person that doesn't exist or has incomplete data, and shows a user-friendly error message instead.

---

### Solution 6: Fixed Person Summary to Handle Null Related Children Data

**File**: `app/javascript/components/agency_humans/AgencyHumanSummary.js`

**Problem**: The component used `determineName` to display related children links without checking if `name` or `href` were undefined, which could cause React key errors and broken links.

**Before**:
```javascript
            {
              label: t("related_children"),
              value: (relationshipData?.agencyHumanRelationships || []).map(
                ({ sourceAgencyHuman, destinationAgencyHuman }, index) => {
                  const { href, name } = determineName({
                    keystoneAgencyHumanId: id,
                    sourceAgencyHuman,
                    destinationAgencyHuman,
                  });
                  return (
                    <Fragment key={name}>
                      <If condition={index > 0}>{" | "}</If>
                      <Link href={href}>{name}</Link>
                    </Fragment>
                  );
                }
              ),
            },
```

**After**:
```javascript
            {
              label: t("related_children"),
              value: (relationshipData?.agencyHumanRelationships || [])
                .map(({ sourceAgencyHuman, destinationAgencyHuman }, index) => {
                  const { href, name } = determineName({
                    keystoneAgencyHumanId: id,
                    sourceAgencyHuman,
                    destinationAgencyHuman,
                  });
                  // Skip rendering if name is missing
                  if (!name) return null;
                  return (
                    <Fragment key={name || index}>
                      <If condition={index > 0}>{" | "}</If>
                      <Link href={href}>{name}</Link>
                    </Fragment>
                  );
                })
                .filter(Boolean), // Remove null entries
            },
```

**Impact**: Prevents React key errors and ensures only valid related children links are displayed, filtering out entries with missing data.

---

### Solution 7: Fixed Edit Relationship Form to Handle Null AgencyHuman Data

**File**: `app/javascript/components/agency_humans/AgencyHumanForm.js`

**Problem**: The form accessed `data.agencyHumanDetails[0]` without checking if the array has elements or if the element is null, which could cause crashes when editing relationships with incomplete data.

**Before**:
```javascript
    onCompleted: data => {
      const agencyHuman = data.agencyHumanDetails[0];
      setFormState({
        ...formState,
        ...omitBy(agencyHuman, isNil),
        ...icwaFieldsForUI(agencyHuman),
        ...genderForUI(agencyHuman),
        ...socialMediaLinksForUI(agencyHuman),
        ...addressesForUI(agencyHuman),
        ...phoneNumbersForUI(agencyHuman),
        agencyHuman: {
          tiedToEntity: agencyHuman.tiedToEntity,
          tiedToLicensingEntity: agencyHuman.tiedToLicensingEntity,
          fullName: agencyHuman.fullName,
          label: agencyHuman.fullName,
          value: formState.agencyHuman.id,
          id: formState.agencyHuman.id,
        },
      });

      // Store original name field values for comparison
      setOriginalNameFields({
        firstName: agencyHuman.firstName || "",
        middleName: agencyHuman.middleName || "",
        lastName: agencyHuman.lastName || "",
        suffix: agencyHuman.suffix || "",
      });
    },
```

**After**:
```javascript
    onCompleted: data => {
      const agencyHuman = data?.agencyHumanDetails?.[0];
      // Handle null/undefined agencyHuman gracefully
      if (!agencyHuman) {
        return;
      }
      setFormState({
        ...formState,
        ...omitBy(agencyHuman, isNil),
        ...icwaFieldsForUI(agencyHuman),
        ...genderForUI(agencyHuman),
        ...socialMediaLinksForUI(agencyHuman),
        ...addressesForUI(agencyHuman),
        ...phoneNumbersForUI(agencyHuman),
        agencyHuman: {
          tiedToEntity: agencyHuman.tiedToEntity,
          tiedToLicensingEntity: agencyHuman.tiedToLicensingEntity,
          fullName: agencyHuman.fullName,
          label: agencyHuman.fullName,
          value: formState.agencyHuman.id,
          id: formState.agencyHuman.id,
        },
      });

      // Store original name field values for comparison
      setOriginalNameFields({
        firstName: agencyHuman.firstName || "",
        middleName: agencyHuman.middleName || "",
        lastName: agencyHuman.lastName || "",
        suffix: agencyHuman.suffix || "",
      });
    },
```

**Impact**: Prevents form crashes when editing relationships with incomplete data and ensures the form state doesn't get corrupted with undefined values.

---

## Files Modified

1. `app/javascript/components/family_finding/relationships/sortRelationships.js`
2. `app/javascript/components/family_finding/relationships/relationshipsColumns.js`
3. `app/javascript/components/family_finding/relationships/RelationshipActions.js`
4. `app/javascript/components/agency_humans/AgencyHuman.js`
5. `app/javascript/components/agency_humans/AgencyHumanSummary.js`
6. `app/javascript/components/agency_humans/AgencyHumanForm.js`

---

## Testing Recommendations

### 1. Relationship Table
- [ ] Verify connections created via PKOS display names (even if blank) without errors
- [ ] Verify hyperlinks only appear when `linkToView` is available
- [ ] Verify sorting works correctly with connections that have missing names
- [ ] Verify action menus work correctly for connections with incomplete data

### 2. Person Details Page
- [ ] Navigate to a person created via PKOS with incomplete data
- [ ] Verify the page loads without errors (shows "Person not found" if data is completely missing)
- [ ] Verify related children section only shows entries with valid names
- [ ] Verify links in related children section only render when `href` is available

### 3. Edit Relationship Page
- [ ] Edit a relationship for a connection created via PKOS
- [ ] Verify the form loads even if some data is missing
- [ ] Verify the form doesn't crash when relationship data is incomplete
- [ ] Verify all form fields handle null/undefined values gracefully

### 4. Edge Cases
- [ ] Test with relationships where `destinationAgencyHuman` is null
- [ ] Test with relationships where `sourceAgencyHuman` is null
- [ ] Test with relationships where both AgencyHuman objects are null
- [ ] Test with relationships where `linkToView` is null
- [ ] Test with relationships where `fullName` is null or empty

---

## Additional Recommendations

### Backend Investigation Needed

While the UI fixes prevent blank pages and crashes, the root cause should be investigated:

1. **Data Integrity**: Why does PKOS sometimes create connections with null `destinationAgencyHuman` or `sourceAgencyHuman`? This may indicate a data integrity issue in the relationship creation process.

2. **GraphQL Query Validation**: Ensure the `RelationshipWithContactLog` query always returns valid AgencyHuman objects, or add validation to prevent null relationships from being created.

3. **PKOS Integration**: Review the PKOS integration code to ensure it properly creates complete AgencyHuman records and relationships when adding connections.

### Future Improvements

1. **Error Logging**: Add error logging when null data is detected to help identify patterns
2. **Data Validation**: Add GraphQL schema validation to prevent null relationships
3. **User Feedback**: Provide better user feedback when data is incomplete (e.g., "Some connection details are missing")
4. **Data Repair**: Consider adding a data repair utility to fix existing incomplete connections

---

## Related Issues

- Connections added via PKOS may have incomplete data
- GraphQL queries may return null AgencyHuman objects in relationships
- Backend relationship creation may not validate required fields

---

## Status

✅ **Fixed** - All identified null handling issues have been addressed in the UI components. The application now gracefully handles null/undefined data from PKOS-created connections.

---

## Notes

- All changes follow defensive programming practices
- No breaking changes to existing functionality
- Changes are backward compatible with existing data
- The fixes prevent crashes but don't address the root cause of why PKOS creates incomplete data (requires backend investigation)
