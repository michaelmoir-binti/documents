# Testing Mechanisms in the Family Application

## Overview

The Family application uses a **multi-layered testing approach** with different frameworks for different types of tests:

1. **Unit Tests**: Test individual components/functions in isolation
2. **Integration Tests**: Test how components work together
3. **End-to-End (E2E) Tests**: Test complete user workflows

---

## Testing Frameworks

### 1. Ruby/Rails Testing: RSpec

**Framework**: RSpec (Ruby testing framework)

**What it tests**:
- Models (ActiveRecord)
- Controllers
- Service Objects
- GraphQL queries and mutations
- Policies (authorization)
- Request specs (HTTP endpoints)

**Test Files Location**: `spec/`

**Test File Naming**: `*_spec.rb`

**Example Structure**:
```ruby
require "rails_helper"

describe(FamilyFinding::Searches::CreateSearch) do
  describe("#call") do
    it("creates a search") do
      child = create(:child)
      worker = create(:agency_worker, agency: child.agency)
      search = described_class.call(
        agency_human: child.agency_human,
        start_date: Time.zone.today,
        assigned_workers: [worker]
      )
      
      expect(search.agency_human_id).to(eq(child.agency_human.id))
      expect(search.start_date).to(eq(Time.zone.today))
    end
  end
end
```

### 2. JavaScript/React Testing: Jest

**Framework**: Jest (JavaScript testing framework)

**What it tests**:
- React components
- JavaScript utilities
- Component rendering
- User interactions
- GraphQL query/mutation hooks

**Test Files Location**: `app/javascript/components/**/*.spec.js`

**Test File Naming**: `*.spec.js`

**Example Structure**:
```javascript
import { render } from "@testing-library/react";
import { MockedProvider } from "@apollo/client/testing";

describe("Searches", () => {
  it("renders searches table", async () => {
    const { baseElement } = render(
      <MockedProvider>
        <Searches counties={[]} />
      </MockedProvider>
    );
    
    expect(baseElement).toMatchSnapshot();
  });
});
```

### 3. End-to-End Testing: Playwright

**Framework**: Playwright (browser automation)

**What it tests**:
- Complete user workflows
- Browser interactions
- Full page rendering
- Cross-browser compatibility

**Test Files Location**: `spec/playwright/e2e/`

**Test File Naming**: `*.spec.ts` (TypeScript)

**Example Structure**:
```typescript
import { test, expect } from "@binti";

test.describe("Family Finding Searches dashboard", () => {
  test("User can see searches and perform actions", async ({
    page,
    session,
    pageObjectModels
  }) => {
    await session.login(agencyAdmin);
    await pageObjectModels.familyFinding.searches.gotoIndex();
    
    await expect(page.getByText("Child Name")).toBeVisible();
  });
});
```

---

## Running Tests

### Ruby/RSpec Tests

**All tests**:
```bash
b test-rspec
```

**Single test file**:
```bash
b test-rspec spec/path/to/file_spec.rb
```

**Specific test**:
```bash
b test-rspec spec/path/to/file_spec.rb:123
```

**Family Finding tests**:
```bash
b test-rspec spec/services/family_finding/
b test-rspec spec/graphql/queries/family_finding_searches_spec.rb
b test-rspec spec/models/family_finding/
```

### JavaScript/Jest Tests

**All tests**:
```bash
yarn test
```

**Single test file**:
```bash
yarn test path/to/file.spec.js
```

**Watch mode** (runs tests on file changes):
```bash
yarn test --watch
```

**Family Finding tests**:
```bash
yarn test app/javascript/components/family_finding/
```

### Playwright E2E Tests

**All tests**:
```bash
PLAYWRIGHT_HTML_OPEN=never yarn playwright test --config spec/playwright.config.ts
```

**Single test file**:
```bash
PLAYWRIGHT_HTML_OPEN=never yarn playwright test --config spec/playwright.config.ts spec/playwright/e2e/path/to/file.spec.ts
```

**Family Finding E2E tests**:
```bash
PLAYWRIGHT_HTML_OPEN=never yarn playwright test --config spec/playwright.config.ts spec/playwright/e2e/family_finding/
```

