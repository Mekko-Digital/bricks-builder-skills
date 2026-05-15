# Quick Reference -- Naming, Class Names, Common JSON

Cheat sheet to look up fast. For depth, jump to the specialized references.

## Class & ID naming

| What | Selector / convention |
|---|---|
| Element root class (by name) | `.brxe-{element-name}` -- e.g. `.brxe-heading`, `.brxe-icon-box` |
| Element root class (by id) | `.brxe-{6charid}` -- e.g. `.brxe-abc123` |
| Element root id attr | `id="brxe-{6charid}"` -- only when CSS settings exist or `_cssId` set |
| Custom CSS id | `id="{settings._cssId}"` (no prefix) |
| Custom classes | from `settings._cssClasses` (space-sep, no dots) |
| Global class | `.brxe-{globalClassId}` -- id from `bricks_global_classes` |
| Component instance | uses `.brxe-{cid}` (component definition id), not the per-instance id |
| Inside `_cssCustom` | `%root%` resolves to `.brxe-{6charid}` at render |
| Element data attribute | `data-x-element="{name}"` (builder context) |

## Element name -> element class file

```
container -> includes/elements/container.php -> \Bricks\Element_Container
icon-box  -> includes/elements/icon-box.php  -> \Bricks\Element_Icon_Box
post-toc  -> includes/elements/post-toc.php  -> \Bricks\Element_Post_Toc
```

Built-in name -> class: replace `-` with `_`, then `ucwords(_)`, prefix `\Bricks\Element_`.

## Saved JSON cheat-sheet (one element)

```jsonc
{
  "id":      "abc123",
  "name":    "container",
  "parent":  "0",                            // or parent id string
  "children":[ "def456", "ghi789" ],
  "settings":{
    "tag":           "section",
    "_direction":    "row",
    "_alignItems":   "center",
    "_justifyContent":"space-between",
    "_columnGap":    "32px",

    "_padding":            { "top":"80px","right":"24px","bottom":"80px","left":"24px" },
    "_padding:tablet_portrait": { "top":"40px","right":"16px","bottom":"40px","left":"16px" },

    "_typography":   { "font-size":"18px","color":{ "hex":"#0f172a" } },
    "_background":   { "color":{ "hex":"#f8fafc" } },
    "_border":       { "radius":{ "top":"12px","right":"12px","bottom":"12px","left":"12px" } },

    "_cssClasses":       "u-container",
    "_cssGlobalClasses": [ "clsAbc123" ],
    "_cssCustom":        "%root% { will-change: transform; }",

    "_interactions": [
      { "trigger":"enterView","action":"startAnimation","target":"self",
        "animationType":"fadeInUp","animationDuration":"0.8s","runOnce":true }
    ],

    "_conditions": [
      { "key":"user_logged_in","compare":"!=","value":"1" }
    ],

    "_attributes": [
      { "name":"data-track","value":"hero" }
    ]
  }
}
```

## Default breakpoint keys

`desktop` (1279, base) * `tablet_portrait` (991) * `mobile_landscape` (767) * `mobile_portrait` (478)

Suffix any settings key: `_padding:mobile_portrait`, `_typography:tablet_portrait`.

## Element categories

`basic` * `layout` * `general` * `media` * `wordpress` * `single` * `query` * `filter` * `woocommerce` * `custom` (used by child theme).

## Control types (the `'type'` field)

`text * textarea * editor * code * number * select * checkbox * color * gradient * image * image-gallery * icon * link * audio * video * svg * typography * border * spacing * dimensions * box-shadow * text-shadow * background * text-align * align-items * justify-content * direction * transform * transition * repeater * query * filters * datepicker * separator * info`

## Common control_options keys (preset option lists)

