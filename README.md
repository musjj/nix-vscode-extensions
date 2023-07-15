# Nix expressions for VS Code Extensions

At the time of writing this, searching `nixpkgs` yields around **200** `VS Code` extensions. However, the `VS Code Marketplace` contains more than **40,000** extensions!

This flake provides Nix expressions for the majority of available extensions from [Open VSX](https://open-vsx.org/) and [VS Code Marketplace](https://marketplace.visualstudio.com/vscode). A `GitHub Action` updates the extensions daily.

That said, you can now use a different set of extensions for `VS Code`/`VSCodium` in each of your projects. Moreover, you can share your flakes and cache them so that other people don't need to install these extensions manually!

## Note

- Extension names and publishers are lowercased.
- If an extension's publisher or name aren't valid Nix identifiers, you may access them by quoting the attribute names like `vscode-marketplace."4"."2"`.
- You may search the repo to find out at what commit a given extension was available.
- We have a permission from MS to use a crawler on their API in this case (see the [discussion](https://github.com/NixOS/nixpkgs/issues/208456)). Please, don't abuse this flake!

## Template

This repository provides a [template](templates/flake.nix).
This template provides a [VSCodium](https://github.com/VSCodium/vscodium) with a couple of extensions.
Try it:

```console
nix flake new vscodium-project -t github:nix-community/nix-vscode-extensions
cd vscodium-project
git init && git add .
nix develop
```

This will print extensions available in the `VSCodium`.

Run `codium .` or `nix run .#codium .` to start `VSCodium` in the current directory.

In case of problems see [Troubleshooting](#troubleshooting).

## Example

There's a sample package `vscodium-with-extensions` with a couple of extensions that you can try.

```console
nix run github:nix-community/nix-vscode-extensions#vscodium-with-extensions -- --list-extensions
```

## Usage

### Extensions

We provide extensions attrsets that contain both universal and platform-specific extensions.
We provide a [reasonable](https://github.com/nix-community/nix-vscode-extensions/issues/20) mapping between the sites target platforms and Nix-supported platforms.

There are several attrsets:

- `vscode-marketplace` and `open-vsx` contain the latest versions of extensions, including pre-release ones. Such pre-release versions expire in some time. That's why, there are `-release` attrsets.
- `vscode-marketplace-release` and `open-vsx-release` contain the release versions of extensions (see [Release extensions](#release-extensions)).
- `forVSCodeVersion "4.228.1"` allows to leave only the extensions [compatible](https://code.visualstudio.com/api/working-with-extensions/publishing-extension#visual-studio-code-compatibility) with the `"4.228.1"` version of `VS Code`.
  - You may supply the actual version of your `VS Code` instead of `"4.228.1"`.

### Overlay

The [flake.nix](./flake.nix) and `default.nix` provide an overlay that you can apply to `nixpkgs`.

### With flakes

See [Template](#template).

Add the following to your `flake.nix` (see [Flakes](https://nixos.wiki/wiki/Flakes)).

```nix
inputs.nix-vscode-extensions.url = "github:nix-community/nix-vscode-extensions";
```

### Without flakes

```nix
let 
  system = builtins.currentSystem;
  extensions =
    (import (builtins.fetchGit {
      url = "https://github.com/nix-community/nix-vscode-extensions";
      ref = "refs/heads/master";
      rev = "c43d9089df96cf8aca157762ed0e2ddca9fcd71e";
    })).extensions.${system};
  extensionsList = with extensions.vscode-marketplace; [
      rust-lang.rust-analyzer
  ];
in ...
```

## Explore

Explore extensions in `nix repl`.

Use your system instead of `x86_64-linux`.

Press the `Tab` button (denoted as `<TAB>` here) to see attrset attributes.

### Get the `extensions` attrset

#### Get with flakes

```console
$ nix repl

nix-repl> :lf github:nix-community/nix-vscode-extensions/c43d9089df96cf8aca157762ed0e2ddca9fcd71e
Added 10 variables.

nix-repl> t = extensions.<TAB>
extensions.aarch64-darwin  extensions.aarch64-linux   extensions.x86_64-darwin   extensions.x86_64-linux

nix-repl> t = extensions.x86_64-linux

nix-repl> t.<TAB>
t.forVSCodeVersion            t.open-vsx-release            t.vscode-marketplace-release
t.open-vsx                    t.vscode-marketplace
```

#### Get without flakes

```console
$ nix repl

nix-repl> t1 = (import (builtins.fetchGit {
                url = "https://github.com/nix-community/nix-vscode-extensions";
                ref = "refs/heads/master";
                rev = "c43d9089df96cf8aca157762ed0e2ddca9fcd71e";
              }))

nix-repl> t = t1.extensions.<TAB>
t1.extensions.aarch64-darwin  t1.extensions.aarch64-linux   t1.extensions.x86_64-darwin   t1.extensions.x86_64-linux

nix-repl> t = t1.extensions.x86_64-linux

nix-repl> t.<TAB>
t.forVSCodeVersion            t.open-vsx-release            t.vscode-marketplace-release
t.open-vsx                    t.vscode-marketplace
```

### Pre-release versions

```console
nix-repl> t.vscode-marketplace.rust-lang.rust-analyzer
«derivation /nix/store/jyzab0pdcgj4q9l73zsnyvc1k7qpb381-vscode-extension-rust-lang-rust-analyzer-0.4.1582.drv»
```

### Release versions

```console
nix-repl> t.vscode-marketplace-release.rust-lang.rust-analyzer
«derivation /nix/store/qjlr7iqgqrf2hd2z21xz96nmblxy680m-vscode-extension-rust-lang-rust-analyzer-0.3.1583.drv»
```

### Pre-release versions compatible with a given version of VS Code

```console
nix-repl> (t.forVSCodeVersion "1.78.2").vscode-marketplace.rust-lang.rust-analyzer
«derivation /nix/store/jyzab0pdcgj4q9l73zsnyvc1k7qpb381-vscode-extension-rust-lang-rust-analyzer-0.4.1582.drv»
```

## Contribute

### Issues

Resolve [issues](https://github.com/nix-community/nix-vscode-extensions/issues).

### README

- Fix links.
- Write new sections.
- Update commit SHA used in examples if they're too old.
- Enhance text.

### Release extensions

The [config](.github/config.yaml) contains several extensions.
We cache the information about the latest **release** versions of these extensions (see [Extensions](#extensions)).
If you'd like to use release versions of an extension, please, add that extension to the config and make a Pull request.

### Main flake

1. (Optionally) Install [direnv](https://direnv.net/), e.g., via `nix profile install nixpkgs#direnv`.

1. Run a devshell. When prompted about `extra-trusted-substituters` answer `y`. This is to use binary caches.

    ```console
    nix develop nix-dev/
    ```

1. (Optionally) Start `VSCodium` with necessary extensions and tools.

    ```console
    nix run nix-dev/#writeSettings
    nix run nix-dev/#codium .
    ```

### Haskell script

1. See the [README](./haskell/README.md).

1. Get the environment.

    ```console
    set -a
    source .env
    ```

1. Run the script.

    ```console
    nix run haskell/#updateExtensions
    ```

### Pull requests

Pull requests are welcome!

## Troubleshooting

- If `Nix`-provided `VSCodium` doesn't pick up the extensions:
  - Close other instance of `Nix`-provided `VSCodium`.
  - Try to reboot your computer and start `VSCodium` again.
- See [troubleshooting](https://github.com/deemp/flakes/blob/main/README/Troubleshooting.md).
