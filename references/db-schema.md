# Database Schema (Bricks Storage Layout)

All Bricks data lives in standard WordPress tables. There are **no custom tables** for elements/templates (only one for form submissions). Everything is `wp_options` and `wp_postmeta`.

## Constants (defined in `{template_dir}/functions.php`)

These constants name the option/meta keys. Always use the constant in PHP code; the literal string only here for reference.

### Per-post meta keys

| Constant | Meta key | Holds |
|---|---|---|
| `BRICKS_DB_PAGE_HEADER` | `_bricks_page_header_2` | Header elements array (template only) |
| `BRICKS_DB_PAGE_CONTENT` | `_bricks_page_content_2` | Body elements array (page/template) |
| `BRICKS_DB_PAGE_FOOTER` | `_bricks_page_footer_2` | Footer elements array (template only) |
| `BRICKS_DB_PAGE_SETTINGS` | `_bricks_page_settings` | Per-page settings object |
| `BRICKS_DB_TEMPLATE_TYPE` | `_bricks_template_type` | Template type string (header/footer/content/popup/section/archive/wc_*/...) |
| `BRICKS_DB_TEMPLATE_SETTINGS` | `_bricks_template_settings` | Template-only settings (popup config, conditions) |
| `BRICKS_DB_EDITOR_MODE` | `_bricks_editor_mode` | `bricks` or `wordpress` (decides which editor renders this post) |

### Site-wide options (`wp_options`)

| Constant | Option name | Holds |
|---|---|---|
| `BRICKS_DB_GLOBAL_SETTINGS` | `bricks_global_settings` | Site settings (API keys, feature toggles) |
| `BRICKS_DB_BREAKPOINTS` | `bricks_breakpoints` | Breakpoint definitions array |
| `BRICKS_DB_THEME_STYLES` | `bricks_theme_styles` | Theme styles array |
| `BRICKS_DB_GLOBAL_CLASSES` | `bricks_global_classes` | Global classes array |
| `BRICKS_DB_GLOBAL_CLASSES_CATEGORIES` | `bricks_global_classes_categories` | Class categories |
| `BRICKS_DB_GLOBAL_CLASSES_LOCKED` | `bricks_global_classes_locked` | Class lock state |
| `BRICKS_DB_GLOBAL_CLASSES_TIMESTAMP` | `bricks_global_classes_timestamp` | Per-class modified timestamps |
| `BRICKS_DB_GLOBAL_CLASSES_TRASH` | `bricks_global_classes_trash` | Soft-deleted classes |
| `BRICKS_DB_GLOBAL_CLASSES_USER` | `bricks_global_classes_user` | Class authorship |
| `BRICKS_DB_PSEUDO_CLASSES` | `bricks_global_pseudo_classes` | Custom pseudo-class rules |
| `BRICKS_DB_GLOBAL_VARIABLES` | `bricks_global_variables` | CSS variables |
| `BRICKS_DB_GLOBAL_VARIABLES_CATEGORIES` | `bricks_global_variables_categories` | Variable categories |
| `BRICKS_DB_COLOR_PALETTE` | `bricks_color_palette` | Color palettes |
| `BRICKS_DB_COMPONENTS` | `bricks_components` | Component definitions |
| `BRICKS_DB_GLOBAL_ELEMENTS` | `bricks_global_elements` | Globally synced elements |
| `BRICKS_DB_GLOBAL_QUERIES` | `bricks_global_queries` | Saved query definitions |
| `BRICKS_DB_GLOBAL_QUERIES_CATEGORIES` | `bricks_global_queries_categories` | Query categories |
| `BRICKS_DB_STYLE_MANAGER` | `bricks_style_manager` | (2.2+) consolidated style state |
| `BRICKS_DB_ELEMENT_MANAGER` | `bricks_element_manager` | Per-element disable state |
| `BRICKS_DB_ICON_SETS` | `bricks_icon_sets` | Registered icon sets (FA, Themify, custom) |
| `BRICKS_DB_CUSTOM_ICONS` | `bricks_custom_icons` | User-uploaded SVG icons |
| `BRICKS_DB_DISABLED_ICON_SETS` | `bricks_disabled_icon_sets` | Disabled built-in icon sets |
| `BRICKS_DB_FONT_FAVORITES` | `bricks_font_favorites` | Favorited Google fonts |
| `BRICKS_DB_CUSTOM_FONTS` | `bricks_fonts` | Uploaded fonts |
| `BRICKS_DB_CUSTOM_FONT_FACES` | `bricks_font_faces` | `@font-face` rules |
| `BRICKS_DB_CUSTOM_FONT_FACE_RULES` | `bricks_font_face_rules` | Generated CSS for custom faces |
| `BRICKS_DB_ADOBE_FONTS` | `bricks_adobe_fonts` | Adobe Typekit kit data |
| `BRICKS_DB_PINNED_ELEMENTS` | `bricks_pinned_elements` | Per-user element favorites |
| `BRICKS_DB_SIDEBARS` | `bricks_sidebars` | Custom sidebars |
| `BRICKS_DB_REMOTE_TEMPLATES` | `bricks_remote_templates` | Cached remote-template metadata |
| `BRICKS_DB_CAPABILITIES_PERMISSIONS` | `bricks_capabilities_permissions` | Per-role builder permissions |
| `BRICKS_DB_PANEL_WIDTH`, `BRICKS_DB_STRUCTURE_WIDTH`, `BRICKS_DB_BUILDER_SCALE_OFF`, `BRICKS_DB_BUILDER_WIDTH_LOCKED` | `bricks_*` | Builder UI prefs |
| `BRICKS_DB_TEMPLATE_SLUG` | `bricks_template` | Template post type slug |
| `BRICKS_DB_TEMPLATE_TAX_BUNDLE` | `template_bundle` | Template bundle taxonomy |
| `BRICKS_DB_TEMPLATE_TAX_TAG` | `template_tag` | Template tag taxonomy |

