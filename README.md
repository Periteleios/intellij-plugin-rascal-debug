# Rascal IntelliJ Debug Plugin -- releases

This repo hosts **built releases only** <br>
The plugin's source lives publicly in [Periteleios/rascal-intellij-debugger-development](https://github.com/Periteleios/rascal-intellij-debugger-development),


## Installation

Settings/Preferences > Plugins > gear icon (⚙) > **Manage Plugin
Repositories...** > **+** > add:

```
https://raw.githubusercontent.com/Periteleios/rascal-intellij-debugger-releases/main/updatePlugins.xml
```

Apply, then find **Rascal Debugger** under Marketplace (it'll show up as
coming from this custom repository) and install it. <br>
Open any `.rsc` file and syntax highlighting, diagnostics/CodeLenses, and
breakpoints/stepping.


# Quick start

#### Prerequisite: 
- project SDK of Java 11+ `File > Project Structure > Project > SDK`
- build the project once so `target/classes` exists and the Maven jars are in your local `~/.m2`
repository:

    ```bash
    ./build.sh
    ```

#### Install the plugin:
- `Settings/Preferences > Plugins > gear icon (⚙) > Manage Plugin Repositories... > + >` <br>
   
    ```
    https://raw.githubusercontent.com/Periteleios/rascal-intellij-debugger-releases/main/updatePlugins.xml
    ```

- Go to the **Marketplace** tab and search "Rascal Debugger"
- Install it, restart when prompted.
- Open any `.rsc` file

### What gets auto-configured, and how

Three plugin classes, each registered via an IntelliJ or LSP4IJ extension
point are worth knowing about if something doesn't work, and you want to
know where to look in `idea.log` (`Help > Show Log in Files/Finder`):

- **`RascalTextMateBundleProvider`** registers the syntax-highlighting
  grammar (`rascal-textmate-bundle/`, shipped as a plugin resource,
  extracted once to a real directory on first use -- TextMate bundles need
  an actual filesystem path, not a jar resource).
  <br><br>
- **`RascalLanguageServerFactory`** registers the `.rsc` Language Server,
  computing its classpath from whichever project is currently open (the
  same computation "Import"/"Run in new Rascal terminal" already used).
  <br><br>
- **`RascalProjectActivity`** auto-creates the "Rascal Attach" DAP
  Run/Debug configuration on project open, including the exact
  `*.rsc -> rascal` Mappings-tab entry that's easy to miss by hand and,
  when missed, causes a completely silent breakpoint failure (no error, no
  gutter dot). Idempotent: if a DAP configuration already exists (of any
  name), it leaves it alone rather than creating a duplicate.

> Note (**Mac only**): `RascalLanguageServerFactory` doesn't inherit the
> Project SDK above.
> If `.rsc` files fail to load, and you see the message
> `JAVA_HOME environment variable is not defined correctly`, set
> `export JAVA_HOME=~/.jdks/<your-11+-JDK>` to `~/.zshenv`
> Restart IntelliJ 




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
