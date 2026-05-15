# Interactions, Animations, GSAP, Popups, Load-More

All animation/interaction logic that Bricks ships natively goes through the **Interactions** system. Bricks does NOT bundle GSAP -- for GSAP you self-enqueue from the child theme.

## 1. Interactions JSON (`settings._interactions`)

Source: `{template_dir}/includes/interactions.php`. Each element's interactions are an array of objects:

```jsonc
"_interactions": [
  {
    "id": "intxyz",                       // generated id
    "trigger": "click",                   // see TRIGGERS below
    "action":  "toggleOffCanvas",         // see ACTIONS below
    "target":  "self",                    // self | custom | popup | offcanvas
    "targetSelector": ".my-modal",        // CSS selector (when target=custom)
    "templateId": 1234,                   // for popup/template targets
    "runOnce":   false,
    "delay":     "0",                     // ms or "1s"
    "disablePreventDefault": false,
    "rootMargin":   "0px 0px -10% 0px",   // enterView/leaveView IntersectionObserver
    "scrollOffset": "100",                // scroll trigger threshold
    "animationType":     "fadeInUp",      // for startAnimation
    "animationDuration": "0.8s",
    "animationDelay":    "0.1s",
    "scrollToOffset":    100,             // px above element when scrollTo
    "scrollToDelay":     0,
    "jsFunction":        "myCustomFunc",  // for action: 'javascript'
    "jsFunctionArgs":    [ /* ... */ ],
    "storageType":       "localStorage",  // for storageAdd/Remove/Count
    "actionAttributeKey":  "data-state",
    "actionAttributeValue":"open",
    "loadMoreQuery":       "main",        // or query-element ID
    "loadMoreTargetSelector": ".gallery-grid",
    "offCanvasSelector":  "#brxe-offcanvas-id",
    "filterElementId":    "abc123",
    "ajaxQueryId":        "main",
    "formId":             "#brxe-form-id",
    "targetFormSelector": "form",
    "popupContextType":   "post",
    "popupContextId":     "42",
    "interactionConditions": [ /* same shape as conditions */ ],
    "interactionConditionsRelation": "and"
  }
]
```

### Triggers (the `trigger` field)

| Trigger | Fires when | Extra fields |
|---|---|---|
| `click` | element clicked | -- |
| `mouseover` | mouse over | -- |
| `mouseenter` / `mouseleave` | enter/leave | -- |
| `focus` / `blur` | focus changes (keyboard or programmatic) | -- |
| `enterView` / `leaveView` | IntersectionObserver crosses viewport | `rootMargin` |
| `animationEnd` | CSS animation completes | optional `animationId` |
| `scroll` | window scroll | `scrollOffset` |
| `contentLoaded` | DOMContentLoaded | optional `delay` |
| `mouseleaveWindow` | exit-intent | -- |
| `ajaxStart` / `ajaxEnd` | query AJAX cycle | `ajaxQueryId` |
| `formSubmit` / `formSuccess` / `formError` | form lifecycle | `formId` |
| `filterOptionEmpty` / `filterOptionNotEmpty` | query filter result count | `filterElementId` |
| `wooAddedToCart` / `wooAddingToCart` / `wooRemovedFromCart` / `wooUpdateCart` / `wooCouponApplied` / `wooCouponRemoved` | WooCommerce events | -- |

### Actions (the `action` field)

| Action | Effect | Required fields |
|---|---|---|
| `show` | adds visibility class | `target`, `targetSelector` |
| `hide` | adds `hide` class | `target`, `targetSelector` |
| `click` | simulate click | `target`, `targetSelector` |
| `setAttribute` / `removeAttribute` / `toggleAttribute` | mutate HTML attribute | `actionAttributeKey`, `actionAttributeValue` |
| `toggleOffCanvas` | open/close offcanvas | `offCanvasSelector` |
| `loadMore` | next page in query loop | `loadMoreQuery` (= `"main"` or query element ID) |
| `loadMoreGallery` | append more images | `loadMoreTargetSelector` |
| `startAnimation` | run an Animate.css animation | `animationType`, `animationDuration`, `animationDelay`, `target`, `targetSelector` |
| `scrollTo` | smooth-scroll to element | `target`, `targetSelector`, `scrollToOffset`, `scrollToDelay` |
| `javascript` | call a global JS function | `jsFunction`, `jsFunctionArgs` |
| `openAddress` / `closeAddress` | map info-box open/close | `infoBoxId` |
| `clearForm` | reset form fields | `targetFormSelector` |
| `storageAdd` / `storageRemove` / `storageCount` | mutate browser storage | `storageType` (`window\|session\|localStorage`), `actionAttributeKey`, `actionAttributeValue` |

### Per-interaction conditions

`interactionConditions` is the same condition format as in `references/conditions.md`. The most useful condition types here are storage-checks (`windowStorage`/`sessionStorage`/`localStorage`) for "show this once per session" patterns.

### Popup actions (open/close popup)

