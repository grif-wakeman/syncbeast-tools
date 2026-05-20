# SyncBeast

Single-page shell that hosts the SyncBeast suite of audio tools. The shell
provides an app switcher and a global theme toggle; each individual tool is
loaded into an iframe and runs untouched.

## File layout

```
/index.html              the shell + switcher (the only "new" file)
/apps/aifbaby.html       the original tools, byte-identical copies
/apps/keybaby.html       (filenames cleaned up — no date stamps)
/apps/lufsbaby.html
/apps/mp3baby.html
/apps/savethatad.html
/apps/solbaby.html
/apps/tapbaby.html
/apps/trimbaby.html
/apps/wavbaby.html
```

No build step. No package.json. No dependencies. Just static HTML.

## URL patterns

- `https://www.syncbeast.tools/`              → opens the shell with the first app
- `https://www.syncbeast.tools/#/aifbaby`      → opens the shell on AIFbaby
- `https://www.syncbeast.tools/#/wavbaby`      → and so on for each app
- `https://www.syncbeast.tools/apps/aifbaby.html` → direct access to one tool, no shell

Browser back/forward works between apps because each switch pushes a history entry.

## Adding a new app

1. Drop the new HTML file into `/apps/` (any clean filename, e.g. `newbaby.html`).
2. Open `index.html`, find the `APPS` array near the top of the `<script>` block,
   and add one line:

   ```js
   { id: 'newbaby', name: 'NEWbaby', file: 'apps/newbaby.html' },
   ```

3. Commit and push. Done.

Order in the array = order in the switcher. The `id` is what shows up in the URL
(`#/newbaby`), so keep it lowercase and URL-safe.

If the new app has its own theme key (e.g. `newbaby-theme`), add that string to
the `PER_APP_THEME_KEYS` array right below `APPS` — this keeps theme in sync
between the shell and the new tool when it first loads. The shell still works
without this step; it just means the new app might briefly load in the wrong
theme on first visit.

## How theme sync works

- The shell stores its own theme under the key `syncbeast-theme` AND mirrors
  the same value into every per-app theme key (`aifbaby-theme`, `wavbaby-theme`,
  etc.) so iframes pick up the right theme on initial load with no flash.
- After an iframe loads, the shell injects a small `<style>` rule that hides
  the in-app `#themeToggle` so there aren't two of them. The shell's toggle
  controls everything.
- A `MutationObserver` on the iframe's `<html data-theme>` attribute syncs any
  in-iframe theme change back up to the shell, so things stay consistent even
  if some other in-app code path flips the theme.

## How code stays untouched

The files under `/apps/` are byte-identical copies of the originals. The shell
only modifies an iframe's DOM *at runtime* (to hide the in-app theme toggle and
to mirror the active theme). Nothing is rewritten on disk. Visiting a tool
directly at `/apps/xxx.html` gives you the original standalone experience with
its own theme toggle intact.

## Deploying to GitHub Pages with the custom domain

1. Push this folder to the repo root (or a `gh-pages` branch).
2. Repo Settings → Pages → set source to that branch / root.
3. Add `www.syncbeast.tools` as the custom domain in the Pages settings.
   GitHub will create / update the `CNAME` file in the repo for you.
4. At the DNS provider, add a `CNAME` record for `www` pointing to
   `<username>.github.io`.
5. Once Pages reports the domain as verified, enable "Enforce HTTPS".

No `.nojekyll` needed since there are no underscore-prefixed paths.
