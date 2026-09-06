# Rascal IntelliJ Debug Plugin -- releases

This repo hosts **built releases only**. The plugin's source lives privately
in the `adept-base` repo, under `tools/intellij/rascal-terminal-plugin/` --
see that project's `tools/intellij/README.md` for what it does and how it's
built.

This repo exists purely so `adept-base` can stay private while the compiled
plugin is still installable/auto-updatable from a plain public URL, which is
what IntelliJ's custom plugin repository feature requires (it can't
authenticate against a private repo).

## Installing (one-time, per developer)

Settings/Preferences > Plugins > gear icon (⚙) > **Manage Plugin
Repositories...** > **+** > add:

```
https://raw.githubusercontent.com/Periteleios/intellij-plugin-rascal-debug/main/updatePlugins.xml
```

Apply, then find **Rascal Terminal Commands** under Marketplace (it'll show
up as coming from this custom repository) and install it. Future updates
pushed here will show up as normal plugin updates -- no manual reinstall
needed.

## Cutting a new release (maintainers)

1. In `adept-base`, bump `version` in
   `tools/intellij/rascal-terminal-plugin/build.gradle.kts`, then build:
   ```bash
   cd tools/intellij/rascal-terminal-plugin
   ./gradlew buildPlugin
   ```
   Output: `build/distributions/rascal-terminal-plugin-<version>.zip`.
2. On GitHub, in *this* repo: **Releases > Draft a new release**.
   - Tag: `v<version>` (e.g. `v0.1.1`) -- must match exactly, the download
     URL below depends on it.
   - Attach the zip from step 1 as a release asset.
   - Publish.
3. Update `updatePlugins.xml` in this repo: bump the `version` attribute and
   the `url` attribute (new tag + new filename), commit, push to `main`.

That's it -- no separate "publish" step, `raw.githubusercontent.com` serves
the updated file immediately once it's on `main`.
