# User Guide

This is a more in-depth guide specific to GHCup. `ghcup --help` is your friend.

## Basic usage

For the simple interactive TUI (not available on windows), run:

```sh
ghcup tui
```

For the full functionality via cli:

```sh
# list available ghc/cabal versions
ghcup list

# install the recommended GHC version
ghcup install ghc

# install a specific GHC version
ghcup install ghc 8.2.2

# set the currently "active" GHC version
ghcup set ghc 8.4.4

# install cabal-install
ghcup install cabal

# update ghcup itself
ghcup upgrade
```

### Tags and shortcuts

GHCup has a number of tags and version shortcuts, that can be used as arguments to **install**/**set** etc.
All of the following are valid arguments to `ghcup install ghc`:

* `latest`, `recommended`
* `base-4.15.1.0`
* `9.0.2`, `9.0`, `9`

If the argument is omitted, the default is `recommended`.

## Manpages

For man pages to work you need [man-db](http://man-db.nongnu.org/) as your `man` provider, then issue `man ghc`. Manpages only work for the currently set ghc.
`MANPATH` may be required to be unset.

## Shell-completion

Shell completions are in [scripts/shell-completions](https://gitlab.haskell.org/haskell/ghcup-hs/-/tree/master/scripts/shell-completions) directory of this repository.

For bash: install `shell-completions/bash`
as e.g. `/etc/bash_completion.d/ghcup` (depending on distro)
and make sure your bashrc sources the startup script
(`/usr/share/bash-completion/bash_completion` on some distros).

## Portability

`ghcup` is very portable. There are a few exceptions though:

1. `ghcup tui` is only available on non-windows platforms
2. legacy subcommands `ghcup install` (without a tool identifier) and `ghcup install-cabal` may be removed in the future

# Configuration

A configuration file can be put in `~/.ghcup/config.yaml`. The default config file
explaining all possible configurations can be found in this repo: [config.yaml](https://gitlab.haskell.org/haskell/ghcup-hs/-/blob/master/data/config.yaml).

Partial configuration is fine. Command line options always override the config file settings.

## Env variables

This is the complete list of env variables that change GHCup behavior:

* `GHCUP_USE_XDG_DIRS`: see [XDG support](#xdg-support) above
* `GHCUP_INSTALL_BASE_PREFIX`: the base of ghcup (default: `$HOME`)
* `GHCUP_CURL_OPTS`: additional options that can be passed to curl
* `GHCUP_WGET_OPTS`: additional options that can be passed to wget
* `GHCUP_GPG_OPTS`: additional options that can be passed to gpg
* `GHCUP_SKIP_UPDATE_CHECK`: Skip the (possibly annoying) update check when you run a command
* `CC`/`LD` etc.: full environment is passed to the build system when compiling GHC via GHCup

### XDG support

To enable XDG style directories, set the environment variable `GHCUP_USE_XDG_DIRS` to anything.

Then you can control the locations via XDG environment variables as such:

* `XDG_DATA_HOME`: GHCs will be unpacked in `ghcup/ghc` subdir (default: `~/.local/share`)
* `XDG_CACHE_HOME`: logs and download files will be stored in `ghcup` subdir (default: `~/.cache`)
* `XDG_BIN_HOME`: binaries end up here (default: `~/.local/bin`)
* `XDG_CONFIG_HOME`: the config file is stored in `ghcup` subdir as `config.yaml` (default: `~/.config`)

**Note that `ghcup` makes some assumptions about structure of files in `XDG_BIN_HOME`. So if you have other tools
installing e.g. stack/cabal/ghc into it, this will likely clash. In that case consider disabling XDG support.**

## Caching

GHCup has a few caching mechanisms to avoid redownloads. All cached files end up in `~/.ghcup/cache` by default.

### Downloads cache

Downloaded tarballs (such as GHC, cabal, etc.) are not cached by default unless you pass `ghcup --cache` or set caching
in your [config](#configuration) via `ghcup config set cache true`.

### Metadata cache

The metadata files (also see [github.com/haskell/ghcup-metadata](https://github.com/haskell/ghcup-metadata))
have a 5 minutes cache per default depending on the last access time of the file. That means if you run
`ghcup list` 10 times in a row, only the first time will trigger a download attempt.

### Clearing the cache

If you experience problems, consider clearing the cache via `ghcup gc --cache`.

## Metadata

The metadata are the files that describe tool versions, where to download them etc. and
can be viewed here: [https://github.com/haskell/ghcup-metadata](https://github.com/haskell/ghcup-metadata)

### Mirrors

GHCup allows to use custom mirrors/download-info hosted by yourself or 3rd parties.

To use a mirror, set the following option in `~/.ghcup/config.yaml`:

```yml
url-source:
  # Accepts file/http/https scheme
  OwnSource: "https://some-url/ghcup-0.0.6.yaml"
```

See [config.yaml](https://gitlab.haskell.org/haskell/ghcup-hs/-/blob/master/data/config.yaml)
for more options.

Alternatively you can do it via a cli switch:

```sh
ghcup --url-source=https://some-url/ghcup-0.0.6.yaml list
```

#### Known mirrors

1. [https://mirror.sjtu.edu.cn/docs/ghcup](https://mirror.sjtu.edu.cn/docs/ghcup)

### (Pre-)Release channels

A release channel is basically just a metadata file location. You can add additional release
channels that complement the default one, such as the **prerelease channel** like so:

```sh
ghcup config add-release-channel https://raw.githubusercontent.com/haskell/ghcup-metadata/master/ghcup-prereleases-0.0.7.yaml
```

This will result in `~/.ghcup/config.yaml` to contain this record:

```yml
url-source:
  AddSource:
  - Right: https://raw.githubusercontent.com/haskell/ghcup-metadata/master/ghcup-prereleases-0.0.7.yaml
```

You can add as many channels as you like. They are combined under *Last*, so versions from the prerelease channel
here overwrite the default ones, if any.

To remove the channel, delete the entire `url-source` section or set it back to the default:

```yml
url-source:
  GHCupURL: []
```

If you want to combine your release channel with a mirror, you'd do it like so:

```yml
url-source:
  OwnSource:
  # base metadata
  - "https://mirror.sjtu.edu.cn/ghcup/yaml/ghcup/data/ghcup-0.0.6.yaml"
  # prerelease channel
  - "https://raw.githubusercontent.com/haskell/ghcup-metadata/master/ghcup-prereleases-0.0.7.yaml"
```

# More on installation

## Installing custom bindists

There are a couple of good use cases to install custom bindists:

1. manually built bindists (e.g. with patches)
    - example: `ghcup install ghc -u 'file:///home/mearwald/tmp/ghc-eff-patches/ghc-8.10.2-x86_64-deb10-linux.tar.xz' 8.10.2-eff`
2. GHC head CI bindists
    - example: `ghcup install ghc -u 'https://gitlab.haskell.org/api/v4/projects/1/jobs/artifacts/master/raw/ghc-x86_64-fedora27-linux.tar.xz?job=validate-x86_64-linux-fedora27' head`
3. DWARF bindists
    - example: `ghcup install ghc -u 'https://downloads.haskell.org/~ghc/8.10.2/ghc-8.10.2-x86_64-deb10-linux-dwarf.tar.xz' 8.10.2-dwarf`

Since the version parser is pretty lax, `8.10.2-eff` and `head` are both valid versions
and produce the binaries `ghc-8.10.2-eff` and `ghc-head` respectively.
GHCup always needs to know which version the bindist corresponds to (this is not automatically
detected).

## Compiling GHC from source

Compiling from source is supported for both source tarballs and arbitrary git refs. See `ghcup compile ghc --help`
for a list of all available options.

If you need to overwrite the existing `build.mk`, check the default files
in [data/build_mk](https://gitlab.haskell.org/haskell/ghcup-hs/-/tree/master/data/build_mk), copy them somewhere, adjust them and
pass `--config path/to/build.mk` to `ghcup compile ghc`.
Common `build.mk` options are explained [here](https://gitlab.haskell.org/ghc/ghc/-/wikis/building/using#build-configuration).

Make sure your system meets all the [prerequisites](https://gitlab.haskell.org/ghc/ghc/-/wikis/building/preparation).

### Cross support

ghcup can compile and install a cross GHC for any target. However, this
requires that the build host has a complete cross toolchain and various
libraries installed for the target platform.

Consult the GHC documentation on the [prerequisites](https://gitlab.haskell.org/ghc/ghc/-/wikis/building/cross-compiling#tools-to-install).
For distributions with non-standard locations of cross toolchain and
libraries, this may need some tweaking of `build.mk` or configure args.
See `ghcup compile ghc --help` for further information.

## Isolated installs

**Before using isolated installs, make sure to have at least GHCup version 0.1.17.8!**

Ghcup also enables you to install a tool (GHC, Cabal, HLS, Stack) at an isolated location of your choosing.
These installs, as the name suggests, are separate from your main installs and DO NOT conflict with them.


- No symlinks are made to these isolated installed tools, you'd have to manually point to them wherever you intend to use them.

- These installs, can also NOT be deleted from ghcup, you'd have to go and manually delete these.

You need to use the `--isolate` or `-i` flag followed by the directory path.

Examples:

1. install an isolated GHC version at location /home/user/isolated_dir/ghc/  
    - `ghcup install ghc 8.10.5 --isolate /home/user/isolated_dir/ghc`

2. isolated install Cabal at a location you desire  
    - `ghcup install cabal --isolate /home/username/my_isolated_dir/`

3. do an isolated install with a custom bindist  
    - `ghcup install ghc --isolate /home/username/my_isolated_dir/ -u 'https://gitlab.haskell.org/api/v4/projects/1/jobs/artifacts/master/raw/ghc-x86_64-fedora27-linux.tar.xz?job=validate-x86_64-linux-fedora27' head`

4. isolated install HLS  
    - `ghcup install hls --isolate /home/username/dir/hls/`

5. you can even compile ghc to an isolated location.  
    - `ghcup compile ghc -j 4 -v 9.0.1 -b 8.10.5 -i /home/username/my/dir/ghc` 

## Continuous integration

On windows, ghcup can be installed automatically on a CI runner non-interactively like so:

```ps
Set-ExecutionPolicy Bypass -Scope Process -Force;[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;Invoke-Command -ScriptBlock ([ScriptBlock]::Create((Invoke-WebRequest https://www.haskell.org/ghcup/sh/bootstrap-haskell.ps1 -UseBasicParsing))) -ArgumentList $false,$true,$true,$false,$false,$false,$false,"C:\"
```

On linux/darwin/freebsd, run the following on your runner:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | BOOTSTRAP_HASKELL_NONINTERACTIVE=1 BOOTSTRAP_HASKELL_MINIMAL=1 sh
```

This will just install `ghcup` and on windows additionally `msys2`.

For the full list of env variables and parameters to tweak the script behavior, see:

* [bootstrap-haskell for linux/darwin/freebsd](https://gitlab.haskell.org/haskell/ghcup-hs/-/blob/master/scripts/bootstrap/bootstrap-haskell#L7)
* [bootstrap-haskell.ps1 for windows](https://gitlab.haskell.org/haskell/ghcup-hs/-/blob/master/scripts/bootstrap/bootstrap-haskell.ps1#L17)

### github workflows

On github workflows you can use [https://github.com/haskell/actions/](https://github.com/haskell/actions/).
GHCup itself is also pre-installed on all platforms, but may use non-standard install locations.

## GPG verification

GHCup supports verifying the GPG signature of the metadata file. The metadata file then contains SHA256 hashes of all downloads, so
this is cryptographically secure.

First, obtain the gpg keys:

```sh
gpg --batch --keyserver keys.openpgp.org     --recv-keys 7784930957807690A66EBDBE3786C5262ECB4A3F
gpg --batch --keyserver keyserver.ubuntu.com --recv-keys FE5AB6C91FEA597C3B31180B73EDE9E8CFBAEF01
```

Then verify the gpg key in one of these ways:

1. find out where I live and visit me to do offline key signing
2. figure out my mobile phone number and call me to verify the fingerprint
3. more boring: contact me on Libera IRC (`maerwald`) and verify the fingerprint

Once you've verified the key, you have to figure out if you trust me.

If you trust me, then you can configure gpg in `~/.ghcup/config.yaml`:

```yml
gpg-setting: GPGLax # GPGStrict | GPGLax | GPGNone
```

In `GPGStrict` mode, ghcup will fail if verification fails. In `GPGLax` mode it will just print a warning.
You can also pass the mode via `ghcup --gpg <strict|lax|none>`.
   
# Tips and tricks

## ghcup run

If you don't want to explicitly switch the active GHC all the time and are using
tools that rely on the plain `ghc` binary, GHCup provides an easy way to execute
commands with a certain toolchain prepended to PATH, e.g.:

```sh
ghcup run --ghc 8.10.7 --cabal latest --hls latest --stack latest --install -- code Setup.hs
```

This will execute vscode with GHC set to 8.10.7 and all other tools to their latest version.
