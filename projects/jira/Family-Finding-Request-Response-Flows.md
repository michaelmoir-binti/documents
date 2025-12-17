# Family Finding Feature: Request/Response Flow Analysis

## Architecture Overview

This application uses a **hybrid architecture**:
- **Backend**: Ruby on Rails (server-side)
  - Handles routing, authorization, business logic
  - Serves initial HTML page
  - Provides GraphQL API endpoints
- **Frontend**: JavaScript/React (client-side)
  - Handles UI rendering and interactions
  - Makes GraphQL queries/mutations
  - Manages client-side state

---

## Flow 1: Initial Page Load - Family Finding Searches Dashboard

### Overview
When a user navigates to the Family Finding Searches page, the browser loads an HTML page that includes JavaScript, which then fetches and displays the search data.

### Detailed Flow

```
┌─────────────┐
│   Browser   │
└──────┬──────┘
       │
       │ 1. GET /family_finding/searches
       │    (Initial page request)
       ▼
┌─────────────────────────────────────┐
│   Rails Server (Ruby)               │
│   ───────────────────────────────   │
│   Routes: config/routes.rb          │
│   → Matches route to controller     │
│   → FamilyFindingSearchesController │
│   → Renders HTML template           │
│   → Includes JavaScript bundles     │
└──────┬──────────────────────────────┘
       │
       │ 2. HTML Response
       │    - Basic page structure
       │    - <div id="root"></div>
       │    - JavaScript bundle URLs
       │    - CSS files
       │
       ▼
┌─────────────┐
│   Browser   │
│   ────────  │
│   • Parses HTML                    │
│   • Downloads JavaScript bundles   │
│   • Executes JavaScript            │
│   • React mounts to #root          │
└──────┬──────┘
       │
       │ 3. JavaScript Execution
       │    Searches.js component renders
       │    → useQuery hooks trigger
       │
       ▼
┌─────────────────────────────────────┐
│   React Component (Client-Side)     │
│   ───────────────────────────────   │
│   File: Searches.js                 │
│                                     │
│   Component renders:                │
│   • Layout with breadcrumbs        │
│   • ContentTabs (Open/Closed/All)   │
│   • Table component (empty)        │
│                                     │
│   useQuery hooks execute:           │
│   • CurrentUserQuery               │
│   • AgencyQuery                    │
│   • FamilyFindingSearchesQuery     │
└──────┬──────────────────────────────┘
       │
       │ 4. GraphQL Queries (HTTP POST)
       │    POST /graphql
       │    {
       │      query: FamilyFindingSearchesQuery
       │      variables: { status: "open", ... }
       │    }
       │
       ▼
┌─────────────────────────────────────┐
│   Rails GraphQL Server (Ruby)        │
│   ───────────────────────────────   │
│   File: app/graphql/queries/         │
│        family_finding_searches.rb     │
│                                     │
│   1. Authorization check            │
│      authorized_via_policy_scope    │
│                                     │
│   2. Call service object            │
│      FamilyFinding::Searches::       │
│        GetSearches.call(...)         │
│                                     │
│   3. Query database                  │
│      • Filter by status             │
│      • Apply filters (dates, etc)   │
│      • Paginate results             │
│      • Load associations            │
│                                     │
│   4. Return GraphQL response         │
└──────┬──────────────────────────────┘
       │
       │ 5. JSON Response
       │    {
       │      data: {
       │        familyFindingSearches: {
       │          nodes: [...],
       │          pageInfo: {...}
       │        },
       │        openSearches: { totalCount: 5 },
       │        closedSearches: { totalCount: 3 }
       │      }
       │    }
       │
       ▼
┌─────────────────────────────────────┐
│   React Component (Client-Side)     │
│   ───────────────────────────────   │
│   File: Searches.js                  │
│                                     │
│   • Apollo Client receives data     │
│   • Component re-renders            │
│   • Table displays search results   │
│   • Tab counts update              │
│   • Filters populate                │
└─────────────────────────────────────┘
```

### Files Involved

#### Server-Side (Ruby)
1. **Routes** (`config/routes.rb`)
   - Defines URL → controller mapping
   - Example: `/family_finding/searches` → `FamilyFindingSearchesController`

2. **Controller** (`app/controllers/family_finding/searches_controller.rb` - if exists)
   - OR handled by React Router (SPA routing)
   - Renders initial HTML page

3. **GraphQL Query Resolver** (`app/graphql/queries/family_finding_searches.rb`)
   ```ruby
   class FamilyFindingSearches < BasePaginatedQuery
     query do |status: "all", ...|
       authorized_via_policy_scope
       
       ::FamilyFinding::Searches::GetSearches.call(
         scoped_search: policy_scope(::FamilyFinding::Search),
         status: status,
         ...
       )
     end
   end
   ```

