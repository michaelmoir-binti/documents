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

### What is .erb?

**.erb** stands for **ERB (Embedded Ruby)**. It's a templating system used in Ruby on Rails.

**How it works:**
- `.erb` files are HTML templates with embedded Ruby code
- Ruby code is executed on the **server** before sending HTML to the browser
- The Ruby code is replaced with its output (text, HTML, etc.)
- Final result is pure HTML sent to the browser

**Syntax:**
```erb
<!-- Ruby code that outputs text -->
<%= ruby_expression %>

<!-- Ruby code that doesn't output (loops, conditionals) -->
<% ruby_code %>

<!-- HTML that's sent as-is -->
<div>Hello World</div>
```

**Example:**
```erb
<h1>Welcome, <%= current_user.name %>!</h1>
<% if user.admin? %>
  <p>You are an admin</p>
<% end %>
```

**When executed on server:**
- `<%= current_user.name %>` → Evaluates Ruby, outputs "John"
- `<% if user.admin? %>` → Evaluates Ruby condition, includes/excludes HTML
- Result sent to browser: `<h1>Welcome, John!</h1><p>You are an admin</p>`

**Key point:** All Ruby code runs **server-side**. The browser never sees Ruby code, only the final HTML output.

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
│   → FamilyFindingSearchesController#index │
│   → Renders view template:          │
│      app/views/family_finding/      │
│        searches/index.html.erb      │
│   → Uses layout:                    │
│      app/views/layouts/             │
│        application_full_width.erb   │
│   → Includes JavaScript bundles     │
│   → react_component helper renders  │
│      mounting div for React         │
└──────┬──────────────────────────────┘
       │
       │ 2. HTML Response
       │    - Complete HTML document
       │    - <div data-react-component="...">
       │      (mounting point for React)
       │    - JavaScript bundle URLs
       │    - CSS files
       │    - Server-side props (counties data)
       │
       ▼
┌─────────────┐
│   Browser   │
│   ────────  │
│   • Parses HTML                    │
│   • Downloads JavaScript bundles   │
│   • Executes JavaScript            │
│   • React finds react_component div│
│   • React mounts to div            │
│   • Receives props (counties)      │
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

### HTML Template Files

The initial HTML page is composed of multiple template files that Rails renders together:

#### 1. Base HTML Layout
**File**: `app/views/layouts/_application.html.erb`

This is the root HTML template that provides the basic HTML structure:
```erb
<!DOCTYPE html>
<html lang="<%= (I18n.locale or "en") %>">
  <head>
    <title><%= meta_title %></title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- CSS files -->
    <%= stylesheet_link_tag "shared_styles_sprockets_dependent" %>
    <%= stylesheet_packs_with_chunks_tag "applicant_only", "application", media: :all %>
    <!-- JavaScript bundles -->
    <%= javascript_include_tag "application" %>
    <!-- Other meta tags, CSRF tokens, etc. -->
  </head>
  <body class="<%= BintiFamily::DEPLOYMENT_ENV %>">
    <div id="wrapper">
      <%= yield :body_content %>
    </div>
    <%= render "shared/footer", type: "application" %>
  </body>
</html>
```

**What it does:**
- Provides `<html>`, `<head>`, `<body>` structure
- Includes CSS and JavaScript bundle references
- Sets up meta tags, CSRF tokens
- Provides a wrapper div where page content goes

#### 2. Layout Wrapper
**File**: `app/views/layouts/application_full_width.erb`

This wraps the base layout and provides the page structure:
```erb
<% content_for(:body_content) do %>
    <%= render partial: "layouts/non_applicant_portal_header" %>
    <main>
        <%= render partial: "shared/notifications", locals: { flash_messages: flash } %>
        <%= yield %>  <!-- This is where the view content goes -->
    </main>
<% end %>

<%= render("layouts/application") %>
```

**What it does:**
- Renders the header/navigation
- Sets up the main content area
- Includes flash message notifications
- Uses `yield` to insert the specific view content

#### 3. Family Finding Searches View
**File**: `app/views/family_finding/searches/index.html.erb`

This is the specific template for the Family Finding searches page:
```erb
<%- content_for(:title, "Binti | #{I18n.t("javascript.components.family_finding.searches.title")}") %>
<% counties_by_state = Geography::USA::CountiesOfStates::ALL %>
<%= react_component("family_finding/searches/Searches", {
      counties: current_user.agency.state ? counties_by_state[current_user.agency.state] : counties_by_state.values.flatten.uniq
    },
  ) %>
```

**What it does:**
- Sets the page title (via `content_for(:title)`)
- Prepares data (counties list) from server-side Ruby
- Uses `react_component` helper to render the React component
- Passes server-side data (counties) as props to React

#### How `react_component` Works

The `react_component` helper (from the `react-rails` gem) does the following:

1. **Server-side (Ruby):**
   - Renders a `<div>` with data attributes:
     ```html
     <div 
       data-react-component="family_finding/searches/Searches"
       data-react-props='{"counties":["County1","County2",...]}'
       data-react-class="..."
     ></div>
     ```

2. **Client-side (JavaScript):**
   - JavaScript bundle (loaded in `<head>`) finds all `react_component` divs
   - React mounts to those divs
   - Props are passed to the React component
   - Component renders and takes over the UI

**Key Points:**
- The HTML template is **minimal** - it doesn't contain search data
- The `react_component` helper creates a **mounting point** for React
- **Server-side data** (like counties) can be passed as props
- All **data fetching** happens client-side via GraphQL after React mounts

### Files Involved

#### Server-Side (Ruby)
1. **Routes** (`config/routes.rb`)
   - Defines URL → controller mapping
   - Example: `/family_finding/searches` → `FamilyFindingSearchesController#index`

2. **Controller** (`app/controllers/family_finding/searches_controller.rb` - if exists)
   - Handles the HTTP request
   - Renders the view template
   - May prepare data for the view

3. **View Template** (`app/views/family_finding/searches/index.html.erb`)
   - ERB template that renders the HTML shell
   - Uses `react_component` helper to mount React
   - Passes server-side data as props

4. **Layout Files**
   - `app/views/layouts/_application.html.erb` - Base HTML structure
   - `app/views/layouts/application_full_width.erb` - Page wrapper

5. **GraphQL Query Resolver** (`app/graphql/queries/family_finding_searches.rb`)
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
- Rails renders initial HTML shell using ERB templates
- Layout files compose together (_application.html.erb → application_full_width.erb → index.html.erb)
- `react_component` helper creates mounting div with data attributes
- Server-side data (counties) passed as props via data attributes
- JavaScript bundle references included in `<head>`
- No search data in initial HTML (fetched client-side via GraphQL)

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
