# Containers, Sections, Blocks, Divs -- Layout System

The four nestable layout elements (`section`, `container`, `block`, `div`) share a control set inherited from `Element_Container`. The differences are mostly **default tag** and **default children**:

| Element | Default tag | Default children when added | Vue component |
|---|---|---|---|
| `section` | `<section>` | one `container` child (so the canonical pattern is section > container > content) | `bricks-nestable` |
| `container` | `<div>` (or `tag` setting) | none (empty) | `bricks-nestable` |
| `block` | `<div>` | none | `bricks-nestable` |
| `div` | `<div>` | none | `bricks-nestable` |

All four accept `tag` (a select with `section`, `header`, `footer`, `aside`, `article`, `nav`, `main`, `div`, `custom`) and `customTag` (text). Set `tag: 'custom'` to use `customTag`.

## Choosing the right one

- **`section`** for top-level page slices (Hero, Features, Pricing, Footer-CTA). Renders semantic `<section>`.
- **`container`** as the inner constrained-width wrapper inside a section. Holds your columns/grid.
- **`block`** for repeating content blocks inside a container (cards, list items).
- **`div`** for the lightest wrapping when semantics don't matter (icon stacks, layout helpers).

The convention you'll see in well-built Bricks sites:
```
section (full-width bg)
  +-- container (max-width inner, flex/grid)
        +-- block / div / heading / icon-box / --
        +-- --
```

## Layout controls

All values below are saved into `settings`. Below `_direction` is a `direction` control (string), `_alignItems` etc. are alignment controls (string), `_columnGap`/`_rowGap` are number controls with units.

### Flex (default `_display: 'flex'`)

```jsonc
"_display":        "flex",
"_direction":      "row",                 // row | column | row-reverse | column-reverse
"_flexWrap":       "nowrap",              // nowrap | wrap | wrap-reverse
"_alignItems":     "center",              // flex-start | center | flex-end | stretch | baseline
"_justifyContent": "space-between",       // flex-start | center | flex-end | space-between | space-around | space-evenly
"_alignContent":   "stretch",             // for wrap rows
"_gap":            "20px",
"_columnGap":      "20px",
"_rowGap":         "20px"
```

**Alignment cheat-sheet (row direction):**
- Center horizontally + vertically: `_direction: row`, `_justifyContent: center`, `_alignItems: center`.
- Stretch evenly across width: `_justifyContent: space-between` and `_flexWrap: wrap` if items overflow.

### Grid

Set `_display: 'grid'` and use:
```jsonc
"_display":             "grid",
"_gridTemplateColumns": "repeat(3, 1fr)",   // any valid CSS string
"_gridTemplateRows":    "auto",
"_gridAutoFlow":        "row",              // row | column | dense
"_gridAutoColumns":     "1fr",
"_gridAutoRows":        "auto",
"_gridGap":             "24px",             // shorthand for both gaps
"_columnGap":           "24px",
"_rowGap":              "24px",
"_alignItems":          "stretch",          // grid items vertical
"_justifyContent":      "start"             // grid items horizontal (track alignment)
```

Per-item placement (set on the **child** element):
```jsonc
"_gridItemColumnSpan": "span 2",
"_gridItemRowSpan":    "span 1",
"_gridItemJustifySelf":"center"
```

### Legacy `_columnCount` (avoid in new work)

Some older elements still use `_columnCount` (number 1--12) -- Bricks generates a 12-col flex grid. Prefer `_display: grid` instead.

### Masonry (image gallery, posts, etc.)

```jsonc
"_useMasonry":                 true,
"_masonryColumn":              3,
"_masonryGutter":              "16px",
"_masonryHorizontalOrder":     true,
"_masonryTransitionDuration":  "0.4s",
"_masonryTransitionMode":      "default"
```
Only elements with `support_masonry = true` will respect these. Confirm at the source -- `Element::support_masonry_element()`.

## Container-only controls

Container has these on top of the shared layout set:

```jsonc
"link":               { "type": "internal", "url": "/case-studies" },
"disableLinkOnClick": false,
"tag":                "section",
"customTag":          "main",
"_sticky":            true,
"_stickyOffset":      "0px",
"_stickyAtBreakpoint":"tablet_portrait",
"_stickyTopOffset":   "80px",
"_overlay": { /* repeater of overlay layers */ },
"_shapeDividers": [ { /* per-divider config */ } ]
```

When `link` is set, the container wraps its content in a clickable wrapper (or renders a stretched-link `<a>` inside). `disableLinkOnClick: true` disables that wrapper at click-time (useful for nested clickable items).

## Backgrounds (full set)

Containers/sections/blocks/divs support **multiple background layers** through `_background`. The standard non-repeater shape (in `references/element-base-controls.md`) is the first/only layer. Multi-layer is saved under `_background.layers` (array). Background image+video can coexist (video plays on top).

## Sticky containers

Use `_position: sticky` + `_top: "0"` + sufficient parent height for the effect to work. Bricks adds the sticky CSS but cannot make a flex child sticky unless its parent layout permits it.

## Aspect ratio

```jsonc
"_aspectRatio": "16/9"
```
Pairs nicely with `_overflow: hidden` and `_background.image`/`_background.video` for video-fill panels.

## Children defaults (custom elements)

If you write a custom nestable element and want it to ship with default children (like `section` ships with a `container`), implement `get_nestable_children()`:

```php
public function get_nestable_children() {
  return [
    [ 'name' => 'container', 'settings' => [], 'children' => [] ],
  ];
}
```
And `get_nestable_item()` for what a "new repeater item" looks like (used by accordions/tabs/sliders nested variants).

## When to nest vs flatten

- Each nestable element adds a DOM wrapper. Don't add unnecessary div/blocks if you can do the layout on the existing container.
- For complex grids with overlapping items, prefer one grid container with grid-area placements over multiple nested flex containers.
- Sliders/tabs/accordions need the **nested** variant (e.g. `slider-nested`) when content needs to be Bricks elements rather than plain text -- the basic `slider`/`tabs`/`accordion` use repeater items with limited content fields.
