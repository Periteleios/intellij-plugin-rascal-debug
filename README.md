# Rascal IntelliJ Debug Plugin -- releases

This repo hosts **built releases only**. The plugin's source lives privately
in the `adept-base` repo, under `tools/intellij/rascal-terminal-plugin/` --
see that project's `tools/intellij/README.md` for what it does and how it's
built.

This repo exists purely so `adept-base` can stay private while the compiled
plugin is still installable/auto-updatable from a plain public URL, which is
what IntelliJ's custom plugin repository feature requires (it can't
authenticate against a private repo).

**This plugin is only useful if you already have `adept-base` checked out**
-- it drives a Rascal REPL and DAP session against that project's own
compiled classes, and does nothing standalone. It also does **not** give
you Rascal syntax highlighting, error diagnostics, or completion by itself
-- those are two separate one-time setup steps. Installing just this plugin
without doing those first will look broken (plain uncolored `.rsc`/`.ptl`
files, no CodeLenses). Full setup, in order, is documented in
`adept-base`'s own `tools/intellij/README.md`:

0. Syntax highlighting (TextMate bundle install script).
1. LSP4IJ + the `.rsc`/`.ptl` language server launcher scripts (editing).
2. **This plugin** + one LSP4IJ DAP run configuration (debugging) -- the
   only piece this repo hosts.

## Installing (one-time, per developer)

Settings/Preferences > Plugins > gear icon (⚙) > **Manage Plugin
Repositories...** > **+** > add:

```
https://raw.githubusercontent.com/Periteleios/intellij-plugin-rascal-debug/main/updatePlugins.xml
```

Apply, then find **Rascal Debugger** under Marketplace (it'll show up as
coming from this custom repository) and install it. Future updates pushed
here will show up as normal plugin updates -- no manual reinstall needed.

Then go do parts 0 and 1 above from `adept-base`'s `tools/intellij/README.md`
if you haven't already -- this plugin alone won't make `.rsc`/`.ptl` files
look or behave like source code.

## Cutting a new release (maintainers)

1. In `adept-base`, bump `version` in
   `tools/intellij/rascal-terminal-plugin/build.gradle.kts`, then build:
   ```bash
   cd tools/intellij/rascal-terminal-plugin
   ./gradlew buildPlugin
   ```
   Output: `build/distributions/rascal-debugger-<version>.zip`.
2. On GitHub, in *this* repo: **Releases > Draft a new release**.
   - Tag: `v<version>` (e.g. `v0.1.1`) -- must match exactly, the download
     URL below depends on it.
   - Attach the zip from step 1 as a release asset -- drag it into the
     **"Attach binaries by dropping them here or selecting them"** box
     (below the description field), not into the description text itself.
     Dropping it into the description instead creates a `user-attachments`
     link, not a real release asset, and the download URL below won't
     resolve.
   - Publish.
3. Update `updatePlugins.xml` in this repo: bump the `version` attribute and
   the `url` attribute (new tag + new filename), commit, push to `main`.

That's it -- no separate "publish" step, `raw.githubusercontent.com` serves
the updated file immediately once it's on `main`.
