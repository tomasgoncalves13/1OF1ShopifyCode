# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Shopify storefront theme for **1OF1 Fútbol** (1of1futbol.com). Forked from **Dawn 15.3.0** (`config/settings_schema.json` → `theme_name`/`theme_version`) with heavy custom branding layered on top. There is no build step, no package manager, no test runner — Shopify compiles Liquid server-side. "Running" the theme means pushing to a Shopify store and previewing there (typically via `shopify theme dev` from the Shopify CLI, run by the user).

## Working conventions specific to this theme

- **Fonts: use ONLY the families already self-hosted by the theme. Never import Google Fonts or any external font CDN.** The canonical list of `@font-face` family names available (declared in the theme's CSS — Poppins block + Montserrat block):

  **Poppins (primary):**
  - `'PoppinsExtraBold'`
  - `'PoppinsBold'`
  - `'PoppinsSemiBold'`
  - `'PoppinsMedium'`
  - `'PoppinsLight'`

  **Montserrat (secondary):**
  - `'MontserratExtraBold'`
  - `'MontserratBold'`
  - `'MontserratSemiBold'`
  - `'MontserratMedium'`
  - `'Montserratregular'` — note the lowercase `r` in "regular"; copy this string verbatim or it won't resolve.

  All names are one word, no spaces. Match the casing exactly. Poppins is the primary family; Montserrat is the secondary/accent. The theme's CSS applies `h2, h3 { font-family: 'PoppinsBold' !important }` globally, so when overriding headings inside a section you must repeat `!important` to win.

  **Mandatory: every new section MUST re-declare the `@font-face` block at the top of its `<style>` tag.** Inheritance/global declarations are unreliable across the editor preview and isolated section contexts — without re-declaring inside the section, the custom fonts fall back to the browser default. Paste this block verbatim at the top of the `<style>` in every section that uses any Poppins/Montserrat name:

  ```css
  @font-face { font-family: 'PoppinsExtraBold'; src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/poppins.extrabold.ttf?v=1747688794') format('truetype'); }
  @font-face { font-family: 'PoppinsBold';      src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/poppins.bold.ttf?v=1747688795') format('truetype'); }
  @font-face { font-family: 'PoppinsSemiBold';  src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/poppins.semibold.ttf?v=1747688794') format('truetype'); }
  @font-face { font-family: 'PoppinsMedium';    src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/poppins.medium.ttf?v=1747688794') format('truetype'); }
  @font-face { font-family: 'PoppinsLight';     src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/poppins.light.ttf?v=1748130416') format('truetype'); }
  @font-face { font-family: 'MontserratExtraBold'; src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/Montserrat-ExtraBold.ttf?v=1775402219') format('truetype'); font-display: swap; }
  @font-face { font-family: 'MontserratBold';      src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/Montserrat-Bold.ttf?v=1775402219') format('truetype'); font-display: swap; }
  @font-face { font-family: 'MontserratSemiBold';  src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/Montserrat-SemiBold.ttf?v=1775402091') format('truetype'); font-display: swap; }
  @font-face { font-family: 'MontserratMedium';    src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/Montserrat-Medium.ttf?v=1775402219') format('truetype'); font-display: swap; }
  @font-face { font-family: 'Montserratregular';   src: url('https://cdn.shopify.com/s/files/1/0907/8472/7374/files/Montserrat-Regular.ttf?v=1775402219') format('truetype'); font-display: swap; }
  ```

  Yes, this is duplicated across many sections — that is intentional. Do not try to "DRY" it into a snippet/include and skip pasting it.

- **Custom sections must use the `1of1-` filename prefix and `oneofone-` (or unique) CSS class prefix.** The codebase mixes Dawn's stock sections with bespoke ones — colliding class names with Dawn or with each other has been a recurring problem. Scope every new section's CSS under a section-id selector (e.g. `.my-section--{{ section.id }} ...`) so multiple instances on one page don't fight.
- **Brand naming:** the brand is **1OF1** / **1of1**. Do not introduce `dzr`, `dazur`, or other foreign brand strings even when porting markup from competitor sites — rewrite identifiers and copy.
- **User-facing copy is Portuguese (pt-PT).** Default `locales/en.default.json`, but customer-visible defaults in section schemas should be Portuguese.
- **Templates are JSON, not Liquid.** Page composition (which sections appear on the homepage, product pages, etc.) lives in `templates/*.json`. Editing those files in code is how you wire a new section into a page; the alternative is the Shopify admin theme editor, which auto-regenerates the same JSON. The header at the top of `templates/index.json` warns that the file is auto-managed — expect the admin to overwrite manual edits if both are touched.
- **Avoid heavy JS dependencies.** When porting components that use Slick/jQuery from other shops, reimplement with native CSS scroll-snap + small vanilla JS. The theme does not bundle Slick or jQuery globally.
- **No build artifacts.** Edit `.liquid`, `.css`, `.js` directly in `assets/`, `sections/`, `snippets/`, `layout/`. Shopify serves them as-is.

## Layout of the theme

- `layout/theme.liquid` — global HTML shell, head, JSON-LD, script includes. `theme.pagefly.liquid` is an alternate layout for pages built with the PageFly app.
- `sections/` — ~110 sections. Stock Dawn sections coexist with `1of1-*.liquid` custom sections; many other custom sections (e.g. `combo-sleeve.liquid`, `compra-mais-poupa-mais.liquid`, `feature-carousel.liquid`) don't carry the `1of1-` prefix because they predate the convention.
- `snippets/` — reusable Liquid partials (card-product, price, icons, etc.).
- `assets/` — all CSS/JS/images, served by Shopify CDN via `{{ 'file' | asset_url }}`. Dawn's component CSS files (`component-*.css`, `section-*.css`) live here alongside custom assets.
- `templates/` — page composition. `index.json` is the homepage; product templates are split per product (`product.airflow-pro.json`, `product.mini-shin-pads.json`, etc.) so each SKU can have a bespoke layout.
- `config/settings_schema.json` + `settings_data.json` — theme settings exposed in the admin Customize panel. `markets.json` controls multi-market behavior.
- `locales/` — translations for ~25 languages. `*.schema.json` files translate admin-facing setting labels; the plain `*.json` files translate storefront strings.

## How custom sections are typically structured

Read an existing one like `sections/1of1-trust-pills.liquid` or `sections/1of1-level-up-collections.liquid` before adding a new one. The pattern is:

1. Top-level `{%- liquid ... -%}` block computes a `uid` from `section.id` (replacing underscores).
2. Markup uses `oneofone-<name>__<element>` BEM-style classes and an outer wrapper with class `oneofone-<name>--{{ uid }}` for scoping.
3. `<style>` block scopes every rule under `.oneofone-<name>--{{ uid }}` and reads colors/spacing from `section.settings.*`.
4. `<script>` is wrapped in an IIFE and guards against double-init via `dataset.<name>Ready === '1'`.
5. `{% schema %}` ends the file with `settings`, `blocks`, and a `presets` entry so the section appears in the theme editor's "Add section" picker.

When wiring a section into the homepage, edit `templates/index.json`: add the section to the `sections` object and include its key in the `order` array.

## User instructions inherited from the global config

The user's global `~/.claude/CLAUDE.md` emphasizes: state assumptions before coding, prefer minimum code, make surgical edits (don't refactor adjacent code, don't delete pre-existing dead code without asking), and define verifiable success criteria for multi-step tasks. Those rules apply here too.
