---
name: bricks-builder-skills
description: Build, edit, and audit pages, templates, popups, and custom elements for the Bricks Builder WordPress theme (installed at {template_dir}). Use when the task involves Bricks elements, JSON page data (`_bricks_page_content_2`, `_bricks_page_header_2`, `_bricks_page_footer_2`), Bricks templates (`bricks_template` post type), child-theme custom elements at {stylesheet_dir}/elements/, theme styles, global classes/variables, breakpoints, forms, query loops, conditions, dynamic-data tags, interactions/animations, or popups. Triggers: "Bricks", "brxe-", "page builder element", "container/section/heading element", "_bricks_", "bricks_template", "register_element", or any code touching the JSON shapes documented here.
---

# Bricks Builder -- Authoritative Skill

You are working on the **Bricks Builder** WordPress theme (paid license). The parent theme is at `{template_dir}` and the active child is at `{stylesheet_dir}`. Custom code goes in the child only -- never edit the parent.

> **Path placeholders.** This skill uses `{template_dir}` and `{stylesheet_dir}` as portable placeholders for the live WordPress paths returned by PHP's `get_template_directory()` and `get_stylesheet_directory()`. They are NOT literal strings to type into shell commands. To resolve them at runtime, use Novamira **Execute PHP**: `return get_template_directory();` -- or read your `wp-config.php` to compute them locally. The skill works on any host (Windows, macOS, Linux, hosted WP) regardless of where the install actually lives on disk.

The whole point of this skill is **no hallucination**. Every element name, control key, animation name, breakpoint key, condition key, dynamic-data tag, and DB option below is verified against the source. When in doubt, **verify against the live WordPress install** -- don't guess.

## 0. Verification -- prefer Novamira MCP over local files

If a Novamira MCP server is configured (it gives full WordPress access via PHP execution + filesystem ops), use it as the **primary** verification path. It targets the **live** WP install -- which is the actual source of truth, and which may differ from static disk paths if the user runs Bricks elsewhere or has updated since this skill was written.

Discover Novamira's exact tool schemas with `ToolSearch query:"novamira"` once per session. The 8 capabilities exposed (verbatim from novamira.ai/docs):

1. **Execute PHP** -- runs PHP with the full WP env (`$wpdb`, `get_option`, `get_post_meta`, plugins loaded). This is your most powerful tool -- use it for anything DB-related.
2. **Read File** -- file contents from the server FS.
3. **Write File** -- create/overwrite (PHP files restricted to a sandbox dir).
4. **Edit File** -- targeted string replacement.
5. **Delete File** -- remove files/dirs.
6. **Disable File** / **Enable File** -- temporarily neutralize sandbox files (toggles `.bak` extension, used for safe rollback during debugging).
7. **List Directory** -- filtered listing.

**Decision rule:**
- Need to read the theme source to verify a control key? -> Novamira **Read File** at `get_template_directory()/includes/elements/heading.php` (or local `Read` if you have parent at `{template_dir}`).
- Need to know what's actually saved for a page/template? -> Novamira **Execute PHP**: `return get_post_meta(42, '_bricks_page_content_2', true);`.
- Need to know what global classes / breakpoints / theme styles exist? -> Novamira **Execute PHP**: `return get_option('bricks_global_classes');`.
- Writing a new custom element? -> Novamira **Write File** at `get_stylesheet_directory()/elements/{name}.php` (the active child).
- See `references/novamira-verification.md` for ready-made PHP one-liners.

If Novamira is **not** configured, fall back to local `Read`/`Grep` against the on-disk paths below -- they're functionally equivalent for read-only work.

## 1. The mental model in 60 seconds

Bricks stores a **page = ordered array of element objects** as one giant serialized PHP array in post meta. There is no per-element row. Each element is a flat object referenced by `parent` and `children` (parent/child IDs). Settings are a flat key->value map.