---

## Test Types for Family Finding

### 1. Model Tests

**Location**: `spec/models/family_finding/`

**What they test**:
- Model validations
- Model associations
- Model methods
- Database constraints

**Example Files**:
- `spec/models/family_finding/search_spec.rb`
- `spec/models/family_finding/clear_person_search_spec.rb`
- `spec/models/family_finding/clear_person_report_spec.rb`

**Example**:
```ruby
describe(FamilyFinding::Search) do
  describe("validations") do
    it("requires start_date") do
      search = build(:family_finding_search, start_date: nil)
      expect(search).not_to(be_valid)
    end
  end
  
  describe("associations") do
    it("belongs to agency_human") do
      search = create(:family_finding_search)
      expect(search.agency_human).to(be_present)
    end
  end
end
```

### 2. Service Object Tests

**Location**: `spec/services/family_finding/` or `spec/lib/services/family_finding/`

**What they test**:
- Business logic
- Data transformations
- External API interactions (mocked)
- Error handling

**Example Files**:
- `spec/services/family_finding/searches/create_search_spec.rb`
- `spec/services/family_finding/searches/get_searches_spec.rb`
- `spec/lib/services/family_finding/potential_kin_searches/perform_search_spec.rb`

**Example**:
```ruby
describe(FamilyFinding::Searches::CreateSearch) do
  describe("#call") do
    it("creates a search with assigned workers") do
      child = create(:child)
      worker = create(:agency_worker, agency: child.agency)
      
      search = described_class.call(
        agency_human: child.agency_human,
        start_date: Time.zone.today,
        assigned_workers: [worker]
      )
      
      expect(search).to(be_persisted)
      expect(search.assigned_workers).to(include(worker))
    end
  end
end
```

### 3. GraphQL Query/Mutation Tests

**Location**: `spec/graphql/queries/` and `spec/graphql/mutations/`

**What they test**:
- GraphQL query execution
- Authorization (who can query what)
- Data filtering and sorting
- Response structure

**Example Files**:
- `spec/graphql/queries/family_finding_searches_spec.rb`
- `spec/graphql/mutations/create_family_finding_search_spec.rb`

**Example**:
```ruby
describe(Queries::FamilyFindingSearches, :graphql) do
  let(:query) { graphql_client_query("FamilyFindingSearches") }
  
  def execute_query(variables = { status: "all" })
    graphql_execute(query, current_user: user, variables: variables)
  end
  
  it("returns searches for user's agency") do
    search = create(:family_finding_search, agency: user.agency)
    
    response = execute_query
    searches = response["data"]["familyFindingSearches"]["nodes"]
    
    expect(searches.length).to(eq(1))
    expect(searches.first["id"]).to(eq(search.id.to_s))
  end
  
  it("filters by status") do
    open_search = create(:family_finding_search, end_date: nil)
    closed_search = create(:family_finding_search, end_date: Time.zone.today)
    
    response = execute_query({ status: "open" })
    searches = response["data"]["familyFindingSearches"]["nodes"]
    
    expect(searches.length).to(eq(1))
    expect(searches.first["id"]).to(eq(open_search.id.to_s))
  end
end
```

### 4. Controller/Request Tests

**Location**: `spec/requests/` or `spec/controllers/`

**What they test**:
- HTTP request/response
- Routing
- Authorization at controller level
- Status codes

**Example Files**:
- `spec/requests/family_finding/potential_kin_search_controller_spec.rb`

**Example**:
```ruby
describe(FamilyFinding::PotentialKinSearchController) do
  describe("GET #new") do
    it("returns success for authorized user") do
      sign_in(family_finding_admin)
      get(new_family_finding_clear_person_search_path)
      expect(response).to(be_successful)
    end
    
    it("redirects unauthorized user") do
      sign_in(unauthorized_user)
      get(new_family_finding_clear_person_search_path)
      expect(response).to(redirect_to(root_path))
    end
  end
end
```

### 5. Policy Tests

**Location**: `spec/policies/`

