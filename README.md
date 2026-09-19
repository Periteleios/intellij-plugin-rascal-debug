# Rascal IntelliJ Debug Plugin -- releases

This repo hosts **built releases only**. The plugin's source lives privately
in the `adept-base` repo, under `tools/intellij/rascal-terminal-plugin/` --
see that project's `tools/intellij/README.md` for what it does and how it's
built.

This repo exists purely so `adept-base` can stay private while the compiled
plugin is still installable/auto-updatable from a plain public URL, which is
what IntelliJ's custom plugin repository feature requires (it can't
authenticate against a private repo).

**This plugin alone does not give you Rascal syntax highlighting, error
diagnostics, or completion** -- those are two separate one-time setup
steps. Installing just this plugin without doing those first will look
broken (plain uncolored `.rsc`/`.ptl` files, no CodeLenses). Full setup, in
order:

0. **Syntax highlighting** -- otherwise `.rsc`/`.ptl` render as plain,
   uncolored text. This repo hosts the actual TextMate bundle + setup
   script directly (mirrored from `adept-base`'s `scripts/intellij-rascal-
   bundle/`, kept in sync -- see "Keeping the syntax-highlighting bundle in
   sync" below) -- it's
   generic Rascal grammar, not tied to any private codebase, so **this one
   step needs nothing from `adept-base`**. Clone this repo (or download it
   as a zip from the green "Code" button above), then from that checkout,
   one-time, idempotent (auto-detects your IntelliJ profile dir, or pass
   one explicitly):
   ```bash
   tools/intellij/setup-intellij-rascal-highlighting.sh
   ```
   Restart IntelliJ, then verify: Settings > Editor > TextMate Bundles
   should list `rascal-basic` enabled. See
   [docs/intellij_rascal_highlighting.md](docs/intellij_rascal_highlighting.md)
   for why the grammar needed patching.

1. **Editing** (diagnostics, hover, completion, CodeLenses) -- **unlike
   part 0, these scripts are NOT in this repo and can't be**: they resolve
   their own project root and require `adept-base`'s actual `pom.xml` +
   compiled `target/classes` to exist there, then shell out to `mvn
   dependency:build-classpath` against that real Maven project to launch
   the actual language server jar. They only mean something run in place
   inside a real `adept-base` checkout -- copying just the script text
   elsewhere gives you a script that errors out the instant it runs (no
   POM found). So: in your `adept-base` checkout (not this repo), install
   [LSP4IJ](https://plugins.jetbrains.com/plugin/23257-lsp4ij) from the
   Marketplace, then wire up `adept-base`'s own launcher scripts as two
   Language Servers (Settings > Languages & Frameworks > Language
   Servers > **+**, once per file type):
   ```bash
   # in your adept-base checkout
   chmod +x tools/intellij/run-rsc-lsp.sh tools/intellij/run-ptl-lsp.sh
   ```
   | | `.rsc` server | `.ptl` server |
   |---|---|---|
   | Command | absolute path to `<adept-base>/tools/intellij/run-rsc-lsp.sh` | absolute path to `<adept-base>/tools/intellij/run-ptl-lsp.sh` |
   | Mappings | `*.rsc` | `*.ptl` |

2. **This plugin** + one LSP4IJ DAP run configuration (debugging) -- also
   needs an `adept-base` checkout to actually do anything (it drives a
   Rascal REPL and DAP session against that project's own compiled
   classes), but the plugin binary itself is what this repo hosts (below).

Steps 1 and 2 are `adept-base`-specific and documented in full in that
project's own `tools/intellij/README.md`.

## Installing (one-time, per developer)

Settings/Preferences > Plugins > gear icon (⚙) > **Manage Plugin
Repositories...** > **+** > add:

```
https://raw.githubusercontent.com/Periteleios/intellij-plugin-rascal-debug/main/updatePlugins.xml
```

Apply, then find **Rascal Debugger** under Marketplace (it'll show up as
coming from this custom repository) and install it. Future updates pushed
here will show up as normal plugin updates -- no manual reinstall needed.

Then go do part 0 above (from this repo) and part 1 (from `adept-base`'s
`tools/intellij/README.md`) if you haven't already -- this plugin alone
won't make `.rsc`/`.ptl` files look or behave like source code.

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

### Keeping the syntax-highlighting bundle in sync

`scripts/intellij-rascal-bundle/`, `tools/intellij/setup-intellij-rascal-
highlighting.sh`, and `docs/intellij_rascal_highlighting.md` in this repo
are plain copies of the same paths in `adept-base` -- generic Rascal
grammar, not tied to any private code, so they're safe to mirror here
verbatim (identical relative paths on purpose, so the setup script's own
path resolution needs no changes). Whenever those files change in
`adept-base` (e.g. a grammar fix), re-copy them here and push -- no version
bump or release needed, this isn't tied to the plugin's own versioning:

```bash
# from an adept-base checkout, ADEPT_BASE=/path/to/adept-base
cp -r "$ADEPT_BASE"/scripts/intellij-rascal-bundle scripts/
cp "$ADEPT_BASE"/tools/intellij/setup-intellij-rascal-highlighting.sh tools/intellij/
cp "$ADEPT_BASE"/docs/intellij_rascal_highlighting.md docs/
git add -A && git commit -m "Sync syntax-highlighting bundle from adept-base" && git push origin main
```
