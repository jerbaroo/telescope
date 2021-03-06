# Development Instructions
First install [Nix](https://nixos.org/download.html) the package manager, and
install [Cachix](https://docs.cachix.org/) to make use of our binary cache. Then
clone this repo (with submodules) and change in to the `telescope` directory.
You will also want to configure use of the Reflex-FRP cache, follow step 2
[here](https://github.com/obsidiansystems/obelisk#installing-obelisk). Finally
download pre-built binaries with Cachix, and perform an initial build:

``` bash
curl -L https://nixos.org/nix/install | sh
nix-env -iA cachix -f https://cachix.org/api/v1/install
git clone --recurse-submodules https://github.com/jerbaroo/telescope
cd telescope
./scripts/cachix/use.sh # Will take a long time the first time.
```

Development commands for the Telescope framework:

``` bash
# Type-check (with GHCID) the package passed as first argument.
./scripts/check.sh telescope
# Run all test suites with Cabal.
./scripts/test/suite.sh
# Build all packages with Nix (GHC & GHCJS) and run test suites.
./scripts/test/full.sh
# Start a Hoogle server (optional port argument, default=5000).
./scripts/hoogle.sh 5000
```

Development commands for the "testing" app:

``` bash
# Run the testing-backend server, server restarts on file change.
./scripts/run/dev.sh testing-backend
# Run the testing-frontend server, server restarts on file change.
./scripts/run/dev.sh testing-frontend
# Enter a REPL for interacting with the testing's database. TODO: FIX.
./scripts/repl.sh testing-backend
```

Production commands for the "testing" app:

``` bash 
# Build the testing-backend server.
./scripts/build/prod.sh testing-backend
# Generate the testing-frontend static files.
./scripts/build/prod.sh testing-frontend
# Run the testing-backend server.
./scripts/run/prod.sh testing-backend
```