**What they test**:
- Authorization rules
- Permission checks
- Policy scope filtering

**Example Files**:
- `spec/policies/family_finding/search_policy_spec.rb`
- `spec/policies/family_finding/clear_search_policy_spec.rb`

**Example**:
```ruby
describe(FamilyFinding::SearchPolicy) do
  describe("#show?") do
    it("allows agency admin to view search") do
      search = create(:family_finding_search, agency: user.agency)
      expect(policy(search).show?).to(be(true))
    end
    
    it("denies access to other agency's search") do
      search = create(:family_finding_search, agency: other_agency)
      expect(policy(search).show?).to(be(false))
    end
  end
end
```

### 6. React Component Tests

**Location**: `app/javascript/components/**/*.spec.js`

**What they test**:
- Component rendering
- User interactions
- Props handling
- State management
- GraphQL query/mutation hooks

**Example Files**:
- `app/javascript/components/family_finding/searches/Searches.spec.js`
- `app/javascript/components/family_finding/relationships/relationshipsColumns.spec.js`

**Example**:
```javascript
describe("Searches", () => {
  it("renders searches table", async () => {
    const { baseElement } = render(
      <MockedProvider>
        <Searches counties={[]} />
      </MockedProvider>
    );
    
    await waitForDynamicLoad(() => 
      isEmpty(queryAllByTestId("table-loading-indicator"))
    );
    
    expect(baseElement).toMatchSnapshot();
  });
  
  it("hides create button without permission", async () => {
    graphqlHelpersPolicyMock({ defaultReturnBool: false });
    
    const { queryByLabelText } = render(
      <MockedProvider>
        <Searches counties={[]} />
      </MockedProvider>
    );
    
    expect(queryByLabelText("Create New")).not.toBeInTheDocument();
  });
});
```

### 7. Utility Function Tests

**Location**: `app/javascript/components/**/*.spec.js` (alongside components)

**What they test**:
- Helper functions
- Data transformations
- Sorting/filtering logic

**Example Files**:
- `app/javascript/components/family_finding/relationships/sortRelationships.spec.js`
- `app/javascript/components/family_finding/relationships/relationshipConversion.spec.js`

**Example**:
```javascript
describe("determineName", () => {
  it("returns name and href for connection", () => {
    const result = determineName({
      keystoneAgencyHumanId: 1,
      destinationAgencyHuman: { id: 1, fullName: "Child" },
      sourceAgencyHuman: { 
        id: 2, 
        fullName: "Parent",
        linkToView: "/people/2"
      }
    });
    
    expect(result.name).toBe("Parent");
    expect(result.href).toBe("/people/2");
  });
  
  it("handles null agency humans gracefully", () => {
    const result = determineName({
      keystoneAgencyHumanId: 1,
      destinationAgencyHuman: null,
      sourceAgencyHuman: null
    });
    
    expect(result.name).toBeUndefined();
    expect(result.href).toBeUndefined();
  });
});
```

### 8. End-to-End (E2E) Tests

**Location**: `spec/playwright/e2e/`

**What they test**:
- Complete user workflows
- Browser interactions
- Full application behavior
- Integration between frontend and backend

**Example Files**:
- `spec/playwright/e2e/family_finding/child_case_status/family_finding_searches_dashboard.spec.ts`

**Example**:
```typescript
test.describe("Family Finding Searches dashboard", () => {
  test("User can view searches and perform actions", async ({
    page,
    session,
    pageObjectModels,
    components
  }) => {
    await session.login(agencyAdmin);
    await pageObjectModels.familyFinding.searches.gotoIndex();
    
    // Verify search appears
    await expect(page.getByText(child.fullName)).toBeVisible();
    
    // Open actions menu
    await components.tables.gotoRowAction({
      rowContent: child.fullName,
      menuButtonLabel: "Actions"
    });
    
    // Verify actions are available
    await expect(
      page.getByRole("menuitem", { name: "Edit Search" })
    ).toBeVisible();
  });
});
```

---

## Test Data Setup

### Ruby: Factories and Shared Contexts

**Factories** (test data builders):
- Location: `spec/factories/`
- Used to create test records
- Example: `create(:family_finding_search)`

