# Rascal IntelliJ Debug Plugin -- releases

This repo hosts **built releases only** -- just `updatePlugins.xml` and the
release assets it points at. The plugin's source lives publicly in
[Periteleios/rascal-intellij-debugger-development](https://github.com/Periteleios/rascal-intellij-debugger-development),
under `tools/intellij/rascal-debugger-plugin/` -- see that project's
`tools/intellij/README.md` for what it does and how it's built, and for
its own manual/advanced setup path if you want highlighting/editing
without installing the plugin at all.

This repo exists because IntelliJ's custom plugin repository feature needs
a static `updatePlugins.xml` + release assets to auto-update from, not a
source checkout -- so releases are cut here even though the source is
public.

**Installing the plugin is all you need to do.** As of 0.1.0, it
auto-configures syntax highlighting, editing, and debugging on its own,
against whatever Maven-based Rascal project is currently open -- no
scripts to run, no manual Language Server or Run/Debug configuration to
set up.

## Installation

Settings/Preferences > Plugins > gear icon (⚙) > **Manage Plugin
Repositories...** > **+** > add:

```
https://raw.githubusercontent.com/Periteleios/rascal-intellij-debugger-releases/main/updatePlugins.xml
```

Apply, then find **Rascal Debugger** under Marketplace (it'll show up as
coming from this custom repository) and install it. Future updates pushed
here will show up as normal plugin updates -- no manual reinstall needed.
Open any `.rsc` file and syntax highlighting, diagnostics/CodeLenses, and
breakpoints/stepping should all just work.

## Cutting a new release (maintainers)

1. On GitHub, in *this* repo: **Releases > Draft a new release**.
   - Tag: `v<version>` (e.g. `v0.1.1`) -- must match exactly, the download
     URL below depends on it.
   - Attach the zip resulting from [your plugin build](https://github.com/Periteleios/rascal-intellij-debugger-development) a release asset -- drag it into the
     **"Attach binaries by dropping them here or selecting them"** box
     (below the description field), not into the description text itself.
     Dropping it into the description instead creates a `user-attachments`
     link, not a real release asset, and the download URL below won't
     resolve.
   - Publish.
2. Update `updatePlugins.xml` in this repo: bump the `version` attribute and
   the `url` attribute (new tag + new filename), commit, push to `main`.

That's it -- no separate "publish" step, `raw.githubusercontent.com` serves
the updated file immediately once it's on `main`.