Popups are templates -- to open a popup, set `action: 'click'` (or other trigger) **plus** `target: 'popup'` and `templateId: <popup-template-id>`. Closing is via `target: 'popup'` and `targetSelector` matching the popup, or by `clickClose` inside the popup itself. See `references/popups-and-templates.md`.

## 2. CSS animations (Animate.css names)

Bricks ships the Animate.css library. The full set is registered as `control_options.animationTypes` in `{template_dir}/includes/setup.php`. Apply via `startAnimation` action's `animationType` field (or via the deprecated `_animation` settings -- prefer interactions).

**Attention seekers:** `bounce`, `flash`, `pulse`, `rubberBand`, `shakeX`, `shakeY`, `headShake`, `swing`, `tada`, `wobble`, `jello`, `heartBeat`.

**Entrances:**
- Back: `backInDown`, `backInLeft`, `backInRight`, `backInUp`
- Bounce: `bounceIn`, `bounceInDown`, `bounceInLeft`, `bounceInRight`, `bounceInUp`
- Fade: `fadeIn`, `fadeInDown`, `fadeInDownBig`, `fadeInLeft`, `fadeInLeftBig`, `fadeInRight`, `fadeInRightBig`, `fadeInUp`, `fadeInUpBig`, `fadeInTopLeft`, `fadeInTopRight`, `fadeInBottomLeft`, `fadeInBottomRight`
- Flip: `flip`, `flipInX`, `flipInY`
- Light Speed: `lightSpeedInRight`, `lightSpeedInLeft`
- Rotate: `rotateIn`, `rotateInDownLeft`, `rotateInDownRight`, `rotateInUpLeft`, `rotateInUpRight`
- Slide: `slideInUp`, `slideInDown`, `slideInLeft`, `slideInRight`
- Zoom: `zoomIn`, `zoomInDown`, `zoomInLeft`, `zoomInRight`, `zoomInUp`
- Other: `jackInTheBox`, `rollIn`

**Exits** (same families, swap In->Out): `fadeOut`, `bounceOut*`, `slideOut*`, `zoomOut*`, `rotateOut*`, `lightSpeedOut*`, `flipOut*`, `backOut*`, `rollOut`, `hinge`.

Animation duration accepts CSS values: `0.3s`, `500ms`, etc. Delay is the same. Bricks injects them as inline `--animate-duration` / `--animate-delay` CSS vars.

## 3. Element parallax (built-in motion)

Without GSAP, you have native parallax via:

```jsonc
"_motionElementParallax":          true,
"_motionElementParallaxSpeedX":    50,           // 0--100, percent of scroll distance
"_motionElementParallaxSpeedY":    50,
"_motionBackgroundParallax":       true,
"_motionBackgroundParallaxSpeed":  30,
"_motionStartVisiblePercent":      0              // 0 = start at viewport bottom, 50 = midpoint
```

Frontend implementation: `[data-brx-motion-parallax]` selector, shared RAF tick, passive scroll listener, respects `prefers-reduced-motion`. Element parallax is independent from background parallax -- set both for a hero with both.

## 4. CSS transitions and filters

```jsonc
"_cssTransition": "all 0.3s ease",
"_cssFilters": [
  { "type": "blur",       "value": "0px" },
  { "type": "brightness", "value": "100" },
  { "type": "contrast",   "value": "100" },
  { "type": "grayscale",  "value": "0" },
  { "type": "hue-rotate", "value": "0deg" },
  { "type": "invert",     "value": "0" },
  { "type": "opacity",    "value": "100" },
  { "type": "saturate",   "value": "100" },
  { "type": "sepia",      "value": "0" }
]
```

## 5. Hover effects

There is no first-class "hover effect" control. Two patterns are used:

1. **Pseudo state in panel:** the builder switches the active pseudo-class to `:hover` and you set `_typography.color` etc. -- Bricks generates `.brxe-id:hover { ... }`.
2. **`_cssCustom` on element** with `%root%:hover` selector -- most reliable for complex multi-property hover sequences:

```css
%root%             { transition: transform 0.4s ease; }
%root%:hover       { transform: translateY(-4px); }
%root%:hover .icon { color: var(--accent); }
```

## 6. GSAP

GSAP is **not bundled** by Bricks. The child theme already enqueues `js/history-bricks.js` and `js/roi-calculator-bricks.js` via `wp_enqueue_script` for specific pages -- that is the established pattern for bespoke GSAP/ScrollTrigger work.

To add GSAP:

```php
// In bricks-child/functions.php enqueue closure
wp_enqueue_script(
  'gsap',
  'https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/gsap.min.js',
  [],
  '3.12.5',
  [ 'strategy' => 'defer', 'in_footer' => true ]
);
wp_enqueue_script(
  'gsap-scrolltrigger',
  'https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/ScrollTrigger.min.js',
  [ 'gsap' ],
  '3.12.5',
  [ 'strategy' => 'defer', 'in_footer' => true ]
);

wp_enqueue_script(
  'sl-page-foo',
  $uri . '/js/foo.js',
  [ 'gsap', 'gsap-scrolltrigger' ],
  filemtime( $dir . '/js/foo.js' ),
  [ 'strategy' => 'defer', 'in_footer' => true ]
);
```