4. **Service Object** (`app/lib/services/family_finding/searches/get_searches.rb`)
   - Business logic for fetching searches
   - Applies filters, pagination, sorting

#### Client-Side (JavaScript)
1. **Main Component** (`app/javascript/components/family_finding/searches/Searches.js`)
   ```javascript
   const Searches = ({ counties }) => {
     // Queries for user and agency data
     const { data: currentUser } = useQuery(CurrentUserQuery);
     const { data: agencyData } = useQuery(AgencyQuery, {...});
     
     // Main search query
     return (
       <GraphQLDataTable
         query={FamilyFindingSearchesQuery}
         ...
       />
     );
   };
   ```

2. **GraphQL Query** (`app/javascript/graphql/queries/FamilyFindingSearches.graphql`)
   ```graphql
   query FamilyFindingSearchesQuery(...) {
     familyFindingSearches(...) {
       nodes { ... }
       pageInfo { ... }
     }
     openSearches: familyFindingSearches(status: open) {
       totalCount
     }
   }
   ```

3. **Table Columns** (`app/javascript/components/family_finding/searches/columns.js`)
   - Defines how each column renders
   - Handles sorting, filtering

### Composition Points

**Server-Side Composition:**
- Rails renders initial HTML shell
- Includes JavaScript bundle references
- No data in initial HTML (empty `<div id="root"></div>`)

**Client-Side Composition:**
- React mounts and renders component tree
- Apollo Client makes GraphQL requests
- Components compose together (Layout → ContentTabs → Table)
- Data flows down via props, events flow up via callbacks

---

## Flow 2: Create a New Search

### Overview
User clicks "Create New Search" button, navigates to form, fills it out, and submits to create a new Family Finding search.

### Detailed Flow

```
┌─────────────┐
│   Browser   │
│   ────────  │
│   User clicks "Create New Search"  │
│   → Link: /family_finding/searches/new
└──────┬──────┘
       │
       │ 1. Navigation (Client-Side Routing)
       │    React Router handles navigation
       │    No server request needed
       │
       ▼
┌─────────────────────────────────────┐
│   React Component (Client-Side)     │
│   ───────────────────────────────   │
│   File: SearchForm.js               │
│                                     │
│   Component renders:                │
│   • Wizard with 3 pages:            │
│     1. Select Children              │
│     2. Sibling Relationships        │
│     3. Search Information           │
│                                     │
│   Initial queries:                  │
│   • AgencyHumanAutocompleteForSearches
│   • AgencyWorkerAutocomplete        │
│   • AgencyHumanRelationshipsForGroup
└──────┬──────────────────────────────┘
       │
       │ 2. GraphQL Queries (for form data)
       │    POST /graphql
       │    - Autocomplete queries for dropdowns
       │    - Relationship data for siblings
       │
       ▼
┌─────────────────────────────────────┐
│   Rails GraphQL Server (Ruby)        │
│   ───────────────────────────────   │
│   Returns:                           │
│   • List of children (autocomplete) │
│   • List of workers (autocomplete)  │
│   • Relationship data               │
└──────┬──────────────────────────────┘
       │
       │ 3. User fills form
       │    • Selects children
       │    • Defines sibling relationships
       │    • Sets start date, workers
       │
       ▼
┌─────────────────────────────────────┐
│   React Component (Client-Side)     │
│   ───────────────────────────────   │
│   File: SearchForm.js                │
│                                     │
│   User clicks "Save"                │
│   → Triggers mutation:              │
│     createSearch({                  │
│       variables: {                  │
│         childAgencyHumanIds: [...], │
│         startDate: "...",            │
│         assignedWorkerIds: [...]    │
│       }                             │
│     })                              │
└──────┬──────────────────────────────┘
       │
       │ 4. GraphQL Mutation
       │    POST /graphql
       │    mutation CreateFamilyFindingSearch(...)
       │
       ▼
┌─────────────────────────────────────┐
│   Rails GraphQL Server (Ruby)       │
│   ───────────────────────────────   │
│   File: app/graphql/mutations/       │
│        create_family_finding_search.rb
│                                     │
│   1. Authorization check            │
│      authorize(FamilyFinding::Search, :create?)
│                                     │
│   2. Validate inputs                │
│      • Check agency_human_ids exist │
│      • Validate dates               │
│                                     │
│   3. Call service object            │
│      FamilyFinding::Searches::      │
│        CreateSearch.call(           │
│          agency_human: ...,          │
│          start_date: ...,            │
│          assigned_workers: ...       │
│        )                            │
│                                     │
│   4. Service object:                │
│      • Creates FamilyFinding::Search records
│      • Associates workers           │
│      • Sets dates                   │
│      • Returns created searches     │
│                                     │
│   5. Return GraphQL response        │
└──────┬──────────────────────────────┘
       │
       │ 5. JSON Response
       │    {
       │      data: {
       │        createFamilyFindingSearch: {
       │          searches: [
       │            { id: "123", ... }
       │          ]
       │        }
       │      }
       │    }
       │
       ▼
┌─────────────────────────────────────┐
│   React Component (Client-Side)     │
│   ───────────────────────────────   │
│   File: SearchForm.js                │
│                                     │
│   Mutation completes                │
│   → onCompleted callback:           │
│     window.location = familyFindingSearchesPath()
│                                     │
│   Redirects to searches list        │
└─────────────────────────────────────┘
```

