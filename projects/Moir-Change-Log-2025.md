# Moir Change Log 2025

This document tracks all JIRA tickets and Pull Requests that I push through to merge and deployment.

---

## ENG-24262: Blank Screen When Editing Relationships (Null Data Handling)

### JIRA
- **Ticket**: [ENG-24262](https://binti.atlassian.net/browse/ENG-24262)
- **Title**: Blank screen when editing relationships / connections added via PKOS

### Pull Request
- **PR**: #19791
- **Branch**: `ENG-24262`
- **PR URL**: https://github.com/binti-family/family/pull/19791

### Purpose
Fix blank pages and missing hyperlinks when viewing relationships for connections created via Potential Kin Online Search (PKOS). The root cause was null/incomplete data being stored and returned, which the UI was not prepared to handle gracefully.

### Problem
When linking potential connections via PKOS (`/family_finding/potential_kin_searches/new`), the system could create `AgencyHuman` records with null or incomplete data (e.g., missing names, null `socialMediaLinks`). This caused:

1. **GraphQL Error**: "Cannot return null for non-nullable field AgencyHuman.socialMediaLinks"
2. **Blank Pages**: UI components crashed when trying to display relationships with null agency human data
3. **Missing Hyperlinks**: Relationship names were undefined, causing broken links

The issue stemmed from PKOS allowing partial data to be saved when `skipInvalidDataErrors: true` was used, which could create records without critical fields.

### Solution
Implemented a multi-layered defense-in-depth approach:

1. **GraphQL Type Fix**: Added custom resolver for `socialMediaLinks` in `AgencyHuman` type to always return an array (never null)
2. **Frontend Defensive Handling**: Added null checks and fallbacks in React components to gracefully handle missing data
3. **Relationship Validation**: Added validation in `Services::Relationships::Create` to ensure agency humans exist before creating relationships
4. **GraphQL Schema Update**: Made `source_agency_human` and `destination_agency_human` nullable in Relationship type to handle edge cases

### Files Affected
- `app/graphql/types/agency_human.rb` - Added `socialMediaLinks` resolver to return empty array instead of null
- `app/graphql/types/relationship.rb` - Made `source_agency_human` and `destination_agency_human` nullable
- `app/javascript/components/agency_humans/AgencyHumanSummary.js` - Added defensive null check for `socialMediaLinks`
- `app/javascript/components/agency_humans/agencyHumanConversion.js` - Added defensive null check with JSDoc
- `app/javascript/components/family_finding/contact_logs/AgencyHumansInputWithContactInfo.js` - Added defensive null check
- `app/javascript/components/family_finding/relationships/RelationshipActions.js` - Added fallback empty strings for name/keystoneName
- `app/javascript/components/family_finding/relationships/relationshipsColumns.js` - Added fallback empty string for name
- `app/javascript/components/family_finding/relationships/sortRelationships.js` - Fixed boolean handling for `isDeceased` (preserve `false` values, only default `null`/`undefined` to `undefined`)
- `app/lib/services/relationships/create.rb` - Added validation to ensure agency humans exist before creating relationships
- `app/graphql/mutations/create_or_update_agency_human.rb` - Moved `agency_human.reload` to occur before relationship creation

### Timestamps
- **PR Created**: 2025-01-XX (TBD)
- **PR Merged**: 2025-01-XX (TBD - requires approval)
- **Deployed**: 2025-01-XX (TBD)

### Kudos and Feedback
- **AI Assistant (Cursor)**: Provided comprehensive analysis, root cause identification, and implementation guidance throughout the investigation and fix
- **Code Reviewers**: (TBD - awaiting PR review)

### Notes
- Fixed unit test failure in `sortRelationships.spec.js` where `isDeceased: false` was incorrectly being returned as `undefined`
- Addressed Pronto lint warning about `MutationResolveTooLong` by adding `PERMIT_LINT_WARNINGS_FOR_REFACTOR` flag to PR description (pre-existing issue, not introduced by this change)

---

## ENG-24390: Hide Browser Warning Banner When Printing Bulk Outreach Letters

### JIRA
- **Ticket**: [ENG-24390](https://binti.atlassian.net/browse/ENG-24390)
- **Title**: Hide unsupported browser warning banner when printing family finding bulk outreach letters

### Pull Request
- **PR**: #TBD (to be updated when PR number is available)
- **Branch**: `ENG-24390`
- **PR URL**: https://github.com/binti-family/family/pull/TBD

### Purpose
Prevent the unsupported browser warning banner (`.page-notification`) from appearing when printing family finding bulk outreach letters. The banner was being included in print output, which is undesirable for official correspondence.

### Problem
When generating and printing family finding bulk outreach letters via the print channel, the unsupported browser warning banner (rendered by `app/views/admin/_supported_browser_check.erb`) was appearing in the printed output. This banner is useful for on-screen display but should not be included in printed documents.

The existing print stylesheet (`family_finding_bulk_outreach.scss`) already had print media queries to hide other UI elements (Zendesk chat button, updates sidebar, etc.), but the browser warning banner was not included.

### Solution
Added a CSS rule in the existing `@media print` block to hide the `.page-notification` class (which includes the browser warning banner) when printing. Used `!important` to ensure the banner is hidden even if other styles might override it, matching the pattern established for other print-specific styles in the same file.

Also added a Playwright E2E test to verify the banner is hidden in print mode, using `page.emulateMedia({ media: 'print' })` to simulate print media and verify the CSS rule is applied correctly.

### Files Affected
- `app/assets/stylesheets/family_finding_bulk_outreach.scss` - Added `.page-notification { display: none !important; }` in `@media print` block with appropriate lint disable comment
- `spec/playwright/e2e/family_finding/bulk_outreach/print_channel.spec.ts` - Added test case "should hide unsupported browser warning banner when printing" (later removed per user request)

### Timestamps
- **PR Created**: 2025-12-22
- **PR Merged**: 2025-12-22 (approved by Piper)
- **Deployed**: 2025-12-XX (TBD)

### Kudos and Feedback
- **Piper**: Approved PR and provided code review feedback
- **AI Assistant (Cursor)**: Provided analysis of the problem, identified the solution, and helped implement the CSS fix and Playwright test
- **AI Reviewer Bot**: Provided feedback on Playwright test best practices (use `components.forms.select()` instead of native `selectOption`, use `page.getByText()` instead of raw locators)

### Notes
- Initially added a Playwright test to verify the banner is hidden, but the test was later removed from the final PR
- Addressed AI Reviewer Bot comments regarding SCSS linting rules and Playwright best practices
- Used `!important` to match existing pattern in the file for print-specific styles

---

## Template for Future Entries

```markdown
## JIRA-XXXXX: Brief Title

### JIRA
- **Ticket**: [JIRA-XXXXX](https://binti.atlassian.net/browse/JIRA-XXXXX)
- **Title**: Full ticket title

### Pull Request
- **PR**: #XXXXX
- **Branch**: `branch-name`
- **PR URL**: https://github.com/binti-family/family/pull/XXXXX

### Purpose
Brief description of what this change accomplishes.

### Problem
Detailed description of the problem being solved.

### Solution
Detailed description of the solution implemented.

### Files Affected
- `path/to/file1.rb` - Description of change
- `path/to/file2.js` - Description of change

### Timestamps
- **PR Created**: YYYY-MM-DD
- **PR Merged**: YYYY-MM-DD
- **Deployed**: YYYY-MM-DD

### Kudos and Feedback
- **Person/Team**: Description of their contribution

### Notes
Any additional notes, follow-up items, or related information.
```
