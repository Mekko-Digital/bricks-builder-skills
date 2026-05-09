# Theme Styles, Global Classes, Variables, Color Palette, Components

These are the **site-wide** layers that override defaults and feed every element.

## 1. Theme Styles (`bricks_theme_styles` option)

Source: `{template_dir}/includes/theme-styles.php` and `{template_dir}/includes/theme-styles/controls\*.php`.

Theme styles are sets of element defaults that apply by **conditions** (post type, template type, archive, etc.). The first theme style whose conditions match the current page wins.

### Shape

```jsonc
[
  {
    "id":         "ts_abc123",
    "label":      "Default",
    "settings":   {
      // "general" group
      "siteLayout":           "wide",
      "siteLayoutBoxedMaxWidth": "1440px",

      // "colors" group
      "colorsPrimary":   { "hex": "#0ea5e9" },
      "colorsSecondary": { "hex": "#1e293b" },

      // "links"
      "linkColor":       { "hex": "#0ea5e9" },
      "linkColorHover":  { "hex": "#0284c7" },

      // "typography" -- site-level font
      "typographyBody":   { "font-family":"Inter","font-size":"16px","line-height":"1.6","color":{"hex":"#0f172a"} },
      "typographyH1":     { "font-size":"56px","font-weight":"700","line-height":"1.1" },
      "typographyH2":     { "font-size":"40px","font-weight":"700","line-height":"1.2" },
      "typographyH3":     { "font-size":"30px","font-weight":"600","line-height":"1.25" },
      "typographyH4":     { "font-size":"24px","font-weight":"600","line-height":"1.3" },
      "typographyH5":     { "font-size":"20px","font-weight":"600","line-height":"1.4" },
      "typographyH6":     { "font-size":"18px","font-weight":"600","line-height":"1.4" },

      // "content" -- vertical rhythm
      "contentMargin": { "top": "0", "bottom": "1rem" },

      // Per-element groups (one per Bricks element with theme-style support)
      "element-button":  { /* see element-button.php controls */ },
      "element-form":    { /* see element-form.php controls */ },
      "element-heading": { /* ... */ },
      "element-container":{ /* ... */ },
      "element-section": { /* ... */ },
      "element-block":   { /* ... */ },
      "element-div":     { /* ... */ },
      "element-image":   { /* ... */ },
      "element-accordion":{ /* ... */ },
      "element-alert":   { /* ... */ },
      "element-carousel":{ /* ... */ },
      "element-code":    { /* ... */ },
      "element-counter": { /* ... */ },
      "element-divider": { /* ... */ },
      "element-icon-box":{ /* ... */ },
      "element-image-gallery":{ /* ... */ },
      "element-list":    { /* ... */ },
      "element-nav-menu":{ /* ... */ },
      "element-pricing-tables":{ /* ... */ },
      "element-progress-bar":{ /* ... */ },
      "element-related-posts":{ /* ... */ },
      "element-search":  { /* ... */ },
      "element-sidebar": { /* ... */ },
      "element-slider":  { /* ... */ },
      "element-social-icons":{ /* ... */ },
      "element-svg":     { /* ... */ },
      "element-tabs":    { /* ... */ },
      "element-team-members":{ /* ... */ },
      "element-testimonials":{ /* ... */ },
      "element-text-basic":{ /* ... */ },
      "element-text":    { /* ... */ },
      "element-video":   { /* ... */ },
      "element-wordpress":{ /* ... */ },
      "element-post-content":{ /* ... */ },
      "element-post-meta":{ /* ... */ },
      "element-post-navigation":{ /* ... */ },
      "element-post-taxonomy":{ /* ... */ },
      "element-post-title":{ /* ... */ },

      // Custom CSS injected globally
      "customCSS": ":root { --max-width: 1280px; }"
    },
    "conditions": [ /* see Conditions reference */ ]
  }
]
```

### Conditions for theme-style targeting

Each theme style has a `conditions` array (same shape as element conditions but with theme-style-specific keys). Available keys (from `theme-styles\controls\conditions.php`):

