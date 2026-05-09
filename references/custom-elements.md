# Custom Elements (Child-Theme Registration)

You add custom Bricks elements in the child theme. The pattern is already established in `{stylesheet_dir}/functions.php:104` and `{stylesheet_dir}/elements\*.php`. **Don't deviate from this pattern** unless you have a specific reason -- Bricks's element loader expects exact class shapes.

## 1. Registration

> **Verify the active child path first** with Novamira: `Execute PHP` -> `return get_stylesheet_directory();`. Write your new element file under `{that}/elements/{name}.php` via Novamira **Write File**, then **Edit File** the registration array in `functions.php`.

In `bricks-child/functions.php`, on `init` priority 11 (after Bricks loads its own elements):

```php
add_action( 'init', function() {
  $element_files = [
    __DIR__ . '/elements/title.php',
    __DIR__ . '/elements/primary-nav.php',
    __DIR__ . '/elements/my-new-element.php',
  ];

  foreach ( $element_files as $file ) {
    if ( file_exists( $file ) ) {
      \Bricks\Elements::register_element( $file );
    }
  }
}, 11 );
```

Custom category label (so they appear under "Custom" in the panel):

```php
add_filter( 'bricks/builder/i18n', function( $i18n ) {
  $i18n['custom'] = esc_html__( 'Custom', 'bricks' );
  return $i18n;
} );
```

## 2. Element class skeleton

```php
<?php
if ( ! defined( 'ABSPATH' ) ) exit;

class Element_My_Thing extends \Bricks\Element {
  public $category     = 'custom';                  // panel group key
  public $name         = 'my-thing';                // saved into element.name in DB
  public $icon         = 'fas fa-anchor';            // FontAwesome 5 / Themify
  public $css_selector = '.my-thing-wrapper';        // default selector for controls with 'css'
  public $tag          = 'div';                      // default HTML tag

  // Optional. Only set if your element wraps other elements.
  // public $nestable = true;

  public function get_label()    { return esc_html__( 'My Thing', 'bricks' ); }
  public function get_keywords() { return [ 'thing', 'demo' ]; }

  public function set_control_groups() {
    $this->control_groups['content'] = [
      'title' => esc_html__( 'Content', 'bricks' ),
      'tab'   => 'content',
    ];
  }

  public function set_controls() {
    $this->controls['title'] = [
      'tab'            => 'content',
      'group'          => 'content',
      'label'          => esc_html__( 'Title', 'bricks' ),
      'type'           => 'text',
      'hasDynamicData' => 'text',          // allow {dynamic_tags} in this field
      'default'        => esc_html__( 'I am a thing', 'bricks' ),
      'placeholder'    => esc_html__( 'Title', 'bricks' ),
    ];

    $this->controls['titleColor'] = [
      'tab'   => 'content',
      'group' => 'content',
      'label' => esc_html__( 'Title color', 'bricks' ),
      'type'  => 'color',
      'css'   => [
        [ 'property' => 'color', 'selector' => '.my-thing-title' ],
      ],
    ];
  }

  public function render() {
    $settings = $this->settings;
    $title    = $settings['title'] ?? '';

    if ( ! $title ) {
      return $this->render_element_placeholder( [
        'icon-class' => 'ti-paragraph',
        'title'      => esc_html__( 'Add a title.', 'bricks' ),
      ] );
    }

    $this->set_attribute( '_root', 'class', 'my-thing-wrapper' );
    $title = $this->render_dynamic_data( $title );

    echo "<div {$this->render_attributes( '_root' )}>";
    echo "<h4 class=\"my-thing-title\">{$title}</h4>";
    echo "</div>";
  }
}
```

## 3. Class & file naming rules (verified)

The Bricks loader in `{template_dir}/includes/elements.php:170-178` constructs the class name from the file's element name:
```php
$class_name = str_replace( '-', '_', $element_name );
$class_name = ucwords( $class_name, '_' );
$class_name = "Bricks\\Element_$class_name";
```

