# Responsive Breakpoints

Source: `{template_dir}/includes/breakpoints.php`. Stored as option `bricks_breakpoints`. Custom breakpoints are gated behind `bricks_global_settings.customBreakpoints = true`.

## Default breakpoints (desktop-first, max-width)

| Key | Label | Width | Icon |
|---|---|---|---|
| `desktop` | Desktop | 1279 | laptop |
| `tablet_portrait` | Tablet portrait | 991 | tablet-portrait |
| `mobile_landscape` | Mobile landscape | 767 | phone-landscape |
| `mobile_portrait` | Mobile portrait | 478 | phone-portrait |

The `desktop` breakpoint is the **base**: settings without a suffix apply at the base. Each smaller breakpoint generates a `@media (max-width: {width}px)` rule that **overrides** base values when the viewport is at or below that width.

## How responsive values are stored

A single setting key `_padding` may have **multiple variants** in the same flat `settings` object, one per breakpoint, suffixed with `:breakpoint_key`:

```jsonc
"_padding":                 { "top":"100px","right":"0","bottom":"100px","left":"0" },
"_padding:tablet_portrait": { "top":"60px","right":"20px","bottom":"60px","left":"20px" },
"_padding:mobile_landscape":{ "top":"40px","right":"16px","bottom":"40px","left":"16px" },
"_padding:mobile_portrait": { "top":"24px","right":"12px","bottom":"24px","left":"12px" }
```

Bricks generates one CSS rule per variant inside the matching media query.

This works for **any** setting on any element -- typography, background, dimensions, transforms, flex direction, even content-side keys like the heading's `tag` (you can switch H1->H2 by viewport, though that has SEO implications and is rarely a good idea).

## Mobile-first

Set a **smaller breakpoint as the base** (`base: true` flag) and Bricks generates `@media (min-width: {width}px)` rules instead. The convention: define a `mobile_portrait` (or your custom small bp) as the base, then `mobile_landscape`, `tablet_portrait`, `desktop` each layer on top.

Switch via the breakpoint manager UI or by editing the option:

```php
update_option( 'bricks_breakpoints', [
  [ 'base'=>true, 'key'=>'mobile_portrait', 'label'=>'Mobile portrait', 'width'=>478 ],
  [             'key'=>'mobile_landscape','label'=>'Mobile landscape','width'=>767 ],
  [             'key'=>'tablet_portrait', 'label'=>'Tablet portrait', 'width'=>991 ],
  [             'key'=>'desktop',         'label'=>'Desktop',         'width'=>1279 ],
] );
```

The `Breakpoints::$is_mobile_first` flag flips and the entire CSS-generation pipeline switches.

## Custom breakpoints

Required to add new breakpoints (e.g., 1440 for "large desktop", 1920 for "ultrawide"):

1. **Enable custom breakpoints:** Bricks > Settings > Performance > "Custom breakpoints" -> ON. This sets `customBreakpoints = true` in `bricks_global_settings`.
2. **Add via the Breakpoint Manager** in the builder UI, OR via AJAX `bricks_update_breakpoints` (used by the manager).

Programmatically:

```php
$bps = get_option( 'bricks_breakpoints', [] );
$bps[] = [
  'key'   => 'large_desktop',
  'label' => 'Large desktop',
  'width' => 1440,
  'icon'  => 'laptop',
  'custom'=> true,
];
// Sort descending by width for desktop-first
usort( $bps, fn($a,$b) => $b['width'] <=> $a['width'] );
update_option( 'bricks_breakpoints', $bps );
```

3. **Regenerate CSS files** afterwards -- Bricks > Settings > General > Regenerate CSS files. Or trigger AJAX `bricks_regenerate_bricks_css_files`.

The key is normalized to lowercase, with `-` and ` ` replaced by `_`. After saving, your new key becomes a valid suffix in settings: `_padding:large_desktop`, `_typography:large_desktop`, etc.

## What `key` is allowed

```php
$key = strtolower( $key );
$key = str_replace( ['-', ' '], '_', $key );
```
Letters, digits, underscores. Avoid leading digits. Avoid existing keys (`desktop`, `tablet_portrait`, `mobile_landscape`, `mobile_portrait`).

## Pause / reset / delete

The breakpoint manager exposes:
- **Pause** (`paused: true`) -- hides the breakpoint from the canvas without deleting its data.
- **Reset** -- restores a default breakpoint to its shipped width.
- **Delete** -- only on `custom: true` breakpoints.

When deleting a custom breakpoint, settings keyed under that breakpoint suffix become orphaned (still in DB, ignored at render). Cleaning them up is manual.

## Builder canvas widths

Each breakpoint can also specify `widthBuilder` -- the canvas width Bricks switches to when you click that breakpoint icon in the builder toolbar. Optional; defaults to `width`.

## CSS generation

When `cssLoading == 'file'` (default), Bricks regenerates per-element CSS files into `{template_dir}/assets/css/elements\*.min.css` and the integrations in `{template_dir}/assets/css/integrations\*.min.css` whenever you save a page or update a breakpoint. Also regenerates `frontend.min.css` and `frontend-light.min.css`.

When `cssLoading == 'inline'`, no separate files -- the page-specific CSS is inlined via `<style>`.

## Order of precedence

Within base + breakpoint overrides:
1. Base value (no suffix).
2. Breakpoint override at the active media query.
3. Pseudo-class override (`:hover`, etc.) at the matching state.
4. Element `_cssCustom` (highest specificity if you write specific selectors).

Pseudo-class overrides cascade on top of breakpoint values within the same `@media` block when you build them via the panel. This is generated CSS -- if it doesn't behave, inspect with DevTools and grep the page-CSS file.

## Targeting in custom CSS

When writing CSS in a child theme stylesheet, use the actual breakpoint widths:

```css
/* Default (desktop) -- base */
.brxe-abc123 { padding: 100px 0; }

/* Tablet portrait -- 991 (max-width) */
@media (max-width: 991px) {
  .brxe-abc123 { padding: 60px 20px; }
}

/* Mobile portrait -- 478 */
@media (max-width: 478px) {
  .brxe-abc123 { padding: 24px 12px; }
}
```

If you registered custom breakpoints, query their width via `Breakpoints::get_breakpoint_by('key', 'large_desktop')['width']` from PHP, or just hard-code if it's stable.

## Known gotcha

`_height: 100vh` on mobile browsers includes the address bar, leading to scroll. Use `100svh` or a CSS variable trick. Bricks doesn't auto-correct this -- set the value yourself.
