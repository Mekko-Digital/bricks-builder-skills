# Bricks Builder Skill for Claude Code

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) that gives Claude deep, verified knowledge of the [Bricks Builder](https://bricksbuilder.io) WordPress theme. With this skill installed, Claude can read and write Bricks page JSON, register custom elements, configure theme styles and breakpoints, build forms and query loops, wire interactions and popups -- all using **the exact control keys, value shapes, and naming conventions Bricks actually uses in the database**.

> **Why it exists.** LLMs hallucinate Bricks control keys (`_bgColor` instead of `_background`, `padding` instead of `_padding`, made-up animation names). This skill grounds every recommendation in the actual Bricks source, so you stop debugging settings that don't exist.

## What's inside

```
bricks-builder-skills/                    # repo root = a Claude Code plugin
+-- .claude-plugin/
|   +-- plugin.json                       # plugin manifest
|   +-- marketplace.json                  # marketplace entry (one-line install)
+-- skills/
    +-- bricks-builder-skills/
        +-- SKILL.md                      # entry point Claude loads automatically
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

## Setup: the skill is tool-agnostic

This skill does **not** require any extra tooling. It works with whatever access you already have to the WordPress install -- pick whichever of these you have, none is preferred over the others:

- **WP-CLI** -- if you have shell access, `wp eval` / `wp post meta get` / `wp option get` let Claude verify control keys, read saved page/template JSON, and inspect global options against the live install. The skill works fully this way.
- **Local files** -- if the theme is on disk, Claude reads `{template_dir}` / `{stylesheet_dir}` directly. Read-only, but enough for the skill to be fully useful.
- **[Novamira MCP](https://github.com/use-novamira/novamira)** (optional) -- gives Claude the same live access via PHP execution and file ops. It **can** work too, but it is **not recommended over WP-CLI or local files** -- it's just one more option. If you already run it, the skill picks it up automatically (no skill config needed); if you don't, **don't install it on this skill's account** -- WP-CLI or local files cover the same ground.

What live access (WP-CLI **or** Novamira) buys you over static disk reads:

- Verify every Bricks control key against the **live theme files** instead of trusting static docs.
- Read what's actually saved in the database for any page or template.
- Discover dynamic-data tags added by your installed plugins (ACF / Meta Box / JetEngine / Pods).

Without any of these the skill still works -- it falls back to the static references that ship in this repo.

## Install

You need [Claude Code](https://claude.com/claude-code) (or any Anthropic agent runtime that supports the [Skill format](https://docs.claude.com/en/docs/claude-code/skills)).

### Option A -- Claude Code plugin (recommended)

This repo is a Claude Code plugin. Inside Claude Code:

```
/plugin marketplace add Mekko-Digital/bricks-builder-skills
/plugin install bricks-builder-skills@mekko-digital
```

The skill is registered and loads on demand whenever you mention Bricks ("build a hero section", "register a custom element", "add a popup"). Update later with `/plugin marketplace update mekko-digital`.

### Option B -- Project-scoped skill (no plugin system)

Copy just the skill folder into your WordPress project (the directory you open Claude Code from). The skill itself lives at `skills/bricks-builder-skills/` inside the repo:

```bash
cd /path/to/your-wp-project
mkdir -p .claude/skills
git clone --depth 1 https://github.com/Mekko-Digital/bricks-builder-skills.git /tmp/bbs
cp -r /tmp/bbs/skills/bricks-builder-skills .claude/skills/bricks-builder-skills
```

The skill loads automatically the next time you start Claude Code in that directory.

### Option C -- User-scoped (available everywhere)

```bash
git clone --depth 1 https://github.com/Mekko-Digital/bricks-builder-skills.git /tmp/bbs
mkdir -p ~/.claude/skills
cp -r /tmp/bbs/skills/bricks-builder-skills ~/.claude/skills/bricks-builder-skills
```

The skill is now available in every Claude Code session.

### Option D -- Community `npx` installer (unofficial)

There is **no first-party `npx skills add`** command from Anthropic; skill/plugin distribution is via the plugin marketplace above. Community CLIs exist that wrap the same git-clone step, e.g.:

```bash
# Unofficial third-party tool -- verify the package before running.
npx skills add Mekko-Digital/bricks-builder-skills
```

These ultimately do the same thing as Option B/C. Prefer Option A unless you have a specific reason to use one.

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

## Live-verification recipes (WP-CLI or Novamira)

When Claude has live access -- via `wp eval` (WP-CLI) **or** Novamira's `Execute PHP` -- the skill exposes ready-made PHP recipes. The same snippets run either way: `wp eval '<php>'` or Novamira Execute PHP. A few examples (full library at [`references/novamira-verification.md`](references/novamira-verification.md)):

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

Whenever live access is available (WP-CLI or Novamira), the skill leans on these over static disk reads -- so it's working against your actual install, not a frozen snapshot. If only disk access is available, the static references still cover it.

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

Built by the **[Mekko Digital](https://mekkodigital.com)** team a small studio that ships fast Webflow & Fullstack MVP solutions. If this skill saves your team hours of "wait, what's the actual key?" debugging, star the repo and tell us in [Discussions](https://github.com/Mekko-Digital/bricks-builder-skills/discussions). We'll keep it up to date with Bricks releases.


Check out our other free tools at **[Mekko Tools](https://mekko.tools/)**.

**Tayyab Ul Islam**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/tayyabulislam16)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/tayyabulislam16)
[![X](https://img.shields.io/badge/X-000000?style=flat&logo=x&logoColor=white)](https://x.com/tayyabulislam16)
[![Facebook](https://img.shields.io/badge/Facebook-1877F2?style=flat&logo=facebook&logoColor=white)](https://www.facebook.com/tayyabulislam16)
[![Upwork](https://img.shields.io/badge/Upwork-6FDA44?style=flat&logo=upwork&logoColor=white)](https://www.upwork.com/freelancers/tayyabulislam16)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-25D366?style=flat&logo=whatsapp&logoColor=white)](https://wa.me/923060282032)

**Hamza Zafar**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/thehamzazafar)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/thehamzazafar)
[![X](https://img.shields.io/badge/X-000000?style=flat&logo=x&logoColor=white)](https://x.com/thehamzazafar)
[![Facebook](https://img.shields.io/badge/Facebook-1877F2?style=flat&logo=facebook&logoColor=white)](https://www.facebook.com/thehamzazafar)
[![Upwork](https://img.shields.io/badge/Upwork-6FDA44?style=flat&logo=upwork&logoColor=white)](https://www.upwork.com/freelancers/~01e0b93aab3dc4cc0d)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-25D366?style=flat&logo=whatsapp&logoColor=white)](https://wa.me/923148023751)