**Shared Contexts**:
- Location: `spec/support/shared_contexts/`
- Reusable test setup (users, agencies, etc.)
- Example: `include_context("policy users")`

**Example**:
```ruby
# In shared_contexts/policy_users.rb
shared_context("policy users") do
  let(:family_finding_admin) { create(:agency_worker, :family_finding_admin) }
  let(:binti_admin) { create(:binti_admin) }
  let(:family_finding_agency) { create(:agency, :family_finding) }
end

# In test file
describe(Queries::FamilyFindingSearches) do
  include_context("policy users")
  
  it("works") do
    user = family_finding_admin  # Available from shared context
    # ...
  end
end
```

### JavaScript: Mocks and Fixtures

**GraphQL Mocks**:
- `MockedProvider` from `@apollo/client/testing`
- Mocks GraphQL queries/mutations
- Returns predefined responses

**Component Mocks**:
- `jest.mock()` to mock dependencies
- Example: Mocking `StatusPill` component

**Example**:
```javascript
// Mock a component
jest.mock("@components/child/status_pill/StatusPill", () => ({
  default: ({ status }) => <div>{status}</div>
}));

// Mock GraphQL
const mocks = [{
  request: { query: FamilyFindingSearchesQuery },
  result: { data: { familyFindingSearches: { nodes: [] } } }
}];

render(
  <MockedProvider mocks={mocks}>
    <Searches />
  </MockedProvider>
);
```

### Playwright: Checkpoints

**Checkpoints** (test data setup):
- Pre-configured test scenarios
- Creates users, children, relationships, etc.
- Reusable across tests

**Example**:
```typescript
test.beforeEach(async ({ checkpoints }) => {
  ({ agencyAdmin, child } = await checkpoints
    .familyFinding
    .familyFindingWithConfidentialAndSealedChildren()
  );
});
```

---

## Testing Patterns

### 1. Test Structure (RSpec)

```ruby
describe(ClassName) do                    # What you're testing
  describe("#method_name") do             # Specific method
    context("when condition") do           # Specific scenario
      it("does something") do             # Specific behavior
        # Arrange
        setup_data
        
        # Act
        result = subject.call
        
        # Assert
        expect(result).to(eq(expected))
      end
    end
  end
end
```

### 2. Test Structure (Jest)

```javascript
describe("ComponentName", () => {
  describe("feature", () => {
    it("does something", () => {
      // Arrange
      const props = { ... };
      
      // Act
      const { getByText } = render(<Component {...props} />);
      
      // Assert
      expect(getByText("Expected")).toBeInTheDocument();
    });
  });
});
```

### 3. Test Structure (Playwright)

```typescript
test.describe("Feature Name", () => {
  test.beforeEach(async ({ checkpoints }) => {
    // Setup test data
  });
  
  test("user can do something", async ({ page, session }) => {
    // Arrange
    await session.login(user);
    
    // Act
    await page.goto("/path");
    await page.click("button");
    
    // Assert
    await expect(page.getByText("Success")).toBeVisible();
  });
});
```

---

## Testing Family Finding Features

### Testing Searches Feature

**Ruby Tests**:
```bash
# Service objects
b test-rspec spec/services/family_finding/searches/

# GraphQL queries
b test-rspec spec/graphql/queries/family_finding_searches_spec.rb

# Models
b test-rspec spec/models/family_finding/search_spec.rb
```

**JavaScript Tests**:
```bash
# React components
yarn test app/javascript/components/family_finding/searches/
```

**E2E Tests**:
```bash
yarn playwright test spec/playwright/e2e/family_finding/child_case_status/family_finding_searches_dashboard.spec.ts
```

### Testing Relationships Feature

**Ruby Tests**:
```bash
# Service objects
b test-rspec spec/lib/services/family_finding/relationships/

# GraphQL queries
b test-rspec spec/graphql/queries/agency_human_relationships_spec.rb
```

**JavaScript Tests**:
```bash
# React components
yarn test app/javascript/components/family_finding/relationships/
```

### Testing PKOS Feature

