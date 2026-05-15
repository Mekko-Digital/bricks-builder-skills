# Base Element Common Controls

Every element extends `\Bricks\Element` and inherits a large set of underscore-prefixed (`_`) settings keys defined in `{template_dir}/includes/elements/base.php`. These are the keys you set in `settings` for any element to control its layout, spacing, typography, background, etc.

**Convention:** keys starting with `_` are base-class controls. Element-specific controls usually do **not** start with underscore (e.g. `text`, `icon`, `link`, `tag`).

## Spacing

```jsonc
"_margin":  { "top": "20px", "right": "0", "bottom": "20px", "left": "0" },
"_padding": { "top": "60px", "right": "20px", "bottom": "60px", "left": "20px" }
```
Units: `px`, `em`, `rem`, `%`, `vw`, `vh`, `ch`, `svh`, `lvh`, `dvh`. Single values like `"60px"` are also accepted by some controls but the four-side object is the canonical shape.

## Typography

```jsonc
"_typography": {
  "font-family":     "Inter",
  "font-size":       "18px",
  "font-weight":     "500",
  "line-height":     "1.5",
  "letter-spacing":  "0",
  "text-align":      "left",
  "text-transform":  "uppercase",
  "text-decoration": "none",
  "color":           { "hex": "#111111" }
}
```
- `font-family`: any local/Adobe/Google/custom-font registered name. Bricks loads Google Fonts on demand.
- `color` always wraps in `{ hex }` / `{ rgb }` / `{ raw: "var(--...)" }`.
- For fluid type, use CSS `clamp()` directly: `"font-size": "clamp(1rem, 2vw, 1.5rem)"`.

## Background

```jsonc
"_background": {
  "color": { "hex": "#0f172a" },
  "image": { "id": 123, "url": "https://.../hero.webp", "filename": "hero.webp" },
  "video": "https://.../bg.mp4",
  "gradient": { "type": "linear", "angle": 135, "stops": [
    { "color": { "hex": "#0f172a" }, "stop": "0%" },
    { "color": { "hex": "#1e293b" }, "stop": "100%" }
  ]},
  "position":   "center center",
  "size":       "cover",
  "repeat":     "no-repeat",
  "attachment": "scroll",
  "parallax":   false
}
```
Layout elements (section/container/block/div) additionally support multiple background layers via a repeater.

## Border + radius

```jsonc
"_border": {
  "width":  { "top": "1px", "right": "1px", "bottom": "1px", "left": "1px" },
  "style":  "solid",
  "color":  { "hex": "#e5e7eb" },
  "radius": { "top": "8px", "right": "8px", "bottom": "8px", "left": "8px" }
}
```
`style`: `solid|dashed|dotted|double|groove|ridge|inset|outset|none`.

## Box shadow

```jsonc
"_boxShadow": {
  "values": { "offsetX": "0", "offsetY": "10px", "blur": "20px", "spread": "0" },
  "color":  { "hex": "rgba(0,0,0,0.1)" },
  "inset":  false
}
```

## Sizing

```jsonc
"_width":    "100%",
"_widthMin": "0",
"_widthMax": "1200px",
"_height":   "auto",
"_heightMin":"100vh",
"_heightMax":"none",
"_aspectRatio": "16/9"
```

## Position & stack

```jsonc
"_position": "relative",
"_top":     "0",
"_right":   "auto",
"_bottom":  "auto",
"_left":    "0",
"_zIndex":  10
```
`_position`: `static|relative|absolute|fixed|sticky`.

## Display, visibility, flow

```jsonc
"_display":       "flex",
"_visibility":    "visible",
"_overflow":      "hidden",
"_opacity":       0.9,
"_cursor":        "pointer",
"_isolation":     "isolate",
"_mixBlendMode":  "multiply",
"_pointerEvents": "auto"
```

## Flex item / grid item

When the element is a child of a flex/grid layout container:

```jsonc
"_flexGrow":   1,
"_flexShrink": 0,
"_flexBasis":  "auto",
"_alignSelf":  "stretch",
"_order":      0,
"_gridItemColumnSpan": "span 2",
"_gridItemRowSpan":    "span 1",
"_gridItemJustifySelf":"center"
```

## Transform & 3D

