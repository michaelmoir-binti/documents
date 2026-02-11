# Bulk Outreach Letters Print Margin Control

## Problem

The Bulk Outreach Letters print preview shows asymmetric margins. Currently:
- No dedicated `@page` rule exists in `family_finding_bulk_outreach.scss`
- The global `print.scss` `@page` rule is applied (1.5in top, 1in right/bottom/left)
- Content has a fixed `width: 700px` that doesn't relate to the actual printable area
- No centering mechanism for the content within the printable area

## Solution

Add a dedicated `@page` rule with SCSS variables at the top of `family_finding_bulk_outreach.scss` for centralized, easy-to-tune margin control.

## Implementation

### File: `app/assets/stylesheets/family_finding_bulk_outreach.scss`

**1. Add SCSS variables at the top of the file (after imports, around line 10):**

```scss
// =============================================================================
// BULK OUTREACH PRINT MARGIN CONFIGURATION
// =============================================================================
// Adjust these values to control print margins for Bulk Outreach Letters.
// These override the global print.scss @page margins for this specific feature.
// Standard US Letter paper: 8.5in x 11in
// =============================================================================
$ff-boc-print-margin-top: 1in;
$ff-boc-print-margin-right: 1in;
$ff-boc-print-margin-bottom: 1in;
$ff-boc-print-margin-left: 1in;
```

**2. Add a dedicated `@page` rule within the `@media print` block (after line 27):**

```scss
@media print {
  // Override global @page margins for Bulk Outreach Letters
  @page {
    size: 8.5in 11in;
    margin-top: $ff-boc-print-margin-top;
    margin-right: $ff-boc-print-margin-right;
    margin-bottom: $ff-boc-print-margin-bottom;
    margin-left: $ff-boc-print-margin-left;
  }

  // existing rules...
}
```

**3. Update `.family_finding_bulk_outreach_content` to use full width and center content:**

```scss
.family_finding_bulk_outreach_content {
  // ... existing styles ...

  @media print {
    margin: 0 auto;  // Center the content horizontally
    padding: 0;      // Remove padding, let @page margins handle spacing
  }

  &.print-channel {
    // ... existing styles ...
    width: 100%;     // Use full printable width instead of fixed 700px
    max-width: 6.5in; // Optional: constrain to reasonable width (8.5in - 2in margins)
    box-sizing: border-box;
  }
}
```

## Key Changes Summary

| Location | Change |
|----------|--------|
| Lines 10-20 | Add SCSS variables for margin configuration |
| Lines 11-27 (inside `@media print`) | Add dedicated `@page` rule |
| Line 50 | Change `margin: 0` to `margin: 0 auto` for centering |
| Line 76 | Change `width: 700px` to `width: 100%` with optional `max-width` |

## Testing Plan

1. Navigate to Family Finding > Bulk Outreach Campaigns
2. Select a campaign with multiple recipients
3. Click "Print" to open print preview
4. Verify:
   - Left and right margins appear equal
   - Content is centered on the page
   - Margins can be adjusted by changing the SCSS variables
5. Test with different browsers (Chrome, Firefox, Safari) as print rendering varies

## Files Involved

- `app/assets/stylesheets/family_finding_bulk_outreach.scss` - Main changes
- `app/assets/stylesheets/print.scss` - Global print styles (reference only, no changes)
- `app/views/family_finding/bulk_outreach_campaigns/print.html.erb` - Print view template (no changes needed)
