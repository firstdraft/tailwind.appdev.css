# tailwind.appdev.css

A customized Tailwind stylesheet for First Draft projects — the successor to [pico.firstdraft.css](https://github.com/firstdraft/pico.firstdraft.css), per [firstdraft/appdev#758](https://github.com/firstdraft/appdev/issues/758). It provides classless styling for semantic HTML (the Pico teaching phase) plus Bootstrap-feel named components (`.btn-primary`, `.card`, `.navbar`, `.alert`, …) for the later phase, all built on Tailwind's default design tokens.

## Usage

```html
<!-- The official Tailwind CSS browser build (Preflight + utility classes, compiled on the fly) -->
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4.3"></script>

<!-- Our classless styles + named Tailwind components -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/firstdraft/tailwind.appdev.css@latest/tailwind.appdev.css">
```

Then **nothing applies until you add the gating class to `<body>`**:

```html
<body class="appdev-styles">
```

— the direct replacement for Pico's `class="pico"`. Without it, pages get Tailwind's Preflight and utility classes only.

Instead of `@latest` you can pin a release, e.g. `@0.0.1` (the version number from `package.json`).

## How it's put together

Two source files in `src/`, one published artifact at the repo root:

- **`src/tailwind.appdev.css`** — the handwritten stylesheet. Everything lives in the `components` cascade layer, so utility classes students write always win over these defaults. Every value references a Tailwind theme variable (`var(--color-zinc-200)`, `calc(var(--spacing) * 4)`, …), never a literal, keeping the classless styles in lockstep with the utility classes.
- **`src/tailwind.theme.css`** — generated: Tailwind's full default theme as plain CSS custom properties. Required because the browser build tree-shakes its theme (it only emits variables that utility classes on the page actually use) and cannot load external files.
- **`tailwind.appdev.css`** (repo root) — the built artifact apps actually link: the source file with the theme baked in where its `@import` sits. One file, one request, variables available at first paint.

## How to update

1. Edit `src/tailwind.appdev.css` (never the root artifact — it is overwritten).
2. Run `bin/build` and commit both files.
3. Bump the SemVer version in `package.json`. Pushing that bump to `main` runs the `release.yml` workflow, which verifies the artifact is freshly built and creates the release tag — automatically making the new version available via [jsDelivr](https://www.jsdelivr.com/github).

### Tailwind version bumps

The baked-in theme is generated from the `@tailwindcss/cli` version pinned in `package.json`. To move to a newer Tailwind:

1. Update the pin in `package.json` (`devDependencies` → `@tailwindcss/cli`).
2. Run `bin/build --regenerate-theme`.
3. Update the `<script src=".../@tailwindcss/browser@X.Y">` pin in app layouts to the same minor version.

Keeping the script tag and the baked theme on the same minor is what prevents drift: a newer browser build could emit utilities referencing theme variables the frozen artifact doesn't define yet.

## Notes

- The gating class is named `appdev-styles` (not `prose`) so [`@tailwindcss/typography`](https://github.com/tailwindlabs/tailwindcss-typography) could be added later without conflict.
- Selectors are individually prefixed with `body.appdev-styles` rather than wrapped in CSS nesting — nesting desugars selector lists into `:is()`, which takes the max specificity of the list and would break overrides like `.btn-outline` vs the base `button` rule.
- Dark mode: the stylesheet's surfaces are tokenized, and a `[data-theme="dark"]` block flips the palette when something stamps `data-theme="dark"` on `<html>` (e.g. a Stimulus controller). Tailwind's own `dark:` variant keys off `prefers-color-scheme`; to make it follow the same toggle, a page must also declare `@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));` inside a `<style type="text/tailwindcss">` block.
- Interactive components (modals, dropdowns, tabs, dismissible alerts) are styled here but need small Stimulus controllers for behavior; see the experiment PRs on #758.
