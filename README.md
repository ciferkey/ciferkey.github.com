# Running:

Based off [this guide](https://stesie.github.io/2016/08/nixos-github-pages-env)

$ nix-shell -p bundler
$ bundler package --no-install --path vendor
$ rm -rf .bundler vendor
$ exit  # leave nix-shell

$ $(nix-build '<nixpkgs>' -A bundix)/bin/bundix
$ rm result   # nix-build created this (linking to bundix build)

$ nix-shell