Inside `js/foo.js`, target elements by their **stable Bricks ID**:

```js
gsap.registerPlugin(ScrollTrigger);

// Bricks elements get a class .brxe-{id}; use that.
gsap.from('.brxe-abc123 .stat-value', {
  scrollTrigger: { trigger: '.brxe-abc123', start: 'top 80%' },
  textContent: 0,
  duration: 2,
  ease: 'power2.out',
  snap: { textContent: 1 }
});
```

If you can't pin a stable ID (because the element is a component instance with rotating IDs), add a custom class via `_cssClasses` and target that.

**Key rules for GSAP in Bricks:**
- Always `defer` the script; Bricks runs its own DOMContentLoaded init that you don't want to race with.
- Respect `prefers-reduced-motion` -- `if (matchMedia('(prefers-reduced-motion: reduce)').matches) return;` at top of the file.
- Make idempotent -- Bricks builder iframe re-runs scripts on settings change. Use `ScrollTrigger.getById(id)` and `kill()` previous before recreating.
- Don't enqueue in the **builder main panel** (`bricks_is_builder_main()` early return -- already in `bricks-child/functions.php`).
- Per-page JS is enqueued only when `is_page('slug')` matches -- that's the pattern in this child.

## 7. Load more & infinite scroll (query loops)

Two paths:

**Infinite scroll (auto):** set `_query.infinite_scroll: true` on the loop element (posts/products/carousel). Frontend uses an IntersectionObserver on the pagination area.

**Manual "Load more" button:** create any element (button, link), add an interaction:
```jsonc
{
  "trigger": "click",
  "action":  "loadMore",
  "loadMoreQuery": "main"   // or the query element's id
}
```
`"main"` resolves at render to the active main query. The button hides itself automatically when no more pages.

**AJAX loader:** set `_query.ajax_loader: true` to show a loading indicator during AJAX fetches. Configure animation with `control_options.ajaxLoaderAnimations`.

**Query filters:** the filter elements (`filter-checkbox`, etc.) bind to a query element by ID. The interaction for binding is internal -- you set `queryId` on the filter element and Bricks dispatches AJAX events. Use `filterOptionEmpty` / `filterOptionNotEmpty` triggers to react.

## 8. Popups

A popup is a template post with:
- `post_type: bricks_template`
- `_bricks_template_type = 'popup'`

The popup's content is the template's `_bricks_page_content_2` -- built like any page.

To open a popup from any element:
```jsonc
"_interactions": [
  { "trigger":"click", "action":"click", "target":"popup", "templateId": 1234 }
]
```
(Some Bricks versions use `"action":"openPopup"` literally; verify against `interactions.php` in your installed version. The data attribute Bricks renders is `data-popup-id`.)

To close from inside the popup, add an element with:
```jsonc
"_interactions": [
  { "trigger":"click", "action":"hide", "target":"popup", "targetSelector":"closest" }
]
```
Or rely on the popup's built-in close-on-backdrop / close-on-ESC settings.

Popup template settings (stored as `_bricks_template_settings` on the popup post):
- `popupCloseOn` (`backdrop`/`esc`/`none`/array combo)
- `popupBodyScroll` (boolean)
- `popupZindex` (default 10000)
- `popupShowOn` / `popupShowAt` (breakpoint visibility)
- `popupBackground` (full background control for backdrop)
- `popupContentWidth` / `popupContentHeight` / `popupContentPadding`
- `popupLimitWindow` / `popupLimitSessionStorage` / `popupLimitLocalStorage` / `popupLimitTimeStorage` (frequency caps)

See `{template_dir}/includes/popups.php` for the full list.

## 9. The deprecated `_animation*` controls

The base class still defines `_animation`, `_animationDuration`, `_animationDelay`, `_animationStagger` for backwards compat. **Don't use these in new work.** Bricks recommends migrating via the built-in converter to interactions. If you encounter them in old templates, expect them to render fine but plan a conversion.

## 10. Quick recipes

**Reveal on scroll:**
```jsonc
"_interactions": [
  {
    "trigger": "enterView",
    "action":  "startAnimation",
    "target":  "self",
    "animationType":     "fadeInUp",
    "animationDuration": "0.8s",
    "animationDelay":    "0s",
    "rootMargin":        "0px 0px -10% 0px",
    "runOnce":           true
  }
]
```

**Show offcanvas on click:**
```jsonc
"_interactions": [
  { "trigger": "click", "action": "toggleOffCanvas", "offCanvasSelector": "#brxe-mymenu" }
]
```

**Open popup, suppress for 24h after first close:**
```jsonc
"_interactions": [
  {
    "trigger": "contentLoaded", "delay": "3000",
    "action":  "click", "target": "popup", "templateId": 1234,
    "interactionConditions": [
      { "conditionType": "localStorage", "storageKey": "popupSeen", "storageCompare": "notExists" }
    ]
  }
]
```
And on the popup's close button:
```jsonc
{ "trigger":"click", "action":"storageAdd", "storageType":"localStorage",
  "actionAttributeKey":"popupSeen", "actionAttributeValue":"1" }
```