- `postType` -- post type slug or array
- `template` -- template post ID
- `templateType` -- `header|footer|content|archive|search|error|popup|section|password_protection|wc_*`
- `archive` -- `home|search|author|date|post_type:foo|taxonomy:cat`
- `single` -- `post_type:foo` or `post_id:42`
- Custom conditions added via `bricks/conditions/options` filter

Order matters: Bricks evaluates theme styles top-down and the **first match wins**. Set fallback styles (with broad conditions) at the end.

### Per-element theme-style controls

Look up exactly which keys an element supports under theme style by reading `{template_dir}/includes/theme-styles/controls/element-{name}.php`. E.g. `element-button.php` (26 KB!) defines every button typography/background/border/state setting.

The convention: each control file returns a controls array. Keys typically map directly to the same `_typography`, `_background`, etc. you'd set on a single element -- the theme style sets a default that any element of that type inherits.

### Helper to read theme style at runtime (PHP)

```php
\Bricks\Theme_Styles::get_setting_by_key( 'heading' );
// returns the theme-style block applicable to <heading> on the current page, or null
```

## 2. Global classes (`bricks_global_classes` option)

Reusable named classes that hold a `settings` block and are referenced by ID on elements.

```jsonc
[
  {
    "id":       "clsAbc123",
    "name":     "Card",
    "category": "uvgrPpC",            // category id from bricks_global_classes_categories
    "modified": 1715200000,
    "user_id":  1,
    "settings": {
      "_padding":  { "top":"24px","right":"24px","bottom":"24px","left":"24px" },
      "_background":{ "color":{ "hex":"#ffffff" } },
      "_border":   { "radius":{ "top":"12px","right":"12px","bottom":"12px","left":"12px" } },
      "_boxShadow":{ "values":{ "offsetX":"0","offsetY":"4px","blur":"12px","spread":"0" }, "color":{ "hex":"rgba(0,0,0,0.06)" } }
    }
  }
]
```

**Reference from an element:** array of class IDs in `_cssGlobalClasses`:

```jsonc
"_cssGlobalClasses": [ "clsAbc123", "clsXyz9" ]
```

Bricks renders these as `.brxe-clsAbc123` (the **id is the class name** -- readable name lives in `name`). Output CSS is generated to a single global classes file.

**Categories** live in `bricks_global_classes_categories` option and are just `{ id, name }` pairs.

**Trash** is `bricks_global_classes_trash` -- soft-deleted classes go here; Bricks tracks counts to detect mass deletion bugs.

**User authorship/timestamps:** `bricks_global_classes_user`, `bricks_global_classes_timestamp` track who created/modified each class -- used by the multi-user collaboration UI.

## 3. Color palette (`bricks_color_palette` option)

Named colors usable across the builder. Each palette entry:

```jsonc
{
  "id":      "color-primary",
  "name":    "Primary",
  "raw":     "var(--brand-primary)",   // optional raw string (CSS var)
  "hex":     "#0ea5e9",
  "rgb":     "rgba(14,165,233,1)"
}
```

A palette is `{ id, name, colors: [ ... ] }`. Multiple palettes can coexist; the active palette feeds the color picker swatches in the builder.

When an element uses a palette color, it stores `{ "hex": "#..." }` plus optionally a reference id. Updates to the palette repaint references on save.

## 4. Global variables (`bricks_global_variables` option)

CSS custom properties registered as design tokens.

```jsonc
[
  { "id":"var_abc","name":"--space-1","value":"4px",  "category":"spacing" },
  { "id":"var_def","name":"--space-2","value":"8px",  "category":"spacing" },
  { "id":"var_xyz","name":"--brand",  "value":"#0ea5e9","category":"colors"  }
]
```

Categories: `bricks_global_variables_categories`.

**Use anywhere a value is accepted:**
```jsonc
"_padding":     { "top": "var(--space-3)", "bottom": "var(--space-3)" },
"_background":  { "color": { "raw": "var(--brand)" } },
"_typography":  { "color": { "raw": "var(--text-primary)" } }
```

