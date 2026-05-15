# Dynamic Data Tags

Source: `{template_dir}/includes/integrations/dynamic-data\` -- multiple provider files.

Dynamic data tags are **inline tokens** that pull values into any text-accepting setting (text, textarea, link URL, image source, etc.). At render Bricks parses tokens and substitutes the resolved value.

## Token syntax

```
{tag_name}
{tag_name:filter}
{tag_name:filter:arg}
{tag_name @modifier:'value'}
{tag_name @modifier1:'value' @modifier2:'value2'}
```

The `:filter` form is short syntax for built-in transforms; the `@modifier:'value'` form is the canonical extensible syntax. Both work.

## Native namespaces

### Post (`{post_*}`)
- `post_title`, `post_id`, `post_url`, `post_slug`, `post_type`, `post_status`
- `post_date`, `post_modified`, `post_time`
- `post_content`, `post_excerpt`
- `post_comments_count`, `post_comments_url`
- `featured_image` -- image; modifiers: `:link`, `:url`, `:meta_size:thumbnail`/`:medium`/`:large`/`:full`, `:alt`, `:caption`
- `read_more` -- anchor to single
- `post_terms_{taxonomy}` -- e.g. `post_terms_category` (CSV of term names)
- `post_terms_{taxonomy}:link` -- linked
- `post_meta_{key}` -- custom field

### Author (`{author_*}`)
- `author_id`, `author_name`, `author_first_name`, `author_last_name`, `author_bio`
- `author_email`, `author_website`, `author_archive_url`
- `author_avatar` -- modifiers same as featured image
- `author_meta_{key}`

### Archive (`{archive_*}`)
- `archive_title`, `archive_description`, `archive_url`

### Term (`{term_*}` -- only resolves on taxonomy archives or in term loops)
- `term_id`, `term_name`, `term_description`, `term_url`, `term_count`, `term_image`
- `term_meta_{key}`

### User (`{user_*}` -- current logged-in user)
- `user_id`, `user_name`, `user_login`, `user_first_name`, `user_last_name`
- `user_email`, `user_url`, `user_avatar`
- `user_registered_date`, `user_role`
- `user_meta_{key}`

### Site (`{site_*}`)
- `site_title`, `site_tagline`, `site_url`, `site_logo`, `site_search_url`
- `current_year`, `current_date`, `current_time`

### URL / request
- `url_parameter_{key}` -- query param
- `cookie_{name}`
- `referrer_url`, `current_url`

### Loop (inside a query loop iteration)
- `loop_index`, `loop_index_real`, `loop_count`
- The post/term/user `*_*` tags resolve against the current loop object

### WordPress menus
- `menu_{location}` -- render a registered menu

### WooCommerce (`{woo_*}` -- when WC active)
- `woo_product_title`, `woo_product_price`, `woo_product_sale_price`, `woo_product_regular_price`, `woo_product_image`, `woo_product_rating`, `woo_product_stock`, `woo_product_description`, `woo_product_short_description`, `woo_product_sku`, `woo_product_categories`, `woo_product_tags`
- `woo_cart_count`, `woo_cart_total`, `woo_cart_url`
- `woo_my_account_url`, `woo_checkout_url`

### Theme / template
- `theme_style_*` -- current applied theme-style values
- `template_url` -- template-file URL

### Echo / shortcode
- `{echo:my_php_function:arg1:arg2}` -- calls a PHP function. Function must be **whitelisted** via `bricks/code/echo_function_names` filter (security).
- `{do_action:my_hook:arg}` -- fires `do_action()` and returns ob output.
- `{shortcode:[my_shortcode foo='bar']}` -- wrap with `[]`.

### ACF (`{acf_*}` -- when ACF active)
- `{acf_field_name}` -- value of ACF field on current post
- Modifiers: `:link`, `:value`, `:meta_size:large` for images, `:format` for dates

### Meta Box (`{mb_*}` -- when MB active)
- `{mb_field_id}` -- value of MB field

### JetEngine (`{je_*}` -- when JE active)
- `{je_field_name}`

### Pods (`{pods_*}` -- when Pods active)
- `{pods_field}`

### Toolset
- `{toolset_field}`

## Modifiers (the `@modifier:'value'` system)

Listed in `{template_dir}/includes/integrations/dynamic-data/dynamic-data-parser.php`:

| Modifier | Purpose | Example |
|---|---|---|
| `@fallback` | Default if empty | `{post_title @fallback:'Untitled'}` |
| `@fallback-image` | Default image (id or URL) | `{featured_image @fallback-image:123}` |
| `@sanitize` | true/false | `{post_meta_html @sanitize:'true'}` |
| `@exclude` | Skip listed values | `{active_filters_count @exclude:'q1w2e3'}` |
| `@start-at` | Start index in an iteration | `@start-at:1` |
| `@pad` | Pad index with zeros | `@pad:'2'` (1 -> 01) |
| `@key` | Pluck nested object key | `{query_api @key:'title|rendered'}` |
| `@is-array` | Force JSON output | `@is-array:'true'` |
| `@date` | Parse date | `{user_meta_birthday @date:'2024-01-01'}` |
| `@from` | Source date format | `@from:'Y-m-d'` |
| `@to` | Output date format | `@to:'d M Y'` |
| `@count` | Length / count | `{post_terms_category @count:'true'}` |
| `@truncate` | String length cap | (or use `:N` short form) |

## Built-in short filters (after a colon)

For quick formatting:

| Filter | Effect | Example |
|---|---|---|
| `{tag:N}` | Truncate to N chars/words | `{post_excerpt:50}` |
| `{tag:format}` | Date format string | `{post_date:F j, Y}` (PHP date) |
| `{tag:link}` | Wrap in `<a>` | `{featured_image:link}` |
| `{tag:url}` | URL only (no markup) | `{featured_image:url}` |
| `{tag:meta_size:thumbnail}` | Image size | `{featured_image:meta_size:large}` |
| `{tag:alt}` | Alt text | `{featured_image:alt}` |
| `{tag:caption}` | Caption | `{featured_image:caption}` |
| `{tag:value}` | Raw value (skip provider formatting) | `{acf_field:value}` |
| `{tag:number_format:2}` | PHP number_format | `{woo_product_price:number_format:2}` |
| `{tag:upper}` / `{tag:lower}` / `{tag:ucfirst}` | Case | |

## Conditional formatting via `_conditions`

Tags don't have inline if/else. For "show only if X has value" wrap the element in a condition:
```jsonc
"_conditions": [{ "key":"dynamic_data","dynamicData":"{post_meta_video_url}","compare":"not_empty" }]
```

## Provider hooks

```php
// Register a custom tag
add_filter( 'bricks/dynamic_tags_list', function( $tags ) {
  $tags[] = [
    'name'  => '{my_thing}',
    'label' => 'My Thing',
    'group' => 'My Plugin',
  ];
  return $tags;
} );

