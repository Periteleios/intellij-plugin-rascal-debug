# Rascal IntelliJ Debug Plugin -- releases

This repo hosts **built releases only**. The plugin's source lives publicly
in [Periteleios/rascal-intellij-debugger](https://github.com/Periteleios/rascal-intellij-debugger),
under `tools/intellij/rascal-debugger-plugin/` -- see that project's
`tools/intellij/README.md` for what it does and how it's built.

This repo exists because IntelliJ's custom plugin repository feature needs
a static `updatePlugins.xml` + release assets to auto-update from, not a
source checkout -- so releases are cut here even though the source is
public.

**This plugin alone does not give you Rascal syntax highlighting, error
diagnostics, or completion** -- those are two separate one-time setup
steps. Installing just this plugin without doing those first will look
broken (plain uncolored `.rsc` files, no CodeLenses). Full setup, in
order:

1. **Syntax highlighting** -- otherwise `.rsc` renders as plain,
   uncolored text. This repo hosts the actual TextMate bundle + setup
   script directly (mirrored from `rascal-intellij-debugger`'s
   `tools/intellij/rascal-textmate-bundle/`, kept in sync -- see "Keeping
   the mirrored highlighting/editing files in sync" below) -- it's
   generic Rascal grammar, not tied to any specific project, so **this one
   step needs nothing else**. Clone this repo (or download it
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

2. **Editing** (diagnostics, hover, completion, CodeLenses) -- also hosted
   directly in this repo (`tools/intellij/run-rsc-lsp.sh`,
   `compute-classpath.sh`), but unlike part 1's grammar, this script
   genuinely can't run standalone: it launches the actual language server
   jar against an actual Maven-built Rascal project, so **you still need
   one such checkout on disk somewhere** -- same requirement as part 3
   below, and just as generic: any Maven-based Rascal project works. This
   script just no longer has to physically live inside whichever one you
   point it at. Do that by setting `RASCAL_PROJECT_ROOT` to its absolute path
   (in the Language Server's **Environment variables** field in IntelliJ,
   not the Command field) -- the name is historical, from when this only
   ever worked against a project called `adept-base`; the variable itself
   works with any Maven-based Rascal project.
   Install [LSP4IJ](https://plugins.jetbrains.com/plugin/23257-lsp4ij)
   from the Marketplace, then wire up this script as a Language
   Server (Settings > Languages & Frameworks > Language Servers > **+**):
   ```bash
   chmod +x tools/intellij/run-rsc-lsp.sh
   ```
   | | `.rsc` server |
   |---|---|
   | Command | absolute path to `tools/intellij/run-rsc-lsp.sh` (in this checkout) |
   | Mappings | `*.rsc` |
   | Environment variables | `RASCAL_PROJECT_ROOT=/absolute/path/to/a/rascal/project` |

   Verified live: this script starts the real Rascal Language Server
   (2.22.4) and emits real JSON-RPC LSP messages when run this way, from
   this repo's own checkout, pointed at a separate project checkout.

3. **This plugin** + one LSP4IJ DAP run configuration (debugging) -- same
   deal as part 2, just built in rather than an env var: "Import"/"Run in
   new Rascal terminal" computes its classpath from whatever project is
   currently open in IntelliJ, via that project's own `pom.xml`. Works
   against any Maven-based Rascal project.

All three parts are documented in full in
[rascal-intellij-debugger](https://github.com/Periteleios/rascal-intellij-debugger)'s
own `tools/intellij/README.md`.

## Installing (one-time, per developer)

Settings/Preferences > Plugins > gear icon (⚙) > **Manage Plugin
Repositories...** > **+** > add:

```
https://raw.githubusercontent.com/Periteleios/rascal-intellij-debugger-releases/main/updatePlugins.xml
```

Apply, then find **Rascal Debugger** under Marketplace (it'll show up as
coming from this custom repository) and install it. Future updates pushed
here will show up as normal plugin updates -- no manual reinstall needed.

Then go do parts 1 and 2 above (both from this repo now) if you haven't
already -- this plugin alone won't make `.rsc` files look or behave
like source code.

## Cutting a new release (maintainers)

1. In [rascal-intellij-debugger](https://github.com/Periteleios/rascal-intellij-debugger),
   bump `version` in
   `tools/intellij/rascal-debugger-plugin/build.gradle.kts`, then build:
   ```bash
   cd tools/intellij/rascal-debugger-plugin
   export JAVA_HOME=~/.jdks/openjdk-26.0.2.1   # any JDK 17+
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
highlighting.sh`, `run-rsc-lsp.sh`, `compute-classpath.sh`) in this repo
are plain copies of the same content in
[rascal-intellij-debugger](https://github.com/Periteleios/rascal-intellij-debugger)
(at `tools/intellij/rascal-textmate-bundle/`, `tools/intellij/docs/
intellij_rascal_highlighting.md`, and `tools/intellij/{setup-intellij-
rascal-highlighting.sh,run-rsc-lsp.sh,compute-classpath.sh}` there), kept
at these paths on purpose (so none of their own path-resolution logic
needs changes here). The LSP launcher script only works this way because
it supports the `RASCAL_PROJECT_ROOT` override (see part 2 above) --
don't mirror an older copy that predates that, it won't run standalone.
Whenever any of these files change in `rascal-intellij-debugger` (a
grammar fix, a classpath change, a new `RASCAL_PROJECT_ROOT`-style
option), re-copy them here and push -- no version bump or release needed,
this isn't tied to the plugin's own versioning:

```bash
# from a rascal-intellij-debugger checkout, SOURCE=/path/to/rascal-intellij-debugger
cp -r "$SOURCE"/tools/intellij/rascal-textmate-bundle scripts/intellij-rascal-bundle
cp "$SOURCE"/tools/intellij/docs/intellij_rascal_highlighting.md docs/
cp "$SOURCE"/tools/intellij/{setup-intellij-rascal-highlighting.sh,run-rsc-lsp.sh,compute-classpath.sh} tools/intellij/
git add -A && git commit -m "Sync highlighting/editing files from rascal-intellij-debugger" && git push origin main
```