**Ruby Tests**:
```bash
# Controllers
b test-rspec spec/requests/family_finding/potential_kin_search_controller_spec.rb

# Service objects
b test-rspec spec/lib/services/family_finding/potential_kin_searches/

# Models
b test-rspec spec/models/family_finding/clear_person_search_spec.rb
```

**JavaScript Tests**:
```bash
# React components
yarn test app/javascript/components/family_finding/potential_kin_search/
```

---

## Test Helpers and Utilities

### Ruby Test Helpers

**GraphQL Helpers**:
- `graphql_execute(query, current_user: user, variables: {})`
- `graphql_client_query("QueryName")`

**Factory Helpers**:
- `create(:model_name)` - Creates and saves to database
- `build(:model_name)` - Creates but doesn't save
- `build_stubbed(:model_name)` - Creates in memory only

**Policy Helpers**:
- `include_context("policy users")` - Sets up test users
- `policy(resource).action?` - Tests permissions

### JavaScript Test Helpers

**Testing Library**:
- `render()` - Renders React components
- `screen.getByText()` - Finds elements
- `userEvent.click()` - Simulates user interactions

**GraphQL Helpers**:
- `MockedProvider` - Mocks GraphQL queries
- `waitForDynamicLoad()` - Waits for async loading

**Table Utilities**:
- `tableUtilities(user).openFilters()` - Helper for testing tables

### Playwright Helpers

**Page Object Models**:
- `pageObjectModels.familyFinding.searches.gotoIndex()`
- Encapsulates navigation logic

**Components**:
- `components.tables.gotoRowAction()` - Helper for table interactions

**Session**:
- `session.login(user)` - Handles authentication

---

## Best Practices

### 1. Test Organization

- **One test file per source file**: `Searches.js` → `Searches.spec.js`
- **Group related tests**: Use `describe` blocks
- **Clear test names**: Describe what the test verifies

### 2. Test Data

**Ruby**:
- Use factories instead of `FactoryBot.create` for Application models
- Use `GenerateTestApplicationFromTemplate` for Application models
- Use shared contexts for common setup

**JavaScript**:
- Mock external dependencies
- Use `MockedProvider` for GraphQL
- Keep test data minimal and focused

### 3. Test Isolation

- Each test should be independent
- Clean up test data (database transactions roll back automatically in RSpec)
- Don't rely on test execution order

### 4. Assertions

- **Be specific**: Test exact behavior, not just "it works"
- **Test edge cases**: Null values, empty arrays, etc.
- **Test error cases**: What happens when things go wrong?

### 5. Coverage

- Test happy paths (normal operation)
- Test error paths (failures)
- Test edge cases (boundary conditions)
- Test authorization (permissions)

---

## Common Test Scenarios for Family Finding

### 1. Testing Null Handling

**Why**: PKOS can create connections with null data

**Example**:
```javascript
it("handles null agency humans gracefully", () => {
  const result = determineName({
    keystoneAgencyHumanId: 1,
    destinationAgencyHuman: null,
    sourceAgencyHuman: null
  });
  
  expect(result.name).toBeUndefined();
  expect(result.href).toBeUndefined();
});
```

### 2. Testing Authorization

**Why**: Users should only see data they're allowed to see

**Example**:
```ruby
it("returns only searches user can see") do
  user_search = create(:family_finding_search, agency: user.agency)
  other_search = create(:family_finding_search, agency: other_agency)
  
  response = execute_query
  searches = response["data"]["familyFindingSearches"]["nodes"]
  
  expect(searches.length).to(eq(1))
  expect(searches.first["id"]).to(eq(user_search.id.to_s))
end
```

### 3. Testing Data Filtering

**Why**: Searches can be filtered by multiple criteria

**Example**:
```ruby
it("filters by child name") do
  matching_search = create(:family_finding_search, 
    agency_human: create(:agency_human, full_name: "John Doe"))
  other_search = create(:family_finding_search,
    agency_human: create(:agency_human, full_name: "Jane Smith"))
  
  response = execute_query({ child_name_contains: "John" })
  searches = response["data"]["familyFindingSearches"]["nodes"]
  
  expect(searches.length).to(eq(1))
  expect(searches.first["id"]).to(eq(matching_search.id.to_s))
end
```

