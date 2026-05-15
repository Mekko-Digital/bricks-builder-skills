# Query Loop, Pagination, Filters

Source: `{template_dir}/includes/query.php` (3127 lines).

A "query loop" element renders the same children once per result. Loop-capable elements are: `posts`, `related-posts`, `carousel` (when `type=posts`), and any container element that has a query attached (set on the container's `_query` setting -> makes it iterate).

## Object types

The loop's source. Set on the query setting's `objectType`:

| objectType | Iterates | Engine |
|---|---|---|
| `post` | Posts/Pages/CPT | `WP_Query` |
| `term` | Taxonomy terms | `WP_Term_Query` |
| `user` | Users | `WP_User_Query` |
| `woocommerce` | WooCommerce products | WC + WP_Query |
| `api` | Remote API responses | `wp_remote_get`, JSON parsing |
| `array` | Arbitrary PHP array | `bricks/query/array` filter |
| `acf-relationship` | ACF relationship field's posts | ACF |
| `metabox-relationship` | Meta Box relationship field | Meta Box |

## Post query -- full setting shape

Saved under `settings._query` (or `settings.query` for elements like `posts`):

```jsonc
"_query": {
  "objectType":      "post",
  "post_type":       "post",                 // string OR array
  "posts_per_page":  6,
  "paged":           1,                       // overridden by URL on archives
  "offset":          0,
  "orderby":         "date",                  // see options below
  "order":           "DESC",                  // ASC | DESC
  "meta_key":        "",                      // when orderby=meta_value
  "post__in":        [12, 34],
  "post__not_in":    [5],
  "exclude_current_post": true,
  "post_parent":     0,
  "ignore_sticky_posts": true,
  "tax_query":   [
    { "taxonomy":"category", "field":"slug", "terms":["news"], "operator":"IN" },
    { "relation":"AND" }
  ],
  "meta_query":  [
    { "key":"price", "value":"100", "compare":">=", "type":"NUMERIC" },
    { "relation":"AND" }
  ],
  "s":               "search query",
  "disable_query_merge": false,               // don't merge with main query on archive
  "infinite_scroll": true,                    // auto-load next page on scroll
  "ajax_loader":     true,
  "useQueryEditor":  false,
  "queryEditor":     "return [ 'post_type' => 'project' ];",   // returns WP_Query args
  "disable_update_post_meta_cache":  true,
  "disable_update_post_term_cache":  true,
  "no_results_text": "Nothing found"
}
```

### Orderby options (see `setup.php` `control_options.queryOrderBy`)

`date`, `modified`, `title`, `name`, `ID`, `author`, `comment_count`, `menu_order`, `parent`, `rand`, `meta_value`, `meta_value_num`, `post__in` (preserve `post__in` order), `relevance` (search), `none`.

### tax_query operators

`IN` (any), `NOT IN`, `AND` (all), `EXISTS`, `NOT EXISTS`. Mix with a `relation` entry for AND/OR among multiple sub-queries.

### meta_query compare

`=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `BETWEEN`, `NOT BETWEEN`, `EXISTS`, `NOT EXISTS`, `REGEXP`, `RLIKE`, `NOT REGEXP`. `type`: `CHAR | NUMERIC | DATETIME | DATE | TIME | BINARY | DECIMAL | SIGNED | UNSIGNED`.

## Term query

```jsonc
"_query": {
  "objectType": "term",
  "taxonomy":   "category",            // string or array
  "number":     20,
  "paged":      1,
  "offset":     0,
  "orderby":    "name",                // name|count|term_id|slug|description|none|include
  "order":      "ASC",
  "hide_empty": true,
  "meta_query": [],
  "term__in":   [],
  "term__not_in":[],
  "exclude":    "3,4",
  "search":     "",
  "current_post_term": false           // limit to current post's terms
}
```

## User query

```jsonc
"_query": {
  "objectType": "user",
  "number":     10,
  "paged":      1,
  "offset":     0,
  "orderby":    "user_login",          // user_login|display_name|user_registered|post_count|ID
  "order":      "DESC",
  "search":     "*foo*",
  "role":       "subscriber",          // string or array
  "role__in":   ["editor","author"],
  "role__not_in":[],
  "meta_query": [],
  "include":    [1,2],
  "exclude":    [3,4]
}
```

## WooCommerce products

`objectType: 'woocommerce'` -- supports the same `tax_query`/`meta_query`/etc. as posts plus WC-specific filters (in stock, on sale, featured) when query rendered via the Woo product loop element.

## API loop

```jsonc
"_query": {
  "objectType": "api",
  "apiUrl":     "https://api.example.com/items",
  "apiHeaders": "{\"Authorization\":\"Bearer ...\"}",
  "apiBody":    "",
  "apiMethod":  "GET",
  "apiPath":    "data.items",          // dot-path into JSON to find the array
  "apiCacheTime": 300                   // seconds
}
```

Inside the loop, dynamic data: `{query_api:title}`, `{query_api @key:'title|rendered'}`.

## Loop iteration & dynamic data

Inside any descendant element of the looped container, dynamic-data tags resolve against the **loop object** (the current post/term/user/array item):
- `{post_title}`, `{post_url}`, `{post_excerpt:50}`, `{featured_image}`, `{author_name}` -- when looping posts.
- `{term_name}`, `{term_url}`, `{term_count}` -- when looping terms.
- `{user_name}`, `{user_email}` -- when looping users.
- `{loop_index}`, `{loop_index_real}` -- current iteration index (0-based).

## Pagination element (`pagination`)

```jsonc
"settings": {
  "queryId":   "abc123",            // ID of the query element to paginate (or 'main')
  "style":     "default",
  "pageNumbers": true,
  "showPrev":  true, "prevText": "Prev",
  "showNext":  true, "nextText": "Next",
  "showFirst": false, "showLast": false,
  "midSize":   2,
  "endSize":   1
}
```

When `queryId` is `'main'` it paginates the WP main query (e.g. on archive pages).

## Infinite scroll vs Load-More

- **Infinite scroll:** `_query.infinite_scroll: true` on the loop. Bricks adds a sentinel and IntersectionObserver auto-loads the next page.
- **Load-more button:** any element + interaction:
  ```jsonc
  { "trigger":"click", "action":"loadMore", "loadMoreQuery": "main" }
  ```
  When all pages loaded, the button hides itself.

## Query filter elements

Filter elements (`filter-checkbox`, `filter-radio`, `filter-select`, `filter-search`, `filter-range`, `filter-datepicker`, `filter-submit`, `filter-active-filters`) bind to a target query loop element. Common fields:

```jsonc
"settings": {
  "queryId":   "abc123",                // target query element id
  "filterKey": "category",              // meta_key | tax:slug | post_field
  "filterSource": "taxonomy",           // taxonomy | postMeta | postField | userMeta | ...
  "filterTaxonomy": "category",
  "filterMetaKey": "price",
  "filterCompare": "IN",                // IN | NOT IN | AND | =, etc.
  "filterAutoSubmit": true,
  "filterUpdateUrl":  true,             // sync to URL params
  "filterPlaceholder":"All",
  "options": [ { "value":"news","label":"News" } ],
  "multiSelect": true,
  "searchable":  false,
  "minChars":    2
}
```

`filter-submit` triggers the submit when not auto-submitting; `filter-active-filters` shows current selections with clear-buttons. Filters require `bricks_global_settings.queryFilters: true` (Bricks > Settings > Query Filters).

## Index / cache

The query-filters indexer (`{template_dir}/includes/query-filters-indexer.php` and `query-filters.php`) maintains an index table for fast filter UIs. After major content changes, rebuild via Bricks > Settings > Query Filters > Reindex.

## Filters/hooks

```php
// Modify final query args
add_filter( 'bricks/posts/query_vars',  $cb, 10, 3 ); // ($query_vars, $settings, $element_id)
add_filter( 'bricks/terms/query_vars',  $cb, 10, 3 );
add_filter( 'bricks/users/query_vars',  $cb, 10, 3 );

// PHP query loop result (objectType=array)
add_filter( 'bricks/query/array',       $cb, 10, 2 ); // ($items, $settings)

// Loop object override (advanced)
add_filter( 'bricks/query/loop_object', $cb, 10, 3 );

// Empty state
add_filter( 'bricks/query/no_results_content', $cb, 10, 2 );
```