## The element-array shape (definitive)

```jsonc
[
  {
    "id":         "abc123",                  // 6-char string
    "name":       "section",                 // element name
    "parent":     "0",                        // string id of parent or "0" for root
    "children":   ["def456","ghi789"],
    "settings":   { /* control values */ },

    // Optional
    "label":      "Hero",
    "themeStyles":{ "tag":"h1" },
    "cid":        "cmp_xyz",                  // component instance: definition id
    "instanceId": "1",                        // component instance ordinal
    "global":     "abc123"                    // if globally synced (legacy)
  },
  // ... more elements at all depths in flat array ...
]
```

Notes:
- The array is **flat** -- children references resolve through `parent`/`children` IDs.
- Order doesn't strictly matter -- Bricks rebuilds the tree at render -- but the builder writes them in tree order.
- IDs are stable: deleting and re-adding gives a new ID. Don't depend on a specific value.
- Values are PHP-serialized into the meta cell (WP-standard `update_post_meta`).

## Reading at runtime

```php
$page_id  = get_the_ID();
$elements = \Bricks\Database::get_data( $page_id );             // current area auto
$elements = \Bricks\Database::get_data( $page_id, 'header' );    // explicit area: header|footer|content
```

`get_data` returns `[]` if no Bricks data is stored -- i.e. the post wasn't built with Bricks.

**Via Novamira** (no need to be inside WP runtime):
```php
return \Bricks\Database::get_data( 42, 'content' );
```

## Writing programmatically

```php
$elements = [
  [
    'id'       => bin2hex( random_bytes(3) ),       // any 6-char string
    'name'     => 'section',
    'parent'   => '0',
    'children' => [],
    'settings' => [
      'tag'      => 'section',
      '_padding' => [ 'top'=>'80px','right'=>'0','bottom'=>'80px','left'=>'0' ],
    ],
  ],
];
update_post_meta( $page_id, BRICKS_DB_PAGE_CONTENT, $elements );
```

After programmatic writes, regenerate the page-CSS file:
```php
\Bricks\Assets::generate_css_file_from_post_id( $page_id );
```

If `cssLoading == 'inline'`, no file is needed -- Bricks will inline on next render.

## The save AJAX flow

When a user saves in the builder, `wp_ajax_bricks_save_post` (see `{template_dir}/includes/ajax.php`) writes the same three meta keys (header/content/footer) plus `_bricks_page_settings`. It also updates global classes/queries/components for the whole site if the user touched them. Saving a post can therefore mutate site-wide options -- guard yourself when scripting.

## Form submissions table

If `save-submission` action is used, Bricks creates a custom table on activation:
- Table: `{wp_prefix}bricks_form_submissions`
- Columns: `id`, `form_id`, `form_name`, `post_id`, `user_id`, `email`, `status`, `data`, `referrer`, `ip`, `useragent`, `created_at`.
- `data` is JSON of all submitted fields.
- Admin viewer at WP Admin -> Bricks -> Form Submissions.

## Query-filter index table

When query filters are enabled and indexed:
- Table: `{wp_prefix}bricks_query_filters_index`
- Used by filter elements to build options lists fast without running the full query each time.
- Rebuild via Bricks > Settings > Query Filters > Reindex (or AJAX `bricks_query_filters_reindex`).

## Cache invalidation

Bricks caches:
- Per-post element CSS files in `{template_dir}/assets/css\` (when `cssLoading=file`).
- Page-CSS in transients when `cssLoading=inline`.
- The active-templates resolution at request scope.

Force regenerate (admin):
- Bricks > Settings > General -> "Regenerate CSS files".
- Or AJAX action `bricks_regenerate_bricks_css_files`.

## How to safely inspect a page's Bricks JSON in DB

**Preferred -- Novamira Execute PHP** (live install, automatic unserialize):
```php
return get_post_meta( 42, '_bricks_page_content_2', true );
```

**WP-CLI (if you have shell access):**
```bash
wp post meta get 42 _bricks_page_content_2 --format=json
```

**Raw SQL (last resort):**
```sql
SELECT meta_value FROM wp_postmeta
 WHERE post_id = 42 AND meta_key = '_bricks_page_content_2';
```
Result is PHP-serialized -- pipe through `unserialize()`.

Modify with caution -- invalid serialized data will crash the page. Snapshot first (see `references/novamira-verification.md` ? Backup before any DB write).

## Editor-mode meta

`_bricks_editor_mode` decides which editor renders a post:
- `bricks` (default if user opens in Bricks) -- Bricks reads `_bricks_page_content_2`.
- `wordpress` -- Bricks ignores the post and WP renders `post_content`.

When converting a Gutenberg page to Bricks, Bricks parses blocks via `convert_block_to_element_settings` on each element class (see `Element_Heading::convert_block_to_element_settings` in heading.php). When converting back, `convert_element_settings_to_block` produces `wp:block` strings.
