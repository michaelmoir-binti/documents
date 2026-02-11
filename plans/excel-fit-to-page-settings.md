# Excel "Fit to Page" Settings in Family Finding Reports

## Location

The "Fit to Page" print setting is configured in:

**`app/lib/services/family_finding/private/generate_efforts_report_excel.rb`**

## Where It's Set

The `fit_to` method is called on each worksheet's `page_setup`:

```ruby
sheet.page_setup.fit_to(width: 1, height: 1)
```

This appears in multiple places:
- **Line 107** - Summary sheet
- **Line 380** - Connection sheets (Relatives, Fictive Kin, Potential Connections)
- **Line 421** - Contact Log sheet

## What This Setting Does

The `fit_to(width: 1, height: 1)` call tells Excel to scale the worksheet to fit on exactly **1 page wide by 1 page tall** when printing. This is the "Fit to Page" setting visible in Excel's print preview.

## Available Options

This uses the [Axlsx gem](https://github.com/randym/axlsx) (also known as `caxlsx`), which provides full control over page setup options.

### Option 1: Remove the fit_to setting entirely
Remove the `sheet.page_setup.fit_to(width: 1, height: 1)` lines to use default print settings.

### Option 2: Fit to width only (let height be automatic)
```ruby
sheet.page_setup.fit_to(width: 1)  # Fit to 1 page wide, unlimited height
```

### Option 3: Use scaling percentage instead
```ruby
sheet.page_setup.scale = 75  # Scale to 75% of normal size
```

### Option 4: Set specific page orientation and margins
```ruby
sheet.page_setup do |setup|
  setup.orientation = :landscape  # or :portrait
  setup.paper_size = Axlsx::PAPERSIZE_LETTER  # or PAPERSIZE_A4
  setup.fit_to_width = 1
  setup.fit_to_height = 0  # 0 means automatic/unlimited
end
```

### Option 5: Disable fit_to and use default printing
Simply remove the `fit_to` call and the worksheet will print at 100% scale with natural page breaks.

## References

- [Axlsx PageSetup documentation](https://www.rubydoc.info/gems/caxlsx/Axlsx/PageSetup)
- [Axlsx gem on GitHub](https://github.com/caxlsx/caxlsx)