```json
// One element in the array
{
  "id": "abc123",                 // 6-char alphanumeric ID, string
  "name": "container",            // element name from includes/elements/
  "parent": "0",                  // "0" = top-level, else parent element id
  "children": ["def456","ghi789"],// array of child element ids (only nestables)
  "settings": {
    "tag": "section",
    "_padding": { "top": "60px", "right": "20px", "bottom": "60px", "left": "20px" },
    "_padding:tablet_portrait": { "top": "30px", "right": "16px", "bottom": "30px", "left": "16px" },
    "_typography": { "font-size": "18px", "color": { "hex": "#111" } },
    "_cssGlobalClasses": ["clsAbc12"]
  },
  "label": "Hero",                // optional editor label
  "themeStyles": { "tag": "h1" }, // optional override
  "cid": "cmp-xyz",               // optional: this is a component INSTANCE
  "instanceId": "1"               // optional: with cid, gives unique uid
}
```

The whole page is one such array stored at one of these meta keys (constants in `{template_dir}/functions.php`):

| Constant | Meta key | Holds |
|---|---|---|
| `BRICKS_DB_PAGE_HEADER` | `_bricks_page_header_2` | Header template elements |
| `BRICKS_DB_PAGE_CONTENT` | `_bricks_page_content_2` | Page/template body elements |
| `BRICKS_DB_PAGE_FOOTER` | `_bricks_page_footer_2` | Footer template elements |
| `BRICKS_DB_PAGE_SETTINGS` | `_bricks_page_settings` | Per-page settings (body class, layout, etc.) |
| `BRICKS_DB_TEMPLATE_TYPE` | `_bricks_template_type` | header/footer/content/archive/search/error/popup/section/wc_* |
| `BRICKS_DB_TEMPLATE_SETTINGS` | `_bricks_template_settings` | Template-only settings (popup config, conditions) |

Templates are a custom post type: `bricks_template` (constant `BRICKS_DB_TEMPLATE_SLUG`).

Site-wide options (all `wp_options`):
`bricks_global_settings`, `bricks_global_classes`, `bricks_global_classes_categories`, `bricks_color_palette`, `bricks_global_variables`, `bricks_global_variables_categories`, `bricks_breakpoints`, `bricks_theme_styles`, `bricks_components`, `bricks_global_elements`, `bricks_global_queries`, `bricks_pseudo_classes`, `bricks_element_manager`, `bricks_icon_sets`, `bricks_custom_icons`, `bricks_fonts`, `bricks_font_faces`.

## 2. Class & ID naming convention (memorize this)

Every rendered element gets these classes on its root tag:

- `.brxe-{name}` -- the element name class, e.g. `.brxe-container`, `.brxe-heading`, `.brxe-icon-box`.
- `.brxe-{id}` -- the element's 6-char ID class, e.g. `.brxe-abc123`. This is the targetable selector.
- `id="brxe-{id}"` -- only added when the element has CSS settings or a custom `_cssId` is set.
- `.{custom-class}` -- anything in `settings._cssClasses` (space-separated, no dots).
- Component instances use the **component CID** for the class, not the per-instance id.

When writing custom CSS targeting Bricks elements, prefer `.brxe-{id}` (stable per element) or a custom class added via `_cssClasses`. Never write CSS against `[id^="brxe-"]` blanket selectors.

## 3. The two file layouts that matter

**Theme source (read-only reference):**
```
{template_dir}/
├── includes/elements/           # 73 native elements, one PHP file each
│   ├── base.php                 # abstract Element -- all common controls live here
│   ├── container.php section.php block.php div.php
│   └── heading.php text.php image.php button.php icon.php ...
├── includes/setup.php           # control_options registry (animationTypes, queryOrderBy, etc.)
├── includes/breakpoints.php
├── includes/interactions.php
├── includes/query.php
├── includes/conditions.php
├── includes/database.php        # get_data() / save by post id
├── includes/theme-styles/       # element-specific theme-style controls
├── includes/integrations/dynamic-data/providers/   # all DD tags
├── assets/css/frontend.min.css  # generated, do not edit
└── functions.php                # BRICKS_DB_* constants, BRICKS_VERSION, BRICKS_PATH
```

