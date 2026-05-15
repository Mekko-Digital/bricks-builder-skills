# Bricks Native Elements Catalog

73 native elements verified against the Bricks theme source under `{template_dir}/includes/elements/` (locally `{template_dir}/includes/elements\`). Element-class names follow `\Bricks\Element_{Studly_Case}` (e.g. `icon-box` -> `Element_Icon_Box`). Every element extends `\Bricks\Element` (or `\Bricks\Custom_Render_Element` for query-based ones).

> Verify any element's exact controls live: Novamira **Read File** `{template_dir}/includes/elements/{name}.php`, or `Execute PHP` -> `return file_get_contents( get_template_directory() . '/includes/elements/heading.php' );`.

The element name is the string saved as `element.name` in the DB JSON. The category drives the builder panel grouping. The default `tag` is the HTML tag rendered when no `tag`/`customTag` setting is provided.

---

## Layout (4 -- all nestable)

These are the four structural elements. They share the Container control set (flex/grid). Pick by semantic intent, not capability.

| Name | File | Default tag | Children default | Use for |
|---|---|---|---|---|
| `section` | section.php | `section` | one `container` | top-level page sections (semantic) |
| `container` | container.php | `div` (no semantic tag) | empty | flex/grid layout primitive |
| `block` | block.php | `div` | empty | content block, lighter than container |
| `div` | div.php | `div` | empty | minimal-semantics wrapper |

**Common key controls (all four):** `tag`, `customTag`, `link`, `_direction`, `_flexWrap`, `_alignItems`, `_justifyContent`, `_columnGap`, `_rowGap`, `_columnCount` (legacy columns), `_gridTemplateColumns`, `_gridTemplateRows`, `_gridAutoFlow`, `_useMasonry`, `_masonryColumn`, `_masonryGutter`. Container additionally has full background/video/shape-divider controls. See `references/containers-and-layout.md`.

---

## Basic (7)

| Name | File | Default tag | Key controls saved as `settings.{key}` |
|---|---|---|---|
| `heading` | heading.php | `h3` | `text`, `tag` (h1--h6/custom), `customTag`, `type` (hero/lead), `style`, `link`, `separator` (right/left/both/none), `separatorWidth/Height/Spacing/Style/AlignItems/Color` |
| `text` | text.php | `div` | `text` (editor), `type`, `style`, `wordsLimit`, `readMore` |
| `text-basic` | text-basic.php | `div` | plain `text` |
| `text-link` | text-link.php | `a` | `text`, `link` |
| `button` | button.php | `a`/`button` | `text`, `tag`, `size` (xs/sm/md/lg/xl), `style` (primary/secondary/light/dark/etc.), `outline`, `circle`, `block`, `link`, `icon`, `iconTypography`, `iconPosition`, `iconGap` |
| `icon` | icon.php | `i` | `icon` (icon control), `iconColor`, `iconSize`, `link` |
| `image` | image.php | `figure` | `image` (image control: `{id, url, full, size}`), `tag`, `customTag`, `sources` (multi-source), `sizes`, `alt`, `caption`, `link`, `loading` |
| `video` | video.php | `div` | `source` (file/youtube/vimeo), `videoUrl`, `posterImage`, `autoplay`, `muted`, `controls`, `loop`, `width`, `height` |

---

## Layout helpers / general (?30)

| Name | File | Default tag | Nestable | Key controls |
|---|---|---|---|---|
| `accordion` | accordion.php | `div` | no | `accordions` (repeater: `title`, `subtitle`, `anchorId`, `content`), `expandItem`, `independentToggle`, `transition`, `faqSchema` |
| `accordion-nested` | accordion-nested.php | `div` | yes | nested children = panels; `expandItem`, `independentToggle`, `transition`, `faqSchema` |
| `tabs` | tabs.php | `div` | no | `tabs` (repeater: `icon`, `title`, `anchorId`, `content`), `layout`, `accordionLayoutAtBreakpoint` |
| `tabs-nested` | tabs-nested.php | `div` | yes | `direction`, `openTabOn`, `openTab` (default), nested children = tabs |
| `slider` | slider.php | `div` | no | `items` (repeater: `title`, `titleTag`, `content`, `buttonText`, `image`), `type`, `columns`, `arrows`, `dots`, `autoplay` |
| `slider-nested` | slider-nested.php | `div` | yes | `optionsType` (basic/custom), `options` (raw Splide JSON if custom), `type`, `autoplay`, `speed` |
| `carousel` | carousel.php | `div` | no | `type` (media/posts), `items` (image-gallery), `query`, `layout`, `columns`, `arrows`, `dots`, `spaceBetween` |
| `image-gallery` | image-gallery.php | `figure` | no | `images` (image-gallery type), `layout`, `columns`, `lightbox`, `imageLightbox` |
| `nav-menu` | nav-menu.php | `nav` | no | `menu` (WP menu ID), `menuAlignment`, `submenuTrigger` (hover/click), `megamenuColumns`, `mobileMenuEffect` |
| `nav-nested` | nav-nested.php | `nav` | yes | nested children = menu items, `openMenuOn`, `openMenuAtBreakpoint` |
| `dropdown` | dropdown.php | `li` | yes | `text`, `link`, `ariaLabel`, `icon`, `caret`, `megamenu` |
| `offcanvas` | offcanvas.php | `div` | yes | `direction` (left/right/top/bottom), `effect`, `closeOn`, `width`, `height`, `backdropColor` |
| `toggle` | toggle.php | `div` | no | `checked`, `size`, `color` |
| `toggle-mode` | toggle-mode.php | `button` | no | (theme/dark-mode toggle) |
| `divider` | divider.php | `hr` | no | `style`, `thickness`, `width` |
| `icon-box` | icon-box.php | `div` | no | `icon`, `iconPosition` (top/left/right), `title`, `titleTag`, `content`, `link` |
| `social-icons` | social-icons.php | `ul` | no | `items` (repeater: `icon`, `link`, `label`), `layout`, `size`, `gap` |
| `list` | list.php | `ul` | no | `items` (repeater: `text`, `icon`, `link`) |
| `team-members` | team-members.php | `ul` | no | `members` (repeater: `image`, `name`, `role`, `bio`, `social`), `layout`, `columns` |
| `testimonials` | testimonials.php | `ul` | no | `items` (repeater: `image`, `author`, `role`, `content`), `layout`, `columns`, `rating` |
| `pricing-tables` | pricing-tables.php | `ul` | no | `pricingTables` (large repeater) |
| `progress-bar` | progress-bar.php | `div` | no | `value`, `min`, `max`, `animated`, `striped`, `label` |
| `pie-chart` | pie-chart.php | `div` | no | `data`, `options`, `legend`, `tooltip` |
| `counter` | counter.php | `div` | no | `number`, `suffix`, `speed`, `animation` |
| `countdown` | countdown.php | `div` | no | `date`, `timezone`, `format`, `labels` |
| `animated-typing` | animated-typing.php | `div` | no | `typingMode`, `strings` (repeater), `speed`, `backspaceSpeed`, `typeSpeed`, `showCursor` |
| `alert` | alert.php | `div` | no | `content` (editor), `type` (success/info/warning/danger), `dismissible` |
| `breadcrumbs` | breadcrumbs.php | `nav` | no | `maxItems`, `items` (repeater), `homeIcon`, `separator` |
| `back-to-top` | back-to-top.php | `button` | no | `text`, `icon`, `iconPosition`, `offset`, `easing` |
| `rating` | rating.php | `div` | no | `value`, `max`, `size`, `color`, `interactive` |
| `search` | search.php | `form` | no | `placeholder`, `buttonText`, `postTypes`, `submitOnEnter` |
| `logo` | logo.php | `a` | no | `image`, `text`, `link` |
| `code` | code.php | `pre` | no | `code`, `language`, `lineNumbers`, `highlightLines` |
| `html` | html.php | `div` | no | `html` (raw HTML, code editor) |
| `shortcode` | shortcode.php | `div` | no | `shortcode` (string with `[name attr=...]`) |
| `template` | template.php | `div` | no | `templateId`, `dynamicData` (allow DD inside) |
| `svg` | svg.php | `svg` | no | `svg` (raw SVG), `width`, `height` |
| `audio` | audio.php | `div` | no | `source`, `audio`, `title`, `poster`, `autoplay`, `controls`, `loop`, `muted` |
| `instagram-feed` | instagram-feed.php | `div` | no | `accessToken`, `username`, `limit`, `layout`, `columns` |
| `facebook-page` | facebook-page.php | `div` | no | `pageUrl`, `width`, `height`, `tabs`, `smallHeader` |
| `map` | map.php | `div` | no | `address`, `lat`, `lng`, `zoom`, `markers` (repeater), `mapStyle`, `apiKey` |
| `map-leaflet` | map-leaflet.php | `div` | no | `center`, `zoom`, `markers`, `tileLayer`, `attribution` |
| `map-connector` | map-connector.php | `div` | no | (links posts to map markers) |
| `form` | form.php | `form` | no | `fields` (repeater -- see `references/forms.md`), `actions` (array), captchas |

---

## WordPress (4)

| Name | Notes |
|---|---|
| `wordpress` | Generic -- picks shortcode/HTML/dynamic-data |
| `posts` | Custom_Render_Element. `query` + layout + columns + filter + pagination. See `query-loop.md` |
| `related-posts` | `taxonomy`, `queryArgs`, `columns`, `layout`, `imageSize` |
| `sidebar` | `sidebarId`, `before`, `after` |

## Single-post elements (12)

`post-title`, `post-excerpt`, `post-meta`, `post-content`, `post-author`, `post-comments`, `post-taxonomy`, `post-navigation`, `post-sharing`, `post-reading-time`, `post-reading-progress-bar`, `post-toc`. Settings are mostly self-explanatory; `post-meta.metaKey`, `post-taxonomy.taxonomy`, `post-toc.minHeadings`/`excludeHeadings`/`smooth`. `post-content` has no settings -- it just renders `the_content()`.

## Query helpers (2)

`pagination` (style, pageNumbers, arrows, prevText, nextText), `query-results-summary` (format, showCount).

## Filter elements (8 -- only loaded if query filters are enabled)

`filter-checkbox`, `filter-radio`, `filter-select`, `filter-search`, `filter-range`, `filter-datepicker`, `filter-submit`, `filter-active-filters`. Common: `filterKey`, `queryId` (target query loop element ID), `options` (repeater for checkbox/radio/select). See `references/query-loop.md` ? Filters.

---

## Special / abstract (don't instantiate)

- `base.php` -- abstract base class
- `custom-render-element.php` -- abstract base for query elements
- `filter-base.php` -- abstract base for filter elements
- `slot.php` -- component-system slot placeholder

---

## Component instances

A component instance has the same JSON shape as any element PLUS:
```json
{ "id":"abc12", "cid":"cmp-xyz", "instanceId":"1", "name":"container", "settings": { /* overrides only */ } }
```
The component definition lives in `bricks_components` option keyed by `cid` and contains the full element tree. The instance's `id` is **6 chars** and the `uid` is computed at render: `{cid}-{instanceId}` for nested, else `{id}-{instanceId}`.

---

## Control type registry (the `'type'` field on every control)

Verified across all element files. When you write a control, pick from this list -- anything else won't render.

| Type | Saved value shape | Use |
|---|---|---|
| `text` | `"string"` | single-line input; pair with `'hasDynamicData' => 'text'` to allow DD tokens |
| `textarea` | `"string"` | multi-line plain text |
| `editor` | `"<p>...</p>"` | TinyMCE WYSIWYG |
| `code` | `"..."` (with `mode` option: `js`/`css`/`html`/`php`) | code editor |
| `number` | `42` or `"10px"` (when `units: true`) | numeric, optional units |
| `select` | `"option_key"` | dropdown; pass `options` as `[ key => label ]` |
| `checkbox` | `true`/`false` | boolean toggle |
| `color` | `{ "hex":"#FF0000" }` or `{ "rgb":"rgba(...)" }` or `{ "raw":"var(--x)" }` | color picker |
| `gradient` | `{ "type":"linear", "angle":90, "stops":[...] }` | gradient editor |
| `image` | `{ "id":123, "url":"...", "full":"...", "size":"large" }` | single image |
| `image-gallery` | `[ {...}, {...} ]` | multi-image |
| `icon` | `{ "library":"themify", "icon":"ti-star" }` | icon picker |
| `link` | `{ "type":"external\|internal\|lightbox", "url":"...", "newTab":true, "rel":"..." }` | link builder |
| `audio` / `video` / `svg` | source-specific objects / raw text | media |
| `typography` | `{ "font-family":"...", "font-size":"16px", "font-weight":"400", "line-height":"1.5", "letter-spacing":"0", "text-align":"left", "text-transform":"uppercase", "text-decoration":"underline", "color":{"hex":"..."} }` | text styling |
| `border` | `{ "width":{ "top":"1px", "right":"1px", "bottom":"1px", "left":"1px" }, "style":"solid", "color":{...}, "radius":{ "top":"4px",... } }` | border |
| `spacing` | `{ "top":"10px","right":"10px","bottom":"10px","left":"10px" }` | margin/padding |
| `dimensions` | `{ "width":"100%","height":"auto" }` | sizing |
| `box-shadow` | `{ "values":{ "offsetX":"0","offsetY":"2px","blur":"5px","spread":"0" }, "color":{...}, "inset":false }` | shadow |
| `text-shadow` | `{ "x":"1px","y":"1px","blur":"2px","color":{...} }` | text shadow |
| `background` | `{ "color":{...}, "image":{...}, "video":{...}, "gradient":{...}, "position":"...", "size":"cover", "repeat":"no-repeat", "attachment":"scroll", "parallax":false }` | background |
| `align-items` | `"flex-start" \| "center" \| "flex-end" \| "stretch" \| "baseline"` | flex cross-axis |
| `justify-content` | `"flex-start" \| "center" \| "flex-end" \| "space-between" \| "space-around" \| "space-evenly"` | flex main-axis |
| `direction` | `"row" \| "column" \| "row-reverse" \| "column-reverse"` | flex-direction |
| `text-align` | `"left" \| "center" \| "right" \| "justify"` | text alignment |
| `transform` | `{ "translate":{"x":"10px","y":"0"}, "rotate":"45deg", "scale":{"x":1,"y":1}, "skew":{"x":0,"y":0} }` | CSS transform |
| `transition` | `{ "property":"all","duration":"0.3s","timing":"ease","delay":"0s" }` | CSS transition |
| `repeater` | `[ { /* per-item fields */ } ]` | array of sub-objects; define `'fields' => [...]` |
| `query` | `{ "objectType":"post","post_type":"post","posts_per_page":10,... }` | WP_Query builder |
| `filters` | (internal) | filter binding |
| `datepicker` | `"2024-05-08"` or `"2024-05-08 14:30"` | Flatpickr |
| `separator` / `info` | (no value) | UI-only divider/note |
| `apply` / `comment` / `monthly` / `external` | rare/internal | -- |

---

## `control_options` registry (preset option lists you can pull into your custom element)

Defined in `{template_dir}/includes/setup.php` under `Setup::get_control_options()`. Reference inside a control via `'options' => $this->control_options['...']`. Available keys:

`backgroundPosition`, `backgroundRepeat`, `backgroundSize`, `backgroundAttachment`, `backgroundVideoPosterYouTubeSize`, `blendMode`, `buttonSizes`, `styles` (used by Heading "style" + Button), `borderStyle`, `fontWeight`, `fontStyle`, `iconPosition`, `objectFit`, `position`, `queryTypes`, `queryOrder`, `queryOrderBy`, `termsOrderBy`, `usersOrderBy`, `queryCompare`, `queryOperator`, `queryValueType`, `templatesOrderBy`, `templateTypes`, `animationTypes` (the entire Animate.css set -- see `interactions-and-animations.md`), `lightboxAnimationTypes`, `ajaxLoaderAnimations`, `textWrap`, `whiteSpace`, `imageSizes`, `taxonomies`, `userRoles`, `allSectionTemplates`.

---

## Filters for extending elements

```php
// Add element name to the registered list (still need a class/file via register_element)
add_filter( 'bricks/builder/elements', fn( $names ) => array_merge( $names, ['my-thing'] ) );

// Modify any element's controls
add_filter( 'bricks/elements/heading/controls', fn( $c ) => $c );
add_filter( 'bricks/elements/heading/control_groups', fn( $g ) => $g );
add_filter( 'bricks/elements/heading/scripts', fn( $s ) => $s );
```