### Files Involved

#### Server-Side (Ruby)
1. **GraphQL Mutation** (`app/graphql/mutations/create_family_finding_search.rb`)
   ```ruby
   class CreateFamilyFindingSearch < BaseMutation
     def resolve(child_agency_human_ids:, start_date:, assigned_worker_ids: [])
       authorize(FamilyFinding::Search, :create?)
       
       searches = agency_humans.map do |agency_human|
         FamilyFinding::Searches::CreateSearch.call(
           agency_human: agency_human,
           start_date: start_date,
           assigned_workers: assigned_workers
         )
       end
       
       { searches: searches }
     end
   end
   ```

2. **Service Object** (`app/lib/services/family_finding/searches/create_search.rb`)
   - Creates the search record
   - Associates workers
   - Handles business logic

#### Client-Side (JavaScript)
1. **Form Component** (`app/javascript/components/family_finding/searches/SearchForm.js`)
   ```javascript
   export const SearchForm = ({ searchId }) => {
     const [createSearch] = useMutation(CreateFamilyFindingSearch);
     
     const handleSubmit = async () => {
       await createSearch({
         variables: {
           childAgencyHumanIds: formState.children.map(c => c.value),
           startDate: formState.startDate,
           assignedWorkerIds: formState.workers.map(w => w.value)
         }
       });
       
       window.location = familyFindingSearchesPath();
     };
     
     return <Wizard pages={[...]} />;
   };
   ```

2. **GraphQL Mutation** (`app/javascript/graphql/mutations/CreateFamilyFindingSearch.graphql`)
   ```graphql
   mutation CreateFamilyFindingSearch(
     $childAgencyHumanIds: [ID!]!
     $startDate: ISO8601Date!
     $assignedWorkerIds: [ID!]
   ) {
     createFamilyFindingSearch(input: {...}) {
       searches { id, startDate, ... }
     }
   }
   ```

### Composition Points

**Client-Side Composition:**
- Form state managed by `useBintiForm` hook
- Wizard component orchestrates multi-step form
- Each wizard page is a separate component
- Form validation happens client-side before submission

**Server-Side Composition:**
- Authorization happens first (fail fast)
- Service object encapsulates business logic
- Database transaction ensures data consistency
- Returns created records in GraphQL response

---

## Flow 3: Viewing an Existing Search

### Overview
User clicks on a search in the table, navigates to view/edit that search.

### Detailed Flow

```
┌─────────────┐
│   Browser   │
│   ────────  │
│   User clicks search row            │
│   → Link: /family_finding/searches/:id
└──────┬──────┘
       │
       │ 1. Navigation (Client-Side Routing)
       │    React Router handles navigation
       │    URL changes, component re-renders
       │
       ▼
┌─────────────────────────────────────┐
│   React Component (Client-Side)     │
│   ───────────────────────────────   │
│   File: SearchForm.js                │
│                                     │
│   Component receives searchId prop  │
│   → isCreating = false              │
│                                     │
│   Component renders:                │
│   • Loading overlay (while fetching)│
│   • Wizard with pre-filled data     │
│                                     │
│   Triggers query:                   │
│   useQuery(FamilyFindingSearch, {   │
│     variables: { id: searchId },    │
│     skip: false                     │
│   })                                │
└──────┬──────────────────────────────┘
       │
       │ 2. GraphQL Query
       │    POST /graphql
       │    query FamilyFindingSearch($id: ID!) {
       │      familyFindingSearch(id: $id) { ... }
       │    }
       │
       ▼
┌─────────────────────────────────────┐
│   Rails GraphQL Server (Ruby)       │
│   ───────────────────────────────   │
│   File: app/graphql/queries/        │
│        family_finding_search.rb     │
│                                     │
│   1. Authorization check            │
│      authorized_via_policy_scope    │
│                                     │
│   2. Find search                    │
│      policy_scope(::FamilyFinding::Search)
│        .find(id)                    │
│      • Only returns if user has permission
│                                     │
│   3. Load associations              │
│      • agency_human                 │
│      • assigned_workers             │
│      • dates, permissions           │
│                                     │
│   4. Return GraphQL response        │
└──────┬──────────────────────────────┘
       │
       │ 3. JSON Response
       │    {
       │      data: {
       │        familyFindingSearch: {
       │          id: "123",
       │          agencyHuman: { ... },
       │          startDate: "2024-01-01",
       │          assignedWorkers: [...]
       │        }
       │      }
       │    }
       │
       ▼
┌─────────────────────────────────────┐
│   React Component (Client-Side)     │
│   ───────────────────────────────   │
│   File: SearchForm.js                │
│                                     │
│   onCompleted callback:             │
│   setFormState({                    │
│     children: [{ value: id, ... }], │
│     startDate: "...",                │
│     workers: [...]                   │
│   })                                │
│                                     │
│   Component re-renders:             │
│   • Wizard pre-filled with data     │
│   • User can edit and save          │
│                                     │
│   Additional queries:               │
│   • AgencyHumanRelationshipsForGroup
│     (for sibling relationships)     │
└─────────────────────────────────────┘
```

