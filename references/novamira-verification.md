# Novamira -- Live WordPress Verification Recipes

[Novamira](https://github.com/use-novamira/novamira) is an MCP server giving AI agents full WordPress access via PHP execution and filesystem ops. When configured, it's the **primary** verification path for this skill -- it queries the **live install** rather than static disk files, so it's authoritative even when:

- Bricks has been updated since this skill was generated.
- The user runs Bricks on a different machine or hosting.
- Disk files diverge from runtime state (caching, plugin overrides, etc.).

## Discovering tool names

Novamira's exact MCP tool names depend on the server build. Discover them once per session:

```
ToolSearch query:"novamira"
```

The 8 capabilities (verbatim from novamira.ai/docs):
1. **Execute PHP** -- full WP env (`$wpdb`, all WP functions, plugins loaded)
2. **Read File**
3. **Write File** (PHP files restricted to a sandbox dir)
4. **Edit File**
5. **Delete File**
6. **Disable File** / **Enable File** (toggle `.bak` extension for safe rollback)
7. **List Directory**

Tool names you'll likely see: `mcp__novamira__execute_php`, `mcp__novamira__read_file`, `mcp__novamira__write_file`, `mcp__novamira__edit_file`, `mcp__novamira__list_directory`, etc. Always confirm via `ToolSearch` before invoking.

## Safety / blast-radius rules

- **Execute PHP** runs in the live WordPress process. Read-only PHP (return values, get_option, get_post_meta, $wpdb->get_results SELECT) is safe to run unattended.
- **DB writes** (`update_option`, `update_post_meta`, `$wpdb->insert/update/delete`) -- confirm with the user before invoking, even in auto-mode. They can break the site irrecoverably if the payload shape is wrong.
- **Write File** to PHP files -- only into the child theme (`get_stylesheet_directory()`), never the parent (`get_template_directory()`). Write to PHP outside the sandbox is restricted by Novamira itself.
- **Disable File** is the safe-rollback move when you suspect a recent file edit broke things. Re-enable to restore.
- **Delete File** -- confirm with user.

## Path conventions in the running WP

```php
get_template_directory()         // parent theme:  .../wp-content/themes/bricks
get_template_directory_uri()
get_stylesheet_directory()       // active child:  .../wp-content/themes/bricks-child
get_stylesheet_directory_uri()
ABSPATH                          // WP root
WP_CONTENT_DIR                   // .../wp-content
WPMU_PLUGIN_DIR                  // mu-plugins
plugin_dir_path( __FILE__ )      // current plugin
```

Use these in PHP recipes -- never hard-code `f:\bricks\...` style paths in code that runs through Novamira; the WP install may live anywhere.

---

## Recipe library

### A. Verify a native element's control keys

```php
// Read full element source (best for first-time exploration)
return file_get_contents( get_template_directory() . '/includes/elements/heading.php' );
```

```php
// Just the control keys + types -- for quick lookup
$src = file_get_contents( get_template_directory() . '/includes/elements/heading.php' );
preg_match_all( "/\\\$this->controls\\['(\\w+)'\\][^\\[]*\\[\\s*(?:[^]]*?'type'\\s*=>\\s*'([\\w-]+)')?/", $src, $m );
return array_combine( $m[1], $m[2] );
```

```php
// Discover all native element names by scanning the directory
return array_map(
  fn( $f ) => basename( $f, '.php' ),
  glob( get_template_directory() . '/includes/elements/*.php' )
);
```

### B. Read the JSON of a specific page/template

```php
// Page body content
return get_post_meta( 42, '_bricks_page_content_2', true );
```

```php
// Header template body
$header_id = 17; // template post id
return get_post_meta( $header_id, '_bricks_page_header_2', true );
```

```php
// All Bricks meta of a single post in one call
$id   = 42;
$keys = [
  '_bricks_page_content_2','_bricks_page_header_2','_bricks_page_footer_2',
  '_bricks_page_settings','_bricks_template_type','_bricks_template_settings',
  '_bricks_editor_mode',
];
$out = [];
foreach ( $keys as $k ) {
  $v = get_post_meta( $id, $k, true );
  if ( $v !== '' ) $out[ $k ] = $v;
}
return $out;
```

```php
// Find ALL posts/templates that have Bricks content
global $wpdb;
return $wpdb->get_results( "
  SELECT p.ID, p.post_title, p.post_type, p.post_status
    FROM {$wpdb->posts} p
    JOIN {$wpdb->postmeta} m ON m.post_id = p.ID
   WHERE m.meta_key = '_bricks_page_content_2'
     AND m.meta_value <> ''
     AND p.post_status NOT IN ('trash','auto-draft')
   ORDER BY p.post_modified DESC
   LIMIT 50
", ARRAY_A );
```

```php
// Find every Bricks template by type
global $wpdb;
return $wpdb->get_results( $wpdb->prepare( "
  SELECT p.ID, p.post_title, m.meta_value AS template_type
    FROM {$wpdb->posts} p
    JOIN {$wpdb->postmeta} m ON m.post_id = p.ID
   WHERE p.post_type = %s
     AND m.meta_key  = %s
   ORDER BY m.meta_value, p.post_title
", 'bricks_template', '_bricks_template_type' ), ARRAY_A );
```

### C. Inspect global state (options)

```php
// Active breakpoints (with custom ones if any)
return \Bricks\Breakpoints::get_breakpoints();
```

```php
// Or raw DB row
return get_option( 'bricks_breakpoints' );
```

```php
// All theme styles
return get_option( 'bricks_theme_styles', [] );
```

```php
// Global classes -- return name+id for the picker
return array_map(
  fn( $c ) => [ 'id' => $c['id'], 'name' => $c['name'] ],
  get_option( 'bricks_global_classes', [] )
);
```

```php
// Global CSS variables
return get_option( 'bricks_global_variables', [] );
```

```php
// Color palettes
return get_option( 'bricks_color_palette', [] );
```

```php
// Components catalog (definitions)
return array_map(
  fn( $c ) => [ 'id' => $c['id'], 'label' => $c['label'] ?? '' ],
  get_option( 'bricks_components', [] )
);
```

```php
// Bricks site settings (API keys, feature toggles)
$gs = get_option( 'bricks_global_settings', [] );
// strip secrets before returning
foreach ( [ 'apiKeyGoogle','apiKeyMapbox','apiKeyMailchimp','apiKeySendgrid',
           'apiKeyGoogleRecaptcha','apiKeyHCaptcha','apiKeyTurnstile',
           'instagramAccessToken' ] as $k ) {
  if ( isset( $gs[ $k ] ) ) $gs[ $k ] = '***REDACTED***';
}
return $gs;
```

```php
// Element manager -- which native elements are disabled
return get_option( 'bricks_element_manager', [] );
```

### D. Inspect dynamic data tags

```php
// Every registered DD tag (what the user can use in {...} tokens)
do_action( 'init' );
return apply_filters( 'bricks/dynamic_tags_list', [] );
```

```php
// Render a DD string against a specific post (preview what it'll resolve to)
return \Bricks\Integrations\Dynamic_Data\Providers::render_content(
  '{post_title} by {author_name} ({post_date:F j, Y})',
  get_post( 42 )
);
```

### E. Verify which template wins on a given URL

```php
// Simulate a request to URL X and report active templates
$url = 'https://example.com/about/';
$post_id = url_to_postid( $url );
if ( ! $post_id ) return [ 'error' => 'No post for URL' ];

// Force WP to "be on" that post
query_posts( [ 'p' => $post_id, 'post_type' => 'any' ] );
the_post();

\Bricks\Database::set_active_templates();
return \Bricks\Database::$active_templates;
```

### F. Listing files / discovering structure

```php
// Custom elements registered in the active child theme
return array_map( 'basename', glob( get_stylesheet_directory() . '/elements/*.php' ) );
```

```php
// Confirm Bricks version installed
return defined( 'BRICKS_VERSION' ) ? BRICKS_VERSION : 'BRICKS_VERSION not defined';
```

```php
// Theme + child paths
return [
  'parent'        => get_template_directory(),
  'parent_uri'    => get_template_directory_uri(),
  'child'         => get_stylesheet_directory(),
  'child_uri'     => get_stylesheet_directory_uri(),
  'wp_root'       => ABSPATH,
  'content'       => WP_CONTENT_DIR,
];
```

### G. Writing files (custom elements, child theme assets)

Use Novamira's **Write File** for new files, **Edit File** for surgical changes to existing ones. Targets to write into:

- `{stylesheet_dir}/elements/{name}.php` -- new custom element
- `{stylesheet_dir}/functions.php` -- register the element (Edit File: insert into the existing `init` array)
- `{stylesheet_dir}/css/{slug}.css` -- per-page CSS
- `{stylesheet_dir}/js/{slug}.js` -- per-page JS
- `{stylesheet_dir}/inc/{module}.php` -- PHP module

After adding a new custom-element file, you also need to:
1. Edit `functions.php` to add the path to `$element_files` (around the existing `init` priority-11 closure).
2. Optionally bump theme version in `style.css` to bust caches.

### H. Modifying Bricks DB safely (write operations -- confirm with user!)

```php
// Update a global setting
$gs = get_option( 'bricks_global_settings', [] );
$gs['customBreakpoints'] = true;
return update_option( 'bricks_global_settings', $gs ); // true on success
```

```php
// Add a custom breakpoint
$bps = get_option( 'bricks_breakpoints', [] );
if ( empty( $bps ) ) $bps = \Bricks\Breakpoints::get_default_breakpoints();
$bps[] = [
  'key'    => 'large_desktop',
  'label'  => 'Large desktop',
  'width'  => 1440,
  'icon'   => 'laptop',
  'custom' => true,
];
usort( $bps, fn( $a, $b ) => $b['width'] <=> $a['width'] );
return update_option( 'bricks_breakpoints', $bps );
```

```php
// Trigger CSS file regeneration (after breakpoint changes)
$_POST['action'] = 'bricks_regenerate_bricks_css_files';
\Bricks\Breakpoints::regenerate_bricks_css_files();
return 'regenerated';
```

```php
// Programmatic page authoring (DANGEROUS -- ALWAYS confirm + backup first)
$elements = [
  [
    'id'       => substr( md5( uniqid() ), 0, 6 ),
    'name'     => 'section',
    'parent'   => '0',
    'children' => [],
    'settings' => [
      'tag'      => 'section',
      '_padding' => [ 'top'=>'80px','right'=>'24px','bottom'=>'80px','left'=>'24px' ],
    ],
  ],
];
$ok = update_post_meta( 42, '_bricks_page_content_2', $elements );
\Bricks\Assets::generate_css_file_from_post_id( 42 );
return $ok;
```

### I. Backup before any DB write

```php
// Snapshot a post's Bricks meta to a JSON sidecar in uploads
$id      = 42;
$snap    = [];
foreach ( [ '_bricks_page_content_2','_bricks_page_header_2','_bricks_page_footer_2',
            '_bricks_page_settings','_bricks_template_settings' ] as $k ) {
  $v = get_post_meta( $id, $k, true );
  if ( $v !== '' ) $snap[ $k ] = $v;
}
$dir  = trailingslashit( wp_upload_dir()['basedir'] ) . 'bricks-backups/';
if ( ! is_dir( $dir ) ) wp_mkdir_p( $dir );
$file = $dir . "post-{$id}-" . date( 'Ymd-His' ) . '.json';
file_put_contents( $file, wp_json_encode( $snap, JSON_PRETTY_PRINT ) );
return $file;
```

### J. Sanity checks before going live

```php
// Are CSS files current with installed Bricks version?
return [
  'BRICKS_VERSION'                 => BRICKS_VERSION,
  'last_css_regenerated_for'       => get_option( 'bricks_breakpoints_last_generated' ),
  'css_loading'                    => \Bricks\Database::get_setting( 'cssLoading', 'file' ),
  'cache_css'                      => \Bricks\Database::get_setting( 'cacheCSS', true ),
  'query_filters_enabled'          => \Bricks\Helpers::enabled_query_filters(),
  'custom_breakpoints_enabled'     => \Bricks\Database::get_setting( 'customBreakpoints', false ),
];
```

```php
// Find pages that reference a (now non-existent) global class id
$id   = 'clsAbc123';
$rows = $GLOBALS['wpdb']->get_results( $GLOBALS['wpdb']->prepare( "
  SELECT post_id FROM {$GLOBALS['wpdb']->postmeta}
   WHERE meta_key IN ('_bricks_page_content_2','_bricks_page_header_2','_bricks_page_footer_2')
     AND meta_value LIKE %s
", '%' . $GLOBALS['wpdb']->esc_like( $id ) . '%' ), ARRAY_A );
return $rows;
```

---

## When to use Novamira vs local Read

| Task | Novamira | Local Read/Grep |
|---|---|---|
| Look up a control key in a Bricks element | ? Read File `{template_dir}/includes/elements/{name}.php` | ? if you have `{template_dir}` on disk |
| See what's actually saved in the DB | ? Execute PHP `get_post_meta(...)` | ? disk files don't reflect DB |
| List installed templates / popups / components | ? Execute PHP | ? |
| Resolve which header/footer applies to a URL | ? Execute PHP `set_active_templates()` | ? requires WP runtime |
| Read a child-theme element file you're editing | ? Read File | ? |
| Write a new custom element file | ? Write File | ? Write |
| See what dynamic-data tags ACF/MetaBox/etc. add | ? Execute PHP `apply_filters('bricks/dynamic_tags_list', [])` | ? requires plugins loaded |
| Verify Bricks version | ? Execute PHP `BRICKS_VERSION` | ?? disk version may differ from active install |
| Write directly to wp_options | ? Execute PHP `update_option(...)` (confirm first!) | ? |

**Default**: lean on Novamira when present; fall back to local FS otherwise.
