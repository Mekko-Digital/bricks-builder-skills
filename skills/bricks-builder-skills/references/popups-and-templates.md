# Popups & Templates

A **template** in Bricks is a post of type `bricks_template` (constant `BRICKS_DB_TEMPLATE_SLUG`). The template's content is just a Bricks page -- built the same way. What makes a template special is its **type** and its **assignment conditions**.

## Template types (`_bricks_template_type`)

| Type | Purpose | Auto-applies to |
|---|---|---|
| `header` | Site header | every front-end page (matched by conditions) |
| `footer` | Site footer | every front-end page |
| `content` | Body content layout | a single post / archive page |
| `archive` | Custom archive layout | post type / taxonomy archives |
| `search` | Search results layout | search page |
| `error` | 404 page | 404 |
| `password_protection` | Password-protected post gate | password-protected pages |
| `popup` | Popup overlay | triggered manually via interaction |
| `section` | Reusable section (used inside pages) | inserted via Template element |
| `wc_archive` | Woo product archive | shop / category / tag |
| `wc_product` | Woo single product | product page |
| `wc_cart` / `wc_cart_empty` | Cart / empty cart | cart page |
| `wc_form_checkout` / `wc_form_pay` | Checkout / pay | checkout |
| `wc_thankyou` / `wc_order_receipt` | Order received / receipt | order pages |
| `wc_account_*` (8 sub-types) | My Account screens | each account section |

The full list is in `Database::$default_template_types`.

## Template settings (`_bricks_template_settings` post meta)

```jsonc
{
  "templateConditions": [
    { "main": "include",                              // "include" | "exclude"
      "type": "post_type",
      "value": "page" },
    { "relation": "OR" },
    { "main": "include",
      "type": "single",
      "value": "post_id:42" }
  ],

  "headerPosition":  "top",            // top|left|right (for header type)
  "headerWidth":     "100%",
  "headerSticky":    true,
  "headerStickyOnScroll": true,
  "headerStickyShrink":   true,
  "headerStickyShrinkAt": "tablet_portrait",

  "popupCloseOn":           ["backdrop","esc"],
  "popupBodyScroll":        false,
  "popupZindex":            10000,
  "popupShowAt":            "tablet_portrait",
  "popupShowOn":            ["mobile_portrait","mobile_landscape"],
  "popupBreakpointMode":    "at",                // "at" | "on"
  "popupDisableAutoFocus":  false,
  "popupScrollToTop":       true,
  "popupAjax":              false,
  "popupBackground":        { "color":{ "hex":"rgba(0,0,0,0.6)" } },
  "popupBackdropTransition":"opacity 0.3s ease",
  "popupContentWidth":      "560px",
  "popupContentHeight":     "auto",
  "popupContentPadding":    { "top":"32px","right":"32px","bottom":"32px","left":"32px" },

  "popupLimitWindow":         0,                 // show max N times per page load
  "popupLimitSessionStorage": 0,                 // per session
  "popupLimitLocalStorage":   0,                 // until localStorage cleared
  "popupLimitTimeStorage":    24,                // hours

  "popupIsInfoBox":           false,
  "popupInfoBoxWidth":        300,
  "popupIsWoo":               false              // Woo Quick View popup
}
```

## Template assignment conditions

`templateConditions[]` controls **where** the template is applied. Each item:
```jsonc
{ "main":"include"|"exclude", "type":<key>, "value":<...> }
```

Common types (verified in `{template_dir}/includes/templates.php`):
- `entire_website` -- applies everywhere (use for default header/footer)
- `frontpage` -- front page only
- `post_type` -- value: `post|page|product|<cpt>`
- `single` -- value: `post_id:42` or `post_type:foo`
- `archive` -- value: `post_type:foo|taxonomy:cat|home|search|author|date`
- `taxonomy` -- value: taxonomy slug (or specific term: `category:news`)
- `language` -- value: locale code (WPML/Polylang)