Bricks emits the variables onto `:root` in the global stylesheet, so they're usable from `_cssCustom` and child-theme CSS too.

## 5. Components (`bricks_components` option)

Component **definitions** -- reusable element trees with locked structure and per-instance settings.

```jsonc
[
  {
    "id":     "cmp_card",
    "label":  "Card",
    "elements": [
      {
        "id":"root", "name":"container", "parent":"0",
        "children":["heading1","text1","button1"],
        "settings": { /* ... */ }
      },
      { "id":"heading1", "name":"heading", "parent":"root", "settings":{ "text":"{prop:title}" } },
      { "id":"text1",    "name":"text",    "parent":"root", "settings":{ "text":"{prop:body}" } },
      { "id":"button1",  "name":"button",  "parent":"root", "settings":{ "text":"{prop:cta}" } }
    ],
    "props": [
      { "id":"title","label":"Title","type":"text","default":"Title here" },
      { "id":"body", "label":"Body", "type":"editor","default":"..." },
      { "id":"cta",  "label":"CTA",  "type":"text","default":"Learn more" }
    ]
  }
]
```

A **component instance** in a page is a single element entry:
```jsonc
{ "id":"e1","cid":"cmp_card","instanceId":"3","name":"container","settings":{ "title":"Real title" } }
```

The `cid` resolves to the definition; `instanceId` makes the rendered class/uid unique. The `settings` here are **prop overrides** plus any allowed standard settings.

## 6. Page settings (`_bricks_page_settings` post meta)

Per-page overrides:

```jsonc
{
  "bodyClasses":             "is-dark",
  "headerDisabled":          false,
  "footerDisabled":          false,
  "popupDisabled":           false,
  "disableLazyLoad":         false,
  "siteLayout":              "boxed",
  "siteLayoutBoxedMaxWidth": "1280px",
  "siteBackground":          { "color": { "hex": "#ffffff" } },
  "scrollSnapType":          "y mandatory",
  "scrollSnapAlign":         "start",
  "headerSticky":            true,
  "headerStickyOnScroll":    true,
  "headerStickyShrink":      true
}
```

Also accepts any key from the theme-style "general" group, applied as a per-page override.

## 7. Global settings (`bricks_global_settings` option)

Site-wide:

```jsonc
{
  "customBreakpoints":  true,
  "cssLoading":         "file",          // "file" | "inline"
  "cacheCSS":           true,
  "disableSeo":         false,
  "disableOpenGraph":   false,
  "permissionsLicense": "...",
  "apiKeyGoogle":       "...",
  "apiKeyMapbox":       "...",
  "apiKeyMailchimp":    "...",
  "apiKeySendgrid":     "...",
  "apiKeyGoogleRecaptcha": "...",
  "apiKeyHCaptcha":     "...",
  "apiKeyTurnstile":    "...",
  "instagramAccessToken":"...",
  "passwordProtectionDisable": false,
  "queryFilters":       true,             // gates filter elements
  "componentDetectChanges": true
}
```

Most keys map to settings page fields in `{template_dir}/includes/admin/admin-screen-settings.php` (160 KB of UI generation).

## 8. Style Manager (`bricks_style_manager` option, since 2.2)

Newer system that wraps theme styles + global classes + variables into a coordinated state. If present, prefer it over editing theme styles + classes separately.

## 9. Element manager (`bricks_element_manager` option)

Lets admins disable native elements globally:

```jsonc
{
  "heading":  { "status": "active" },
  "carousel": { "status": "disabled" },
  "facebook-page": { "status": "disabled" }
}
```

Mandatory elements (cannot be disabled): `container`, `posts`, `post-comments` (`Elements::mandatory_elements()`).

## 10. Hooks for extending these

```php
add_filter( 'bricks/theme_styles/controls', $cb );           // add theme-style sections
add_filter( 'bricks/theme_styles/element_settings', $cb );   // tweak per-element defaults
add_filter( 'bricks/global_classes', $cb );                  // mutate registered classes
add_filter( 'bricks/global_variables', $cb );                // mutate variables
```