**Where YOU write code:**
```
{stylesheet_dir}/
├── functions.php          # registers custom elements, enqueues per-page CSS/JS
├── style.css              # global child styles
├── elements/*.php         # custom Bricks elements (each extends \Bricks\Element)
├── inc/                   # PHP modules
├── css/*.css              # per-page stylesheets (loaded if is_page('slug'))
└── js/*.js                # per-page or shared JS modules
```

The existing custom elements (`title.php`, `primary-nav.php`, `faq-accordion.php`, `logo-marquee.php`, `stats-counter.php`, `icon.php`, `roi-calculator.php`) are working examples -- read them before adding new ones.

## 4. When you're about to do X, read Y

Treat the references in `references/` as the source of truth. They are too long to load eagerly; **Read them on demand** when the task touches that area.

| Task | Read first |
|---|---|
| Spec/build any element JSON, look up element name/category/key controls | `references/elements-catalog.md` |
| Apply spacing/typography/border/transform/parallax to ANY element | `references/element-base-controls.md` |
| Build a layout -- section/container/block/div, flex/grid, gap, masonry | `references/containers-and-layout.md` |
| Add hover/click/scroll behavior, popups, load-more, custom JS hooks | `references/interactions-and-animations.md` |
| Set responsive values, add a custom breakpoint, mobile-first | `references/responsive-breakpoints.md` |
| Edit theme styles, global classes, color palette, CSS variables | `references/theme-styles-and-globals.md` |
| Build a contact/login/registration/webhook form | `references/forms.md` |
| Configure a posts/products/users/terms loop, filters, pagination | `references/query-loop.md` |
| Show/hide based on user/post/date/viewport | `references/conditions.md` |
| Insert post/user/ACF/MetaBox/JetEngine/Pods values into text | `references/dynamic-data.md` |
| Build/wire a popup or template (header/footer/archive/single) | `references/popups-and-templates.md` |
| Register a new custom element via the child theme | `references/custom-elements.md` |
| Inspect or write Bricks data directly to the DB | `references/db-schema.md` |
| Use Novamira to verify control keys, read DB state, write files into the WP install | `references/novamira-verification.md` |
| Style cheat sheet -- class names, JSON value shapes, common keys | `references/quick-reference.md` |

Pattern: **before generating any settings JSON, open the relevant reference and copy keys/value shapes verbatim.** Do not invent control keys -- every control key for a native element is verifiable inside `set_controls()` of the element file. Verify via:
- **Novamira (preferred):** Read File `{TEMPLATEPATH}/includes/elements/{name}.php`, or Execute PHP `return file_get_contents( get_template_directory() . "/includes/elements/{$name}.php" );`
- **Local fallback:** `Read` `{template_dir}/includes/elements/{name}.php`

## 5. The four absolute rules

1. **Never edit `{template_dir}/`** -- it is the licensed theme. Updates will overwrite. All custom work goes in `{stylesheet_dir}`.
2. **Custom elements must extend `\Bricks\Element`** and be registered via `\Bricks\Elements::register_element( $file )` on the `init` hook (priority 11 -- see `bricks-child/functions.php:104`). The class name must follow the pattern Bricks expects, and the file must define one class.
3. **Never assume a control key -- verify it.** A heading's text key is `text`, an icon-box's content key is `content`, a button's text key is `text`, but countdown uses `date`, alert uses `content` (editor type). Read the element file or `references/elements-catalog.md`.
4. **Responsive overrides use `:breakpoint_key` suffix on the SAME flat settings object** -- e.g. `_padding`, `_padding:tablet_portrait`, `_padding:mobile_portrait`. They are **not** nested. The breakpoint keys are the literal `key` values from `bricks_breakpoints` option (defaults: `desktop`, `tablet_portrait`, `mobile_landscape`, `mobile_portrait`).

## 6. Workflow when the user asks you to build a page or section