This is for **built-in** elements. For custom elements registered via `Elements::register_element($file)` **without** an explicit class name, the loader takes `end(get_declared_classes())` -- i.e. the **last declared class** in your file. So:

- **Use any class name you want** as long as it's the only/last class declared in the file.
- The class **must extend** `\Bricks\Element` (or `\Bricks\Custom_Render_Element` for query-driven loops).
- The file **must define `$name`** as a unique string. This is the `name` value saved into element JSON in the DB.

If you want explicit registration (clearer):
```php
\Bricks\Elements::register_element( $file, 'my-thing', '\\Bricks\\Element_My_Thing' );
```

## 4. Available `$this` properties (read & set)

From `{template_dir}/includes/elements/base.php`:

- `$this->settings` -- element settings JSON (read-only here; changing has no DB effect at render).
- `$this->theme_styles` -- applied theme-style block for this element type.
- `$this->id` -- 6-char element ID.
- `$this->cid`, `$this->uid` -- component IDs (when in a component).
- `$this->is_frontend` -- `true` on frontend, `false` in builder.
- `$this->original_query` -- set by parent loops; useful when your element runs inside a loop.

Methods you'll use in `render()`:

- `$this->set_attribute( $key, $name, $value )` -- adds an HTML attribute to a render group. `_root` is the outer wrapper.
- `$this->render_attributes( $key )` -- renders the group as `name="value" ...`. Pass `true` as 2nd arg to include user-set `_attributes` repeater.
- `$this->render_dynamic_data( $string )` -- resolve `{tag}` tokens in any string.
- `$this->render_element_placeholder( $args )` -- outputs the empty-state placeholder block in the builder.
- `$this->set_link_attributes( $key, $link_setting )` -- fills href/target/rel from a `link` control's value.
- `$this->set_attribute( '_root', 'data-bricks-something', wp_json_encode( $config ) )` -- pass server-rendered config to JS.

## 5. Nestable custom element

Set `public $nestable = true;` and provide:

```php
public function get_nestable_item() {
  // What a single repeater item looks like -- used when adding a new child.
  return [
    'name'      => 'block',
    'label'     => esc_html__( 'Tab', 'bricks' ),
    'settings'  => [],
    'children'  => [
      [ 'name'=>'heading', 'settings'=>[ 'text'=>'Tab heading', 'tag'=>'h3' ] ],
      [ 'name'=>'text',    'settings'=>[ 'text'=>'<p>Content...</p>' ] ],
    ],
  ];
}

public function get_nestable_children() {
  // Default children when this element is first added to canvas.
  return [
    $this->get_nestable_item(),
    $this->get_nestable_item(),
  ];
}
```

For non-layout nestables (slider/tabs/accordion variants), set:

```php
public $nestable_html = '<div class="my-tabs">{children}</div>';
```

The `{children}` placeholder is replaced with rendered children at render time.

## 6. Builder Vue render (optional, for instant preview)

If you implement `render_builder()`, the builder uses a Vue template instead of AJAX-rendering on every settings change:

```php
public static function render_builder() { ?>
  <script type="text/x-template" id="tmpl-bricks-element-my-thing">
    <component :is="tag" class="my-thing-wrapper">
      <contenteditable
        v-if="settings.title"
        tag="h4"
        :name="name"
        :settings="settings"
        controlKey="title"
        class="my-thing-title" />
    </component>
  </script>
<?php }
```

Use `<contenteditable>` for inline-editable text. The `controlKey` must match a control of type `text`/`textarea`/`editor` in `set_controls()`.

If you skip `render_builder()`, the builder calls `render()` via AJAX on each change. That's slower but always accurate.

## 7. Enqueueing scripts/styles

The element class can declare scripts to run on its rendered HTML:

```php
public $scripts = [ 'slMyThingInit' ];
```

Then in your enqueued JS bundle (loaded site-wide):
```js
window.bricksData = window.bricksData || {};
window.slMyThingInit = function() {
  document.querySelectorAll('.my-thing-wrapper').forEach( el => { /* ... */ });
};
```

