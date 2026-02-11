# Genogram Print Font Size Investigation

## Problem

Workers who print the Genogram are seeing the font-size as very small.

## Root Cause Analysis

The Genogram uses **React Flow** (`@xyflow/react`) to render an interactive diagram. The small font size when printing is caused by multiple factors:

### 1. Download/Export Method (Primary Cause)

**File:** `app/javascript/heart/components/node_diagram/DownloadButton.js`

The download button uses `html-to-image` library to capture the diagram as a PNG:

```javascript
toPng(document.querySelector(".react-flow__viewport"), {
  backgroundColor: "#ffffff",
  width: nodesBounds.width + 100,
  height: nodesBounds.height + 100,
  style: {
    width: nodesBounds.width,
    height: nodesBounds.height,
    transform: "none",
  },
})
```

The image is captured at the **current viewport size**. If the genogram is zoomed out to fit many nodes, the text appears small in the downloaded image.

### 2. Fixed Node Dimensions

**File:** `app/javascript/components/family_finding/genogram/Genogram.js` (lines 422-423)

```javascript
nodeHeight={170}
nodeWidth={200}
```

These fixed dimensions control how much space each node has. Text must fit within these bounds.

### 3. Text Styling in Nodes

**File:** `app/javascript/components/family_finding/genogram/Connection.js` (lines 34-36)

```javascript
<Text textStyle="emphasis-100">{name}</Text>
<Text textStyle="body-100">
  {determineRelationshipTypeForTable(relationship)}
</Text>
```

Font sizes are controlled by Heart's `Text` component using `textStyle` props.

### 4. Container Constraints

**File:** `app/javascript/components/family_finding/genogram/Connection.module.scss`

```scss
.container {
  max-width: 200px;
  text-align: center;
}
```

### 5. No Print-Specific Styles

There are **no `@media print` styles** for genogram components. When users print directly (Ctrl+P), the genogram renders at whatever zoom level it's currently at.

---

## Proposed Solutions

### Option 1: Scale Up Downloaded Image (Recommended)

**File to modify:** `app/javascript/heart/components/node_diagram/DownloadButton.js`

Add a scale factor to the `toPng` call:

```javascript
const DOWNLOAD_SCALE = 2; // or 3 for even larger

toPng(document.querySelector(".react-flow__viewport"), {
  backgroundColor: "#ffffff",
  width: (nodesBounds.width + 100) * DOWNLOAD_SCALE,
  height: (nodesBounds.height + 100) * DOWNLOAD_SCALE,
  style: {
    width: nodesBounds.width * DOWNLOAD_SCALE,
    height: nodesBounds.height * DOWNLOAD_SCALE,
    transform: `scale(${DOWNLOAD_SCALE})`,
    transformOrigin: "top left",
  },
})
```

**Pros:** Simple change, affects only downloaded images
**Cons:** Larger file sizes

### Option 2: Increase Node Dimensions

**File to modify:** `app/javascript/components/family_finding/genogram/Genogram.js`

```javascript
// Current
nodeHeight={170}
nodeWidth={200}

// Proposed
nodeHeight={220}
nodeWidth={260}
```

**Pros:** Larger nodes = more room for text
**Cons:** Affects on-screen display, may require more scrolling/zooming

### Option 3: Use Larger Text Styles

**File to modify:** `app/javascript/components/family_finding/genogram/Connection.js`

```javascript
// Current
<Text textStyle="emphasis-100">{name}</Text>
<Text textStyle="body-100">

// Proposed
<Text textStyle="emphasis-200">{name}</Text>
<Text textStyle="body-200">
```

**Pros:** Directly addresses font size
**Cons:** May cause text overflow in nodes

### Option 4: Add Print-Specific CSS

**File to modify:** `app/javascript/components/family_finding/genogram/Connection.module.scss`

```scss
@media print {
  .container {
    font-size: 14pt !important;
  }
  
  // Force larger text in print
  :global(.react-flow__node) {
    transform: scale(1.5) !important;
  }
}
```

**Pros:** Only affects print output
**Cons:** May cause layout issues, doesn't help with downloaded PNG

### Option 5: Add Configurable Scale to Download Button

**File to modify:** `app/javascript/heart/components/node_diagram/DownloadButton.js`

Add a `downloadScale` prop to `NodeDiagram` that gets passed to `DownloadButton`:

```javascript
const DownloadButton = ({ fileName, scale = 2 }) => {
  // Use scale in toPng options
}
```

**Pros:** Flexible, can be adjusted per use case
**Cons:** More complex implementation

---

## Recommended Implementation

**Option 1 (Scale Up Downloaded Image)** is recommended because:

1. It's a minimal change (single file)
2. It only affects the download output, not the interactive UI
3. It's a common pattern for high-DPI/print-quality exports
4. No risk of breaking existing layout

### Files to Modify

1. `app/javascript/heart/components/node_diagram/DownloadButton.js` - Add scale factor
2. `app/javascript/heart/components/node_diagram/DownloadButton.spec.js` - Update tests if needed

### Testing Plan

1. Generate a genogram with multiple connections
2. Download the image using the download button
3. Print the downloaded image and verify font size is readable
4. Test with both small (few nodes) and large (many nodes) genograms
