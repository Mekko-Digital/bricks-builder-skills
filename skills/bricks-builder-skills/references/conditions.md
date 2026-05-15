# Conditions (Show/Hide Logic)

Source: `{template_dir}/includes/conditions.php` (1190 lines).

Conditions can be attached to:
- Any element via `settings._conditions`
- Form fields via the field's `_conditions`
- Theme styles (when do they apply?)
- Templates (`_bricks_template_settings.templateConditions`)
- Interactions (per-interaction `interactionConditions`)

## Shape

Array alternating between condition objects and `relation` separators.

```jsonc
"_conditions": [
  { "key":"user_logged_in", "compare":"==", "value":"1" },
  { "relation":"AND" },
  { "key":"user_role",      "compare":"==", "value":["administrator","editor"] }
]
```

Multiple groups are achieved by alternating relations:

```jsonc
[
  { "key":"post_type", "compare":"==", "value":"page" },
  { "relation":"OR" },
  [
    { "key":"post_type",  "compare":"==", "value":"post" },
    { "relation":"AND" },
    { "key":"post_status","compare":"==", "value":"publish" }
  ]
]
```

(Nested arrays are AND-grouped within an outer relation.)

## Available condition keys

Grouped by the panel sections in `{template_dir}/includes/conditions.php`. Compare operators differ per key -- read the source if unsure.

### Post
| Key | Operators | Value |
|---|---|---|
| `post_id` | `==`, `!=`, `>=`, `<=`, `>`, `<` | numeric ID |
| `post_title` | `==`, `!=`, `contains`, `contains_not` | string |
| `post_parent` | `==`, `!=`, `>=`, `<=`, `>`, `<` | numeric |
| `post_status` | `==`, `!=` | `publish`, `draft`, `pending`, `private`, ... |
| `post_author` | `==`, `!=` | user ID |
| `post_date` | `==`, `!=`, `>=`, `<=`, `>`, `<` | date (datepicker) |
| `featured_image` | `==`, `!=` | `1` (set) / `0` (unset) |

### User
| Key | Operators | Value |
|---|---|---|
| `user_logged_in` | `==`, `!=` | `1` / `0` |
| `user_id` | `==`, `!=`, `>=`, `<=`, `>`, `<` | numeric |
| `user_registered` | `<`, `>` | date |
| `user_role` | `==`, `!=` | role name(s); array supported |
| `user_capability` | `==`, `!=` | capability slug |
| `user_meta` | many | `metaKey` + `value` (custom panel) |

### Date / time
| Key | Operators | Value format |
|---|---|---|
| `weekday` | `==`, `!=`, `>=`, `<=`, `>`, `<` | 1=Monday ... 7=Sunday |
| `date` | all comparators | `Y-m-d` |
| `time` | all | `H:i` |
| `datetime` | all | `Y-m-d H:i` |
| `recurring_dates` | between/contains | range pairs |

### Browser / device
| Key | Operators | Value |
|---|---|---|
| `browser` | `==`, `!=` | `chrome`, `firefox`, `safari`, `edge`, `opera`, `ie` |
| `os` | `==`, `!=` | `windows`, `mac`, `linux`, `android`, `ios` |
| `device` | `==`, `!=` | `desktop`, `tablet`, `mobile` |

### Page / context
| Key | Operators | Value |
|---|---|---|
| `is_front_page`, `is_home`, `is_singular`, `is_archive`, `is_search`, `is_404` | `==` | `1` / `0` |
| `language` | `==`, `!=` | locale code (with WPML/Polylang) |
| `query_result_count` | numeric comparators | numeric |

### Other
| Key | Notes |
|---|---|
| `dynamic_data` | Compare arbitrary DD value against a literal -- `dynamicData` field holds the tag |
| `custom_code` | PHP boolean expression (returns true/false) |
| `referrer` | URL contains/equals |
| `cookie` | cookie name + value comparison |
| `query_parameter` | `?utm_*` etc. |

## Compare operators

`==` equal, `!=` not equal, `>`, `<`, `>=`, `<=`, `contains`, `contains_not`, `in`, `not_in`, `between`, `not_between`, `regex`, `regex_not`, `empty`, `not_empty`, `set`, `not_set`.

Some keys only support a subset -- the panel filters automatically.

## Relations

`AND` -- all conditions in the run must match.
`OR` -- any condition in the run can match.

When you want a complex expression like `(A AND B) OR C`, use nested arrays:
```jsonc
[
  [ { /* A */ }, { "relation":"AND" }, { /* B */ } ],
  { "relation":"OR" },
  { /* C */ }
]
```

## Theme-style conditions

Theme styles use the same condition format under their `conditions` key, but with extra keys:
- `templateType` -- header|footer|content|archive|search|error|popup|section|wc_*
- `template`     -- specific template ID
- `archive`      -- `home|search|author|date|post_type:foo|taxonomy:cat`
- `single`       -- `post_type:foo|post_id:42`

## Template assignment conditions

Templates (the `bricks_template` post type) use `_bricks_template_settings.templateConditions[]` with the same keys + special template-targeting keys (where to render the header/footer/archive/etc.).

## Hooks

```php
add_filter( 'bricks/conditions/options',     $cb );  // add condition keys
add_filter( 'bricks/conditions/result',      $cb, 10, 3 );  // ($result, $condition, $element)
add_filter( 'bricks/conditions/show_element',$cb, 10, 3 );  // override final show/hide
```

To register a custom condition key, return additional entries from `bricks/conditions/options` and handle evaluation by hooking `bricks/conditions/result` for your new `key`.

## Recipes

**Show only to logged-out users:**
```jsonc
[ { "key":"user_logged_in", "compare":"!=", "value":"1" } ]
```

**Hide on archive of a specific CPT:**
```jsonc
[ { "key":"is_archive","compare":"==","value":"1" },
  { "relation":"AND" },
  { "key":"post_type", "compare":"==","value":"product" } ]
```

**Show only on weekdays during business hours:**
```jsonc
[
  { "key":"weekday","compare":">=","value":"1" },
  { "relation":"AND" },
  { "key":"weekday","compare":"<=","value":"5" },
  { "relation":"AND" },
  { "key":"time","compare":">=","value":"09:00" },
  { "relation":"AND" },
  { "key":"time","compare":"<","value":"18:00" }
]
```

**Show if a query produced results (works inside loop wrapper):**
```jsonc
[ { "key":"query_result_count","compare":">","value":"0" } ]
```