Mix include/exclude with `relation` rows for AND/OR logic. The first matching template wins per type -- order them in `Bricks > Templates`.

## Active-template resolution

`Database::set_active_templates()` runs on `wp` and picks one template per slot for the current request:

```php
self::$active_templates = [
  'header'              => 0,
  'footer'              => 0,
  'content'             => 0,
  'archive'             => 0,
  'error'               => 0,
  'search'              => 0,
  'section'             => 0,
  'password_protection' => 0,
  'popup'               => [],   // multiple popups can be active per page
];
```

Override per-page: a page's `_bricks_page_settings` can set `headerDisabled`/`footerDisabled`/`popupDisabled`.

## Popups -- how to trigger them

A popup is just a template. To show it from any element on a page, add an interaction:

```jsonc
"_interactions": [
  {
    "id": "open-newsletter",
    "trigger": "click",
    "action":  "click",                  // verify against your version
    "target":  "popup",
    "templateId": 1234
  }
]
```

In some Bricks versions the action literal is `"openPopup"`/`"closePopup"`; the rendered DOM uses `data-popup-id` and Bricks JS wires the open/close. When in doubt, configure the popup in the builder UI once and inspect the saved JSON to confirm.

To **auto-open on page load** (with frequency cap):
```jsonc
[
  {
    "trigger":"contentLoaded","delay":"3000",
    "action":"click","target":"popup","templateId":1234,
    "interactionConditions":[
      { "conditionType":"localStorage","storageKey":"newsletterSeen","storageCompare":"notExists" }
    ]
  }
]
```

To **close** on a button inside the popup:
```jsonc
{ "trigger":"click", "action":"hide", "target":"popup" }
```
And to remember the dismissal:
```jsonc
{ "trigger":"click","action":"storageAdd","storageType":"localStorage",
  "actionAttributeKey":"newsletterSeen","actionAttributeValue":"1" }
```

## Sections (reusable)

Type `section` templates are inserted into pages via the **Template element** (`name: "template"`):

```jsonc
{ "name":"template", "settings": { "templateId": 567 } }
```

Use sections for: site-wide CTA, footer block fragments, repeated hero patterns. Sections support dynamic data, so you can keep them context-aware.

## Headers / footers

A header/footer template is rendered around the main content. Multiple can exist; conditions decide which.

- `headerPosition` = `top` (default), `left`, or `right` -- when `left`/`right`, the body content gets pushed.
- `headerWidth` controls the side-header width.
- `headerSticky` and `headerStickyShrink` produce the sticky-shrinking header pattern.

The frontend renders the header from `_bricks_page_header_2` of the active header template (Database::get_data with `header` content area).

## Programmatic creation

```php
$id = wp_insert_post( [
  'post_type'   => BRICKS_DB_TEMPLATE_SLUG,
  'post_title'  => 'My Header',
  'post_status' => 'publish',
] );
update_post_meta( $id, BRICKS_DB_TEMPLATE_TYPE,     'header' );
update_post_meta( $id, BRICKS_DB_TEMPLATE_SETTINGS, [
  'templateConditions' => [
    [ 'main' => 'include', 'type' => 'entire_website' ],
  ],
] );
update_post_meta( $id, BRICKS_DB_PAGE_HEADER, [ /* element array */ ] );
```

You also need to invalidate Bricks's CSS cache or save the template once via the UI to regenerate per-template CSS.

## Template bundles & tags

Bricks ships a taxonomy `template_bundle` and `template_tag` to organize templates. These are just WP taxonomies -- `wp_set_post_terms($id, [...], 'template_bundle')`.

## Hooks

```php
add_filter( 'bricks/active_templates',   $cb, 10, 1 );  // override slot resolution
add_filter( 'bricks/template_types',     $cb );          // add custom types
add_filter( 'bricks/templates/conditions', $cb );        // add condition types
add_action( 'bricks/popup/before_render', $cb );
add_action( 'bricks/popup/after_render',  $cb );
```