```jsonc
"_transform": {
  "translate": { "x": "0", "y": "-10px" },
  "rotate":    "0deg",
  "scale":     { "x": 1, "y": 1 },
  "skew":      { "x": "0deg", "y": "0deg" }
},
"_transformOrigin": "center center",
"_perspective":     "1000px",
"_perspectiveOrigin": "center center"
```

## CSS classes, IDs, raw CSS

```jsonc
"_cssId":            "hero-headline",          // no '#'
"_cssClasses":       "u-mb-32 has-shadow",      // space-separated, no dots
"_cssGlobalClasses": [ "clsAbc123", "clsXyz9" ],// IDs from bricks_global_classes
"_cssCustom":        "%root% { will-change: transform; } %root%:hover .icon { transform: scale(1.1); }",
"_cssFilters":       [ { "type": "blur", "value": "0px" }, { "type": "brightness", "value": "100" } ],
"_cssTransition":    "all 0.3s ease"
```
- `%root%` in `_cssCustom` resolves to the element's `.brxe-{id}` selector at render.
- `_cssGlobalClasses` are class **IDs** (the `id` field on a record in `bricks_global_classes`), not class names.

## HTML attributes (custom)

```jsonc
"_attributes": [
  { "name": "data-track", "value": "hero-cta" },
  { "name": "aria-label", "value": "Open menu" }
]
```
By default rendered on element root. Some elements (e.g. `nav-menu`) attach to a child instead -- see element source.

## Interactions

```jsonc
"_interactions": [ /* see references/interactions-and-animations.md */ ]
```

## Conditions

```jsonc
"_conditions": [ /* see references/conditions.md */ ]
```

## Motion (parallax)

```jsonc
"_motionElementParallax":      true,
"_motionElementParallaxSpeedX": 50,
"_motionElementParallaxSpeedY": 50,
"_motionBackgroundParallax":    true,
"_motionBackgroundParallaxSpeed": 30,
"_motionStartVisiblePercent":   0
```
Speeds are percentages, rendered to CSS custom properties: `--brx-motion-parallax-speed-x|y`, `--brx-motion-background-speed`. Implementation respects `prefers-reduced-motion`.

## Responsive override syntax

Every key above can be suffixed with `:{breakpoint_key}` and stored at the same flat level. Example:

```jsonc
"_padding":                          { "top": "100px", "right": "0", "bottom": "100px", "left": "0" },
"_padding:tablet_portrait":          { "top": "60px",  "right": "20px","bottom": "60px","left": "20px" },
"_padding:mobile_portrait":          { "top": "40px",  "right": "16px","bottom": "40px","left": "16px" },
"_typography":                       { "font-size": "48px" },
"_typography:tablet_portrait":       { "font-size": "32px" },
"_typography:mobile_portrait":       { "font-size": "24px" }
```

The breakpoint key is the literal `key` field from a row in `bricks_breakpoints` option (defaults: `desktop`, `tablet_portrait`, `mobile_landscape`, `mobile_portrait`). Custom breakpoints use whatever key you registered (lowercase + underscores).

**Cascade direction depends on `is_mobile_first`:**
- Default (desktop-first, max-width media queries): `desktop` is base, smaller breakpoints **override on smaller screens**.
- Mobile-first (set a small bp as `base`): smallest is base, larger breakpoints **override on larger screens** via min-width.

## Per-pseudo-state overrides

Bricks supports pseudo-classes (`:hover`, `:focus`, `:active`, `:first-child`, custom states). The pseudo state is selected in the panel and saves a duplicated key with the active state stored in `bricks_global_pseudo_classes` option. Generated CSS stacks them. You typically don't write these by hand -- but recognize that hover-only `_background:hover` style entries may exist in older data.

## Set-by-element-source: `set_common_control_groups()`

The base class registers these tabs/groups for every element (visible in the panel):
- **Content** -- element-specific controls and a "Content" tab plus `_basic` group.
- **Style** -- typography, background, border, shadow, layout, transform, sizing, spacing, motion.
- **Custom** -- CSS, attributes, interactions, conditions, performance.

When writing a custom element, your `set_control_groups()` and `set_controls()` only add **element-specific** controls -- the base class fills in the rest.

## Performance

```jsonc
"_lazyLoad":         true,        // lazy-load this element if it has a video/image
"_disableLazyLoad":  false,
"_disableLoop":      false,       // exclude from query loop iteration
"_renderInView":     true         // defer render until in viewport (where supported)
```

These are not on every element -- confirm against `set_controls()` of the specific element file when in doubt.