### 4. Testing Component Rendering

**Why**: Components should render correctly with different data

**Example**:
```javascript
it("renders connection name with link", () => {
  const relationship = {
    destinationAgencyHuman: { id: 1 },
    sourceAgencyHuman: {
      id: 2,
      fullName: "John Doe",
      linkToView: "/people/2"
    }
  };
  
  const column = nameCol({ keystoneAgencyHumanId: 1 });
  const { getByText } = render(column.cell(relationship));
  
  expect(getByText("John Doe")).toHaveAttribute("href", "/people/2");
});
```

### 5. Testing User Interactions

**Why**: Users interact with the UI in various ways

**Example**:
```javascript
it("updates filters when tab changes", async () => {
  const user = userEvent.setup();
  const { getByTitle } = render(<Searches />);
  
  const closedTab = getByTitle("Closed");
  await user.click(closedTab);
  
  const params = getBase64SearchParams();
  expect(params.query.status).toBe("closed");
});
```

---

## Running All Family Finding Tests

### Quick Commands

**All Ruby tests for Family Finding**:
```bash
b test-rspec spec/services/family_finding/ spec/graphql/queries/family_finding* spec/models/family_finding/ spec/policies/family_finding/ spec/requests/family_finding/
```

**All JavaScript tests for Family Finding**:
```bash
yarn test app/javascript/components/family_finding/
```

**All E2E tests for Family Finding**:
```bash
PLAYWRIGHT_HTML_OPEN=never yarn playwright test --config spec/playwright.config.ts spec/playwright/e2e/family_finding/
```

---

## Test Coverage Goals

### What Should Be Tested

1. **Service Objects**: All business logic
2. **GraphQL Queries/Mutations**: Authorization, filtering, data shape
3. **React Components**: Rendering, user interactions, state changes
4. **Models**: Validations, associations, methods
5. **Policies**: All permission checks
6. **Controllers**: Authorization, routing
7. **Utilities**: Helper functions, data transformations

### What Doesn't Need Tests

1. **Simple getters/setters**: Basic attribute access
2. **Framework code**: Rails/React internals
3. **Third-party libraries**: Test your usage, not the library
4. **Trivial code**: Code that's obviously correct

---

## Debugging Tests

### Ruby/RSpec

**Run with verbose output**:
```bash
b test-rspec spec/path/to/file_spec.rb --format documentation
```

**Run with debugger**:
```ruby
it("debugs something") do
  binding.pry  # Opens debugger
  # ...
end
```

### JavaScript/Jest

**Run with verbose output**:
```bash
yarn test --verbose
```

**Run specific test**:
```bash
yarn test -t "test name"
```

**Debug in Node**:
```bash
node --inspect-brk node_modules/.bin/jest --runInBand
```

### Playwright

**Run with UI mode** (shows browser):
```bash
yarn playwright test --ui
```

**Run with headed browser**:
```bash
yarn playwright test --headed
```

**Debug mode**:
```bash
yarn playwright test --debug
```

---

## Summary

The Family application uses a comprehensive testing strategy:

1. **RSpec** for Ruby/Rails (models, services, GraphQL, controllers)
2. **Jest** for JavaScript/React (components, utilities)
3. **Playwright** for E2E (complete user workflows)

**For Family Finding features**:
- Test service objects (business logic)
- Test GraphQL queries/mutations (data access, authorization)
- Test React components (UI, interactions)
- Test models (validations, associations)
- Test policies (authorization)
- Test E2E workflows (complete user journeys)

**Key Commands**:
- Ruby: `b test-rspec spec/path/to/file_spec.rb`
- JavaScript: `yarn test path/to/file.spec.js`
- E2E: `yarn playwright test spec/playwright/e2e/path/to/file.spec.ts`

**Best Practices**:
- Write tests for all new code
- Test edge cases (null values, empty data)
- Test authorization thoroughly
- Keep tests isolated and independent
- Use factories/mocks for test data
