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
   bundle/`, kept in sync -- see "Keeping the mirrored highlighting/editing
   files in sync" below) -- it's
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

1. **Editing** (diagnostics, hover, completion, CodeLenses) -- also hosted
   directly in this repo (`tools/intellij/run-rsc-lsp.sh`, `run-ptl-lsp.sh`,
   `compute-classpath.sh`), but unlike part 0's grammar, these scripts
   genuinely can't run standalone: they launch the actual language server
   jar against an actual Maven-built project, so **you still need an
   `adept-base` checkout on disk somewhere** -- these scripts just no
   longer have to physically live inside it. Point them at yours by
   setting `ADEPT_BASE_ROOT` to its absolute path (in each Language
   Server's **Environment variables** field in IntelliJ, not the Command
   field). Install [LSP4IJ](https://plugins.jetbrains.com/plugin/23257-lsp4ij)
   from the Marketplace, then wire up these scripts as two Language
   Servers (Settings > Languages & Frameworks > Language Servers > **+**,
   once per file type):
   ```bash
   chmod +x tools/intellij/run-rsc-lsp.sh tools/intellij/run-ptl-lsp.sh
   ```
   | | `.rsc` server | `.ptl` server |
   |---|---|---|
   | Command | absolute path to `tools/intellij/run-rsc-lsp.sh` (in this checkout) | absolute path to `tools/intellij/run-ptl-lsp.sh` (in this checkout) |
   | Mappings | `*.rsc` | `*.ptl` |
   | Environment variables | `ADEPT_BASE_ROOT=/absolute/path/to/your/adept-base` | `ADEPT_BASE_ROOT=/absolute/path/to/your/adept-base` |

   Verified live: both scripts start the real Rascal Language Server
   (2.22.4) and emit real JSON-RPC LSP messages when run this way, from
   this repo's own checkout, pointed at a separate `adept-base` checkout.

2. **This plugin** + one LSP4IJ DAP run configuration (debugging) -- same
   deal as part 1: the plugin binary is what this repo hosts (below), but
   it still needs an `adept-base` checkout to drive a Rascal REPL/DAP
   session against that project's own compiled classes.

All three parts are documented in full, alongside the rest of the
`adept-base` IDE setup, in that project's own `tools/intellij/README.md`.

## Installing (one-time, per developer)

Settings/Preferences > Plugins > gear icon (⚙) > **Manage Plugin
Repositories...** > **+** > add:

```
https://raw.githubusercontent.com/Periteleios/intellij-plugin-rascal-debug/main/updatePlugins.xml
```

Apply, then find **Rascal Debugger** under Marketplace (it'll show up as
coming from this custom repository) and install it. Future updates pushed
here will show up as normal plugin updates -- no manual reinstall needed.

Then go do parts 0 and 1 above (both from this repo now) if you haven't
already -- this plugin alone won't make `.rsc`/`.ptl` files look or behave
like source code.

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

### Keeping the mirrored highlighting/editing files in sync

`scripts/intellij-rascal-bundle/`, `docs/intellij_rascal_highlighting.md`,
and everything under `tools/intellij/` (`setup-intellij-rascal-
highlighting.sh`, `run-rsc-lsp.sh`, `run-ptl-lsp.sh`,
`compute-classpath.sh`) in this repo are plain copies of the same paths in
`adept-base`, at identical relative paths on purpose (so none of their own
path-resolution logic needs changes here). The LSP launcher scripts only
work this way because they support the `ADEPT_BASE_ROOT` override (see
part 1 above) -- don't mirror an older copy that predates that, it won't
run standalone. Whenever any of these files change in `adept-base` (a
grammar fix, a classpath change, a new `ADEPT_BASE_ROOT`-style option),
re-copy them here and push -- no version bump or release needed, this
isn't tied to the plugin's own versioning:

```bash
# from an adept-base checkout, ADEPT_BASE=/path/to/adept-base
cp -r "$ADEPT_BASE"/scripts/intellij-rascal-bundle scripts/
cp "$ADEPT_BASE"/docs/intellij_rascal_highlighting.md docs/
cp "$ADEPT_BASE"/tools/intellij/{setup-intellij-rascal-highlighting.sh,run-rsc-lsp.sh,run-ptl-lsp.sh,compute-classpath.sh} tools/intellij/
git add -A && git commit -m "Sync highlighting/editing files from adept-base" && git push origin main
```