`animationTypes`, `lightboxAnimationTypes`, `ajaxLoaderAnimations`, `borderStyle`, `fontWeight`, `fontStyle`, `iconPosition`, `objectFit`, `position`, `queryTypes`, `queryOrder`, `queryOrderBy`, `termsOrderBy`, `usersOrderBy`, `queryCompare`, `queryOperator`, `queryValueType`, `templateTypes`, `imageSizes`, `taxonomies`, `userRoles`, `backgroundPosition`, `backgroundRepeat`, `backgroundSize`, `backgroundAttachment`, `blendMode`, `buttonSizes`, `styles`, `whiteSpace`, `textWrap`.

## DB constants (cheat sheet)

```
BRICKS_DB_PAGE_HEADER         = _bricks_page_header_2
BRICKS_DB_PAGE_CONTENT        = _bricks_page_content_2
BRICKS_DB_PAGE_FOOTER         = _bricks_page_footer_2
BRICKS_DB_PAGE_SETTINGS       = _bricks_page_settings
BRICKS_DB_TEMPLATE_TYPE       = _bricks_template_type
BRICKS_DB_TEMPLATE_SETTINGS   = _bricks_template_settings
BRICKS_DB_EDITOR_MODE         = _bricks_editor_mode

BRICKS_DB_GLOBAL_SETTINGS     = bricks_global_settings
BRICKS_DB_BREAKPOINTS         = bricks_breakpoints
BRICKS_DB_THEME_STYLES        = bricks_theme_styles
BRICKS_DB_GLOBAL_CLASSES      = bricks_global_classes
BRICKS_DB_GLOBAL_VARIABLES    = bricks_global_variables
BRICKS_DB_COLOR_PALETTE       = bricks_color_palette
BRICKS_DB_COMPONENTS          = bricks_components
BRICKS_DB_TEMPLATE_SLUG       = bricks_template
```

## Bricks PHP helpers (use, don't reinvent)

```php
bricks_is_builder()        // editor canvas
bricks_is_builder_main()   // editor chrome (panel side)
bricks_is_builder_iframe() // editor canvas iframe
bricks_is_frontend()
bricks_is_ajax_call()
bricks_is_rest_call()

\Bricks\Helpers::generate_random_id( $with_dash = false )
\Bricks\Database::get_data( $post_id, $area = '' )
\Bricks\Database::get_setting( $key, $fallback = false )      // global setting
\Bricks\Theme_Styles::get_setting_by_key( $element_name )     // applied theme style for element
\Bricks\Breakpoints::get_breakpoints()
\Bricks\Breakpoints::$is_mobile_first
\Bricks\Elements::register_element( $file, $name = '', $class = '' )
\Bricks\Elements::get_element( $element, $property = '' )
```

## Frontend JS globals

- `window.bricksData` -- server-injected config (breakpoints, animationTypes, etc.).
- `window.bricksUtils` -- utility helpers, including `BricksFunction(name, fn)` to register an init that runs on page-load and after AJAX content updates.
- `bricksAnimations` -- Animate.css class names.

When custom JS depends on Bricks-rendered HTML being in the DOM, register via `bricksUtils.BricksFunction` so it re-runs after AJAX-loaded content (loop pagination, query filters, AJAX popups).

## "Did Bricks really save what I think?" -- verify

```bash
# WP-CLI
wp post meta get <id> _bricks_page_content_2 --format=json
```

Or read with PHP:
```php
$elements = get_post_meta( $id, '_bricks_page_content_2', true );
print_r( $elements );
```

If the result is a string (not an array), it's invalid -- Bricks expects a serialized array. Re-save in the builder to fix.

## When the panel and the rendered page disagree

1. Hard refresh (CSS files cache).
2. Bricks > Settings > General > Regenerate CSS files.
3. Confirm `cssLoading` setting (file vs inline) matches your environment.
4. Inspect rendered HTML -- does it have `.brxe-{id}` for the expected element? If not, the JSON is malformed.
5. Check `_conditions` -- element may be conditionally hidden.

## When in doubt

Read the element source: `{template_dir}/includes/elements/{name}.php` is the source of truth for what controls and settings keys exist. Don't guess.