// Resolve it
add_filter( 'bricks/dynamic_data/render_tag', function( $value, $tag, $post, $context ) {
  if ( $tag !== '{my_thing}' ) return $value;
  return get_option( 'my_thing_value', '' );
}, 10, 4 );

// Resolve inside larger strings (text containing the tag)
add_filter( 'bricks/dynamic_data/render_content', function( $content, $post, $context ) {
  // mutate content if needed
  return $content;
}, 10, 3 );
```

## Security note on `{echo:...}`

Bricks blocks `echo:` by default. To allow specific PHP functions:

```php
add_filter( 'bricks/code/echo_function_names', fn( $list ) => array_merge( $list, [ 'my_safe_fn' ] ) );
```

`{echo:eval:...}` and similar are not whitelisted. Don't whitelist functions that take arbitrary code as input.

## Recipes

**Hero subtitle from ACF with fallback:**
```
{acf_hero_subtitle @fallback:'Welcome'}
```

**Author byline + linked archive:**
```
By <a href="{author_archive_url}">{author_name}</a>
```

**Reading time tag:**
```
{post_reading_time} min read
```

**Featured image as link to post:**
```html
<a href="{post_url}"><img src="{featured_image:meta_size:large:url}" alt="{post_title}"></a>
```

**Cart count badge that hides at zero:**
- Element shows `{woo_cart_count}`
- Element `_conditions`: `[{ "key":"dynamic_data","dynamicData":"{woo_cart_count}","compare":">","value":"0" }]`
