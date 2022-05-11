{:title "nix-shell - Use per-project host entries"
 :layout :post
 :tags ["linux" "nix"]}
 
...because who wants to clobber their `/etc/hosts` with project-specific entries.

### The problem

One of my projects is deployed as a docker container (well, actually most of them are).  
It connects to other containers, referring to them via docker-network names as host names,
set up in a project-wide configuration file.

Locally I use live hot reloading to develop, which is all very well, until I need to test
changes that include talking to other containers.  
To test such behaviour, there are three default approaches:

- Compile everything, rebuild the container and fire up all projects with `docker-compose`
  - Not well suited for quick iterations
- Modify the configuration file - requires a lot of discipline
  - When forgotten it requires to stop dev-mode, modify the config and re-start everything
  - Accidentally committing local changes may break production systems
  - Git-ignoring the config may break production systems when there really are changes to commit
- Modify the local machine's `/etc/hosts`
  - Project-specific settings permeate into the whole machine
  - Requires super user privileges
  - Magic solutions that are easily forgotten when on-boarding new team members
  
### My solution

I already use `shell.nix` files for all my projects, to keep development environments
self-contained and transferable, so why not use that.

Searching the web led to two solutions:

- Providing a `HOSTALIAS` env-parameter. This is deprecated and also only ever worked on a subset of applications.
- Using [userhosts](https://github.com/figiel/hosts), which has some quirks but works quite well

I integrated "userhosts" in my `shell.nix`, and the result went really well. It requires three steps:

- Provide a file in `hostfile` format
- Set the `LD_PRELOAD` variable to point to the `libuserhosts.so` file
- Point `HOSTS_FILE` to our hosts file

```language-nix
{ pkgs ? import <nixpkgs> { } }:

let
  myHostsFile = pkgs.writeText "hostsFile" ''
    0.0.0.0 docker-network-1
    0.0.0.0 some-other-docker-network
  '';
in {
  buildInputs = with pkgs; [ userhosts ];
  
  shellHook = ''
    export LD_PRELOAD=${pkgs.userhosts}/lib/libuserhosts.so
    export HOSTS_FILE=${myHostsFile}
  '';
}
```

Nb: 

I like exporting env variables in the `shellHook`, but one could also set them as attributes of the derivation.

This also helps with testing websites/-apps locally without dynamically rewriting all link targets.
