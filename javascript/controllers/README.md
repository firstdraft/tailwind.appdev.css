# Stimulus controllers

`tailwind.appdev.css` styles several interactive components, but CSS alone
cannot open a dialog, toggle a menu, or remember a theme. These are the small
Stimulus controllers that supply the missing behavior.

**These are copy-paste examples, not a dependency.** Nothing here is loaded from
a CDN and nothing needs to be. Copy the file you want into your app:

```
javascript/controllers/modal_controller.js  →  app/javascript/controllers/modal_controller.js
```

The paths line up on purpose, and **the filename matters**: Stimulus derives the
identifier from it, so `modal_controller.js` is what makes `data-controller="modal"`
work. Rename the file and the markup below stops matching.

In a standard Rails app with importmap, that is the whole installation — the
generated `config/importmap.rb` has `pin_all_from "app/javascript/controllers"`,
so a new file in that directory registers itself with no further wiring.

This directory is the canonical copy. Fixes land here first, then get copied
out to the apps that use them.

## What each one is for

| Controller | Supplies behavior for | Styled by |
|---|---|---|
| `modal_controller.js` | Native `<dialog>` modals | the MODALS section |
| `dropdown_controller.js` | Dropdown menus | `.dropdown`, `.dropdown-menu` |
| `tabs_controller.js` | Tab panels | `.nav-underline`, `.tab-pane` |
| `dismiss_controller.js` | Dismissible flash alerts | `.alert-dismissible`, `.btn-close` |
| `dark_mode_controller.js` | The light/dark theme toggle | the `[data-theme="dark"]` block |

`modal_controller.js` and `dark_mode_controller.js` work with plain semantic
markup; the other three go with the named component classes.

## Markup contracts

Each controller expects a particular shape. Every file also carries this
information in its own header comment, so a student who opens the file finds
the usage without leaving their editor.

### Modal

```html
<div data-controller="modal">
  <button data-action="click->modal#open">Open</button>

  <dialog data-modal-target="dialog">
    <div class="card">
      <header>
        <button aria-label="Close" rel="prev" data-action="click->modal#close"></button>
        <h3>Title</h3>
      </header>

      <p>Content...</p>

      <footer>
        <button type="button" class="btn-outline" data-action="click->modal#close">Cancel</button>
        <button type="submit" data-action="click->modal#confirm">Confirm</button>
      </footer>
    </div>
  </dialog>
</div>
```

Closes on the X button, the Cancel button, the Escape key, and a backdrop click.
Backdrop closing can be turned off with `data-modal-close-on-backdrop-value="false"`.

The controller toggles `modal-is-open` / `-opening` / `-closing` on `<html>`;
the stylesheet uses those for scroll locking and the 200ms animations. If you
change one duration, change the other.

The card may be written either way — `<div class="card">` or `<article>` — since
the stylesheet styles both. The controller locates it with
`querySelector("article, .card")` and falls back to the dialog's first child.

### Dropdown

```html
<div class="dropdown" data-controller="dropdown">
  <button data-action="click->dropdown#toggle" aria-expanded="false">Menu</button>

  <ul class="dropdown-menu" data-dropdown-target="menu">
    <li><a href="#">An item</a></li>
  </ul>
</div>
```

No markup is needed to start it closed — `.dropdown-menu` is `display: none` and
the controller adds `show`. Also closes on an outside click or Escape.

### Tabs

```html
<div data-controller="tabs">
  <ul class="nav nav-underline" role="tablist">
    <li><button class="nav-link active" data-tabs-target="tab"
                data-action="click->tabs#select"
                aria-controls="posts-pane" aria-selected="true">Posts</button></li>
    <li><button class="nav-link" data-tabs-target="tab"
                data-action="click->tabs#select"
                aria-controls="likes-pane" aria-selected="false">Likes</button></li>
  </ul>

  <div id="posts-pane" class="tab-pane active" data-tabs-target="panel">...</div>
  <div id="likes-pane" class="tab-pane" data-tabs-target="panel">...</div>
</div>
```

Each button's `aria-controls` names the `id` of the panel it reveals — that
attribute is the wiring, not decoration. Mark the initially-visible tab and
panel `active`.

### Dismiss

```html
<div class="alert alert-success alert-dismissible" role="alert" data-controller="dismiss">
  Saved!
  <button type="button" class="btn-close" data-action="click->dismiss#remove" aria-label="Close"></button>
</div>
```

Removes the element outright. `alert-dismissible` reserves room on the right so
the close button does not overlap the text.

### Dark mode

```html
<button data-controller="dark-mode" data-action="click->dark-mode#toggle"></button>
```

Attach it to the toggle button itself. It stamps `data-theme` on `<html>`,
remembers the choice in `localStorage`, defaults to the operating system
preference, and writes its own label. Leave the button empty.

The stylesheet's own colors follow `data-theme` with no further setup. To make
Tailwind's `dark:` utilities follow the same switch — they key off
`prefers-color-scheme` by default — the page also needs:

```html
<style type="text/tailwindcss">
  @custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));
</style>
```

## A note on hiding

Dropdown menus and tab panels are hidden with **classes**, never the HTML
`hidden` attribute. Two reasons: it matches how Bootstrap behaved, and content
hidden by the attribute is genuinely absent from the page as far as request
specs are concerned. Specs that render without CSS or JavaScript can still find
and assert on a panel hidden by a class; one hidden by the attribute is
invisible to them.

The flip side is that such specs cannot see any of the behavior here at all.
Changes to these files need checking in a real browser.
