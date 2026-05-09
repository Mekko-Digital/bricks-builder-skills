# Bricks Builder Skill for Claude Code

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) that gives Claude deep, verified knowledge of the [Bricks Builder](https://bricksbuilder.io) WordPress theme. With this skill installed, Claude can read and write Bricks page JSON, register custom elements, configure theme styles and breakpoints, build forms and query loops, wire interactions and popups -- all using **the exact control keys, value shapes, and naming conventions Bricks actually uses in the database**.

> **Why it exists.** LLMs hallucinate Bricks control keys (`_bgColor` instead of `_background`, `padding` instead of `_padding`, made-up animation names). This skill grounds every recommendation in the actual Bricks source, so you stop debugging settings that don't exist.

## What's inside

```
bricks-builder-skills/
+-- SKILL.md                          # entry point Claude loads automatically
+-- references/
    +-- elements-catalog.md           # all 73 native elements + 39 control types
    +-- element-base-controls.md      # _typography, _padding, _background, etc.
    +-- containers-and-layout.md      # section/container/block/div, flex, grid, masonry
    +-- interactions-and-animations.md# triggers, actions, Animate.css names, GSAP pattern
    +-- responsive-breakpoints.md     # :breakpoint_key syntax, mobile-first, custom bps
    +-- theme-styles-and-globals.md   # theme styles, classes, variables, palettes, components
    +-- forms.md                      # every field type and action (mailchimp, webhook, --)
    +-- query-loop.md                 # post/term/user/woo/api queries, filters, pagination
    +-- conditions.md                 # show/hide logic, all condition keys + operators
    +-- dynamic-data.md               # {tag} syntax, ACF/MetaBox/JetEngine/Pods integration
    +-- popups-and-templates.md       # template types, assignment, popup config
    +-- custom-elements.md            # full child-theme element registration guide
    +-- db-schema.md                  # every BRICKS_DB_* constant + JSON shapes
    +-- novamira-verification.md      # live-WP verification recipes via Novamira MCP
    +-- quick-reference.md            # cheat sheet -- class names, helpers, JS globals
```

Around **3,500 lines** of reference material, all extracted from Bricks 2.x source files. Verified, not generated.

## Recommended setup: pair with Novamira MCP

This skill is **designed to work alongside [Novamira MCP](https://github.com/use-novamira/novamira)**. Novamira gives Claude live access to your running WordPress install (PHP execution, file ops). Together they let Claude:

- Verify every Bricks control key against the **live theme files** instead of trusting static docs.
- Read what's actually saved in the database for any page or template.
- Write new custom-element files directly into your active child theme.
- Discover dynamic-data tags added by your installed plugins (ACF / Meta Box / JetEngine / Pods).

Without Novamira the skill still works -- it falls back to static references that ship in this repo. With Novamira installed, references stop being a snapshot and start being a verification baseline against your actual install. **Strong recommendation: install both.**

Install Novamira from its docs: [novamira.ai/docs/getting-started](https://novamira.ai/docs/getting-started/), then this skill picks it up automatically (no skill config needed).

## Install

You need [Claude Code](https://claude.com/claude-code) (or any Anthropic agent runtime that supports the [Skill format](https://docs.claude.com/en/docs/claude-code/skills)).

### Option A -- Project-scoped (recommended)

Drop the skill into your WordPress project root (the directory you open Claude Code from):

```bash
cd /path/to/your-wp-project
mkdir -p .claude/skills
git clone https://github.com//bricks-builder-skills.git \
  .claude/skills/bricks-builder-skills
```

The skill loads automatically the next time you start Claude Code in that directory. Trigger it by mentioning Bricks ("build a hero section", "register a custom element", "add a popup") -- Claude will pull in `SKILL.md` and the relevant reference on demand.

### Option B -- User-scoped (available everywhere)

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/Mekko-Digital/bricks-builder-skills.git \
  ~/.claude/skills/bricks-builder-skills
```

The skill is now available in every Claude Code session.

### Option C -- Submodule

```bash
git submodule add https://github.com/Mekko-Digital/bricks-builder-skills.git \
  .claude/skills/bricks-builder-skills
```

## Verify it loaded

Inside Claude Code, ask:

```
What skills are available?
```

You should see `bricks-builder-skills` listed. Or just ask Claude to do anything Bricks-related and watch it reference the skill.

## What it gives you

After installation, Claude can:

- **Build pages** by writing valid `_bricks_page_content_2` JSON -- every element, with correct control keys.
- **Register custom elements** following the Bricks child-theme pattern (the existing class shape, registration on `init` priority 11, etc.).
- **Configure theme styles** with the right per-element control sections (`element-button`, `element-form`, etc.).
- **Add responsive overrides** with the correct `:breakpoint_key` suffix syntax.
- **Wire interactions** -- triggers (click/hover/scroll/enterView/formSubmit/woo*/etc.) and actions (show/hide/toggle/loadMore/startAnimation/scrollTo/storage*/etc.).
- **Build forms** with every supported field type and action (email, webhook, mailchimp, sendgrid, registration, login, create-post, custom code, save-submission).
- **Configure query loops** for post / term / user / WooCommerce / API / array / ACF & Meta Box relationships.
- **Apply conditions** for show/hide logic with all condition keys and operators.
- **Use dynamic data** with the right token syntax, modifiers, and provider prefixes.
- **Set up popups & templates** with the right type, assignment conditions, and frequency caps.

It also enforces the **four absolute rules** documented in `SKILL.md`:

1. Never edit the parent `bricks/` theme -- child only.
2. Custom elements must extend `\Bricks\Element` and register via `Elements::register_element()` on `init` priority 11.
3. Never invent control keys -- verify against the source first.
4. Responsive overrides use `:breakpoint_key` suffixes, not nesting.

## Novamira recipes (when paired)

Once Novamira is installed in your project, the skill exposes ready-made PHP recipes that Claude can run via Novamira's `Execute PHP` tool. A few examples (full library at [`references/novamira-verification.md`](references/novamira-verification.md)):

```php
// What controls does the heading element actually have?
return file_get_contents( get_template_directory() . '/includes/elements/heading.php' );

// What's actually saved on page 42?
return get_post_meta( 42, '_bricks_page_content_2', true );

// What global classes are defined?
return array_map( fn($c) => [ 'id'=>$c['id'], 'name'=>$c['name'] ],
                  get_option( 'bricks_global_classes', [] ) );

// What breakpoints are configured?
return \Bricks\Breakpoints::get_breakpoints();

// Which dynamic-data tags do my plugins add?
do_action( 'init' );
return apply_filters( 'bricks/dynamic_tags_list', [] );
```

The skill prefers these over static disk reads whenever Novamira is detected -- so it's always working against your actual install, not a frozen snapshot of mine.

## Compatibility

| Bricks | Skill version | Notes |
|---|---|---|
| 2.x | 1.0 | Tested. References include 2.0--2.3 features (components, slot, map-leaflet, toggle-mode). |
| 1.9.x | 1.0 | Most controls overlap; popups & query filters introduced in 1.9.6 documented as such. |
| < 1.9 | partial | Some keys won't exist; the skill annotates `@since` versions where it matters. |

If you're on a different Bricks version, regenerate the references against your install (see "Updating" below).

## Usage examples

### "Build a hero section"

Without the skill, Claude guesses. With the skill:

```
You: Build a Bricks hero with a centered heading, subtitle, and two buttons.
     Use a dark background and 80px vertical padding.

Claude: [reads SKILL.md, then references/containers-and-layout.md and
        references/elements-catalog.md, then writes valid JSON with
        _padding {top:80px,...}, _background.color, real element names]
```

### "Add a popup that opens once per session"

```
You: Make a newsletter popup that auto-opens 3 seconds after page load,
     but only once per browser session.

Claude: [reads references/popups-and-templates.md and
        references/interactions-and-animations.md, sets up:
        - bricks_template post with _bricks_template_type=popup
        - interaction with trigger:contentLoaded delay:3000
        - interactionConditions checking sessionStorage
        - close-button interaction that storageAdds the seen flag]
```

### "Register a counter custom element"

```
You: Add a custom element that counts up to a configurable number on scroll.

Claude: [reads references/custom-elements.md, writes
        bricks-child/elements/sl-counter.php extending \Bricks\Element,
        adds it to the registration array in functions.php, enqueues a
        small JS module that uses IntersectionObserver]
```

## Updating the skill for new Bricks versions

When Bricks ships a new version, run a regeneration pass:

1. Update Bricks on disk.
2. Open Claude Code in the project.
3. Prompt: `Regenerate the bricks-builder-skills against the current installed Bricks version. Diff against the existing references and update only what changed.`
4. Review the diffs, commit.

Or do it manually -- every reference cites file paths and line ranges. Re-grep, re-verify, re-write the affected section.

## Contributing

Contributions welcome. The reference files are plain Markdown -- no build step.

**Ground rules:**

- **Verify everything against Bricks source.** Cite `path:line` when you can. The whole point of this skill is no hallucination.
- **Keep `SKILL.md` lean.** It loads eagerly into every Claude session. Push detail into `references/`.
- **Avoid emojis.** Detail and density read better without them.
- **Don't paste Bricks source verbatim** beyond brief snippets -- it's licensed code.
- **Reference shapes, not screenshots.** This skill is for an LLM, not a human flipping through a tutorial.

PRs that improve accuracy beat PRs that add tone.

## Disclaimer

This is an unofficial community skill. Not affiliated with the Bricks Builder team. Bricks Builder is a commercial product -- you need a valid license to use the theme.

The skill itself contains **no Bricks source code** -- only documentation of public-facing shapes and conventions. Use of this skill does not bypass any licensing requirement of Bricks Builder.

## License

MIT -- see [`LICENSE`](LICENSE).

## Credits

Built by the Mekko Digital team while shipping Bricks-based marketing sites at scale. Thanks to:

- The Bricks Builder team -- for a builder whose data shapes are coherent enough to document this densely.
- The Novamira project -- for making live-WP introspection feasible from inside an AI agent.

---

If this skill saves your team hours of "wait, what's the actual key?" debugging, star the repo and tell us in [Discussions](https://github.com/Mekko-Digital/bricks-builder-skills/discussions). We'll keep it up to date with Bricks releases.