### Files Involved

#### Server-Side (Ruby)
1. **GraphQL Query** (`app/graphql/queries/family_finding_search.rb`)
   ```ruby
   class FamilyFindingSearch < BaseQuery
     query do |id:|
       authorized_via_policy_scope
       policy_scope(::FamilyFinding::Search).find(id)
     end
   end
   ```

#### Client-Side (JavaScript)
1. **Form Component** (`app/javascript/components/family_finding/searches/SearchForm.js`)
   ```javascript
   export const SearchForm = ({ searchId }) => {
     const isCreating = !searchId;
     
     const { loading } = useQuery(FamilyFindingSearch, {
       variables: { id: searchId },
       skip: isCreating,
       onCompleted: (data) => {
         setFormState({
           children: [{ value: data.agencyHuman.id, ... }],
           startDate: data.startDate,
           workers: data.assignedWorkers.map(...)
         });
       }
     });
     
     // Additional query for relationships
     const { data: relationshipsData } = useQuery(
       AgencyHumanRelationshipsForGroup,
       { variables: { agencyHumanIds: [...] } }
     );
   };
   ```

2. **GraphQL Query** (`app/javascript/graphql/queries/FamilyFindingSearch.graphql`)
   ```graphql
   query FamilyFindingSearch($id: ID!) {
     familyFindingSearch(id: $id) {
       id
       agencyHuman { ... }
       startDate
       endDate
       assignedWorkers { ... }
     }
   }
   ```

### Composition Points

**Client-Side Composition:**
- Component conditionally loads data based on `searchId` prop
- Multiple queries compose together (search data + relationships)
- Form state populated from query results
- Loading states handled by React hooks

**Server-Side Composition:**
- Policy scope ensures user can only see authorized searches
- GraphQL type system handles data shape
- Associations loaded efficiently (eager loading)

---

## Key Concepts

### Server-Side Rendering (SSR) vs Client-Side Rendering (CSR)

**Initial Page Load:**
- **SSR**: Rails renders HTML shell (minimal)
- **CSR**: React takes over and renders everything

**Data Fetching:**
- **SSR**: No data in initial HTML
- **CSR**: JavaScript fetches data via GraphQL after page loads

**Navigation:**
- **SSR**: Full page reloads (traditional)
- **CSR**: Client-side routing (React Router), no page reload

### GraphQL vs REST

This app uses **GraphQL** instead of REST:
- Single endpoint: `/graphql`
- Client specifies exactly what data it needs
- Type-safe queries
- Efficient data fetching (no over/under-fetching)

### Authorization Flow

1. **Policy Scope** (Ruby)
   - Filters records user can see
   - Applied at query level

2. **Explicit Authorization** (Ruby)
   - `authorize(resource, :action?)`
   - Checks specific permissions

3. **Permission Data** (GraphQL)
   - Returns permissions in response
   - Client can conditionally show/hide UI

### State Management

**Server-Side State:**
- Database (source of truth)
- Session (user authentication)
- No component state

**Client-Side State:**
- React component state (`useState`)
- Form state (`useBintiForm`)
- Apollo Client cache (GraphQL data)
- URL params (filters, tabs)

---

## Summary

### Initial Page Load
1. Browser requests HTML → Rails serves shell
2. JavaScript executes → React mounts
3. React queries GraphQL → Rails processes
4. Data returns → React renders UI

### Create Search
1. Client-side navigation (no server request)
2. Form renders → User fills it out
3. Submit → GraphQL mutation
4. Rails creates record → Returns data
5. Client redirects to list

### View Search
1. Client-side navigation (no server request)
2. Component queries GraphQL for search data
3. Rails returns authorized data
4. React populates form with data
5. User can edit and save

### Key Takeaways

- **Hybrid Architecture**: Rails backend + React frontend
- **GraphQL API**: Single endpoint for all data operations
- **Client-Side Routing**: No full page reloads
- **Authorization**: Server-side (Rails) + Client-side (UI permissions)
- **Composition**: Server composes data, client composes UI
