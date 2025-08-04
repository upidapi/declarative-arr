# Declarative *arr 

Nixos modules for the arr suite (currently sonarr, radarr, prowlarr). They all
work via sending api requests to mimic a user configuring them. Might change in
the future but atm it seams to work quite well.

This project is greatly inspired by [declarative-jellyfin](https://github.com/Sveske-Juice/declarative-jellyfin), especially this readme. 

> [!WARNING]
This is a young project, so beware of bugs. Backup your existing arr config.

# Features
* Indexers
* Indexer proxies
* Download clients
* Quality 
* Media naming
* And the glue to bring it all together

The defaults mimic what happens if you just click ok on everything during setup.

I think the options cover most common use cases. If anyone wants them, feel
free to contribute, or file an issue. They _should_ be quite simple to add. 

## Missing features
* UI Settings
* Notifications 
* Custom formats 
* "Metadata" tab
* Metadata sources


# Usage
Most options reside under `${serviceName}.guiSettings`

## Setup
Add the flake to your `inputs` and import the `nixosModule` in your configuration.

Example minimal flake.nix:
```nix
{
  description = "An example using declarative-arr flake";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs?ref=nixos-unstable";
    declarative-jellyfin.url = "github:upidapi/declarative-jellyfin";
    declarative-jellyfin.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = {
    nixpkgs,
    declarative-arr,
    ...
  }: {
    nixosConfigurations."your-hostname" = nixpkgs.lib.nixosSystem {
      modules = [
        declarative-arr.nixosModules.default # <- this imports the NixOS module that provides the options
        ./configuration.nix # <- your host entrypoint
      ];
    };
  };
}
```

## Examples
For a full setup see [my dotfiles](TOOD)

### Sonarr
```nix
services.sonarr = {
  enable = true;
  group = "media";
  # don't do this, you should use a secrets manager like agenix or sops-nix
  apiKeyFile = pkgs.writeText "your api key"; # 32 characters/numbers
  settings = {
    server = {
      port = 8989;
    };
  };
  guiSettings = {
    host = {
      username = "admin";
      # don't do this, you should use a secrets manager like agenix or sops-nix
      password = pkgs.writeText "your secret password"; 
    };

    # Where sonarr stores shows
    rootFolders = ["/media/tv"];

    # Setup connection to your download client
    downloadClients = {
      "qBittorrent" = {
        implementation = "QBittorrent";
        fields = {
          port = ports.qbit;
          host = "127.0.0.1";
          username = "admin";
          # don't do this, you should use a secrets manager like agenix or
          # sops-nix
          password = pkgs.writeText "password to qbittorrent";;
          sequentialOrder = true;
        };
      };
    };
    
    # configure the various quality bitrate, uses the default if not configured
    quality = {
      HDTV-1080p.bitrate = {
        min = 4;
        preferred = 50; # around 2.9GB / h
        max = 100;
      };
      WEBRip-1080p.bitrate = {
        min = 4;
        preferred = 50;
        max = 100;
      };
      WEBDL-1080p.bitrate = {
        min = 4;
        preferred = 50;
        max = 100;
      };
      Bluray-1080p.bitrate = {
        min = 4;
        preferred = 50;
        max = 100;
      };
      # Prevent 100GB 1080p movies
      "Bluray-1080p Remux".bitrate = {
        min = 4;
        preferred = 50;
        max = 150;
      };
    };
  };
```