1. **Identify the element type(s) needed.** Open `references/elements-catalog.md` and pick by name + category. Prefer `section > container > content elements` for top-level structure (it's the convention reflected in `set_active_templates`).
2. **Look up control keys** for each element you'll use -- open `{template_dir}/includes/elements/{name}.php` and read its `set_controls()`. For every control key with a `'css'` array, the user is likely styling at runtime; for plain keys, set the value once.
3. **Apply layout/spacing/typography** via the base-class underscore keys (`_padding`, `_margin`, `_typography`, `_background`, `_border`, `_boxShadow`, `_cssClasses`, `_cssGlobalClasses`). See `references/element-base-controls.md`.
4. **Wire interactions** if there are any (popups, scroll, hover effects, load-more) -- see `references/interactions-and-animations.md`. Don't reinvent animation classes; Bricks ships with the full Animate.css set, listed there.
5. **Add responsive overrides** with `:breakpoint_key` suffixed keys.
6. **Test in the browser** -- reload the page in WP, open it in the Bricks builder, and confirm the panel renders the controls you wrote. The builder reads back from DB exactly what you wrote.

## 7. Pitfalls observed in this codebase

- **`_bricks_page_content_2`** has the `_2` suffix because v1.x migrated the meta key. Don't omit the `_2`.
- **CSS classes input as `_cssClasses`** must NOT include the dot. `_cssGlobalClasses` references global-class **IDs**, not names -- the IDs come from `bricks_global_classes` option entries.
- **`_columnCount`** on container is for the legacy column system; modern layouts use `_direction` (row/column) + `_alignItems` + `_justifyContent` + `_gap`. Use grid via `_display: grid` plus `_gridTemplateColumns`.
- **`bricks_template` posts** have `_bricks_template_type` meta -- the type drives where the template auto-applies (header/footer/archive/single/etc.). Template assignment conditions live in `_bricks_template_settings.templateConditions`.
- **Heading element**'s `tag` control accepts `'custom'` and then reads `customTag` -- if you only set `tag: custom`, you will render a literal `<custom>` element. Always pair with `customTag`.
- **Form fields' `id`** is what the email `{{field-id}}` placeholder references and what JSON keys submission data by. It is NOT auto-generated from the label -- set it explicitly.
- **Don't enqueue child-theme JS in the builder panel chrome** -- `bricks_is_builder_main()` early return is required (see `bricks-child/functions.php:6`).
- **GSAP is not bundled** by Bricks -- the parallax/scroll system in core is custom JS. If you want GSAP, you enqueue it yourself in the child theme. The `history-bricks.js` and `roi-calculator-bricks.js` files in this child are the existing pattern.
- **The `animation` (entrance animation) controls on element base are deprecated since 1.6** -- do entrance animations via interactions (`startAnimation` action) instead.

## 8. Hook reference (for child-theme PHP)

```php
// Add a custom element class to the registered set (alternative to register_element())
add_filter( 'bricks/builder/elements', function( $names ) { $names[] = 'my-element'; return $names; } );

// Modify controls for a built-in element
add_filter( 'bricks/elements/heading/controls', function( $c ) { /* ... */ return $c; } );

// Add to a control group
add_filter( 'bricks/elements/heading/control_groups', function( $g ) { /* ... */ return $g; } );

// Register a new dynamic-data tag (provider)
add_filter( 'bricks/dynamic_tags_list', function( $tags ) { /* ... */ return $tags; } );
add_filter( 'bricks/dynamic_data/render_tag', function( $value, $tag, $post, $context ) { /* ... */ return $value; }, 10, 4 );

// Register a new condition key
add_filter( 'bricks/conditions/options', function( $opts ) { /* ... */ return $opts; } );

// Add a new theme-style section / control
add_filter( 'bricks/theme_styles/controls', function( $c ) { /* ... */ return $c; } );
```

All hook signatures are verified in `{template_dir}/includes/` -- grep before relying on parameter order.

## 9. Don't forget

- The child's `style.css` and `bricks-frontend` style depend on each other; child enqueue should declare `bricks-frontend` as parent.
- `bricks_is_builder()`, `bricks_is_builder_main()`, `bricks_is_builder_iframe()`, `bricks_is_frontend()`, `bricks_is_ajax_call()` -- these globals are how you guard logic; use them.
- Regenerating CSS files: `Bricks > Settings > General > Regenerate CSS files` button (or the WP-CLI/AJAX `bricks_regenerate_bricks_css_files`). Required after changing custom breakpoints.

For deeper detail, open the right reference in `references/`. Default to **read first, write second**.