Bricks runs all `$scripts` for every element on `BricksFunctions.run` after the page mounts and after AJAX-loaded content (loop pagination, popup AJAX, query filters). That makes them idempotent-friendly: re-init from scratch each time.

For per-page JS (the existing pattern in this child), enqueue conditionally on `is_page('slug')` from `bricks-child/functions.php`.

## 8. Per-element CSS (controls with `'css'`)

Any control with a `'css'` array generates CSS automatically:

```php
$this->controls['titleColor'] = [
  'type'  => 'color',
  'css'   => [
    [ 'property' => 'color', 'selector' => '.my-thing-title' ],
  ],
];
```

The selector is **relative to the element wrapper** (the `.brxe-{id}`). To target the wrapper itself, use `''` (empty) or `&`. To select children, write `'.my-thing-title'`. To target a parent of the wrapper, you can't via CSS controls -- use `_cssCustom`.

If your element uses a single primary inner selector for most controls, set `public $css_selector = '.my-thing-wrapper'` and Bricks fills in any control whose `'css'` array doesn't specify `selector`.

## 9. Dynamic data inside controls

Mark a control with `'hasDynamicData' => 'text'` (or `'image'`, `'media'`, `'link'`) to enable the DD picker UI. In `render()` always pass user input through `$this->render_dynamic_data($value)` before output.

## 10. Required, conditional, repeater controls

```php
$this->controls['variant'] = [
  'type'    => 'select',
  'label'   => 'Variant',
  'options' => [ 'default' => 'Default', 'fancy' => 'Fancy' ],
  'default' => 'default',
];

$this->controls['fancyParticles'] = [
  'type'     => 'number',
  'label'    => 'Particle count',
  'required' => [ 'variant', '=', 'fancy' ],   // show only when variant=fancy
];

$this->controls['items'] = [
  'type'     => 'repeater',
  'label'    => 'Items',
  'titleProperty' => 'label',                   // shown as repeater item title
  'fields'   => [
    'label' => [ 'type' => 'text',  'label' => 'Label' ],
    'icon'  => [ 'type' => 'icon',  'label' => 'Icon' ],
    'link'  => [ 'type' => 'link',  'label' => 'Link' ],
  ],
];
```

`required` operators: `=`, `!=`, `>`, `<`, `>=`, `<=`, `contains`, `!contains`. Multiple conditions = nested array.

## 11. Filters available to your element

```php
add_filter( "bricks/elements/{$this->name}/controls",       $cb );
add_filter( "bricks/elements/{$this->name}/control_groups", $cb );
add_filter( "bricks/elements/{$this->name}/scripts",        $cb );
```

These let third parties extend your element without editing its code.

## 12. Reading existing custom elements in this child

The child theme has 7 custom elements you can use as patterns:
- `title.php`            -- basic content + typography pattern
- `primary-nav.php`      -- complex header/menu element
- `faq-accordion.php`    -- expand/collapse with custom JS hook
- `logo-marquee.php`     -- image scroller / animation
- `stats-counter.php`    -- number animation tied to in-view trigger
- `icon.php`             -- icon picker with custom render
- `roi-calculator.php`   -- interactive calculator with form-like UI

Read them before building anything similar -- they reflect this codebase's house style.

## 13. Pitfalls

- **Don't re-declare a class name** that already exists in another element file. Bricks does `end(get_declared_classes())` and the wrong file may register against the wrong name if class names collide.
- **Always escape output.** `$this->render_dynamic_data()` doesn't sanitize HTML -- apply `esc_html()`/`wp_kses_post()` as appropriate.
- **The element **name** ('my-thing') must be unique across the whole site** -- including across plugins that register Bricks elements.
- **Don't query in `set_controls()`** -- that runs on every builder load. Use `bricks_is_builder()` guards or move expensive options into `add_filter('bricks/elements/{name}/controls')`.
- **Builder vs frontend rendering branch:** `if ( $this->is_frontend )` for behavior that should not run in the builder.
- **Don't enqueue child JS into the builder main panel** -- guard with `bricks_is_builder_main()` early-return (existing pattern).
