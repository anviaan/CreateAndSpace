# Developer Guide

This guide explains how to maintain the modpack with **packwiz** from the
repository root.

## Requirements

- Git
- Go 1.24 or newer
- Java 17
- `packwiz` available on your `PATH`

Install packwiz:

```bash
go install github.com/packwiz/packwiz@latest
export PATH="$(go env GOPATH)/bin:$PATH"
packwiz --help
```

## Repository structure

- `pack.toml`: pack name, version, Minecraft, and Fabric versions.
- `mods/*.pw.toml`: metadata for each mod and its side (`client`, `server`, or
  `both`).
- `index.toml`: generated file index; do not edit it manually.
- `configureddefaults/`: files copied to the client profile when they do not
  already exist.
- `.packwizignore`: repository files that are excluded from the pack.

## Normal workflow

Run these commands from the repository root:

```bash
git checkout -b mod/name-of-change

# Add or update pack files.
packwiz refresh --build
packwiz list -s client
packwiz list -s server

git diff -- pack.toml index.toml mods/
git status
```

Always include `index.toml` with metadata changes.

## Adding a mod

### CurseForge

Use the project slug or, preferably, explicit IDs:

```bash
packwiz curseforge add <slug>
packwiz curseforge add --addon-id <project-id> --file-id <file-id>
```

Example:

```bash
packwiz curseforge add --addon-id 394468 --file-id 5485654
```

### Modrinth

```bash
packwiz modrinth add <slug>
packwiz modrinth add --project-id <project-id> --version-id <version-id>
```

Always choose a version compatible with Minecraft `1.20.1` and Fabric. Explicit
IDs prevent selecting the wrong result from a search.

### Set the mod side

After adding a mod, edit its `.pw.toml` file:

```toml
side = "client" # client only
side = "server" # server only
side = "both"   # client and server
```

Practical rule:

- UI, shaders, HUD, and visual optimizations: `client`.
- World content, blocks, entities, and game logic: `both`.
- Server administration-only mods: `server`.

Verify the result:

```bash
packwiz refresh --build
packwiz list -s server
```

## Updating or removing mods

```bash
# Update one mod.
packwiz update <name>

# Update all external files.
packwiz update --all

# Remove a mod interactively.
packwiz remove
```

Always review and test the changes before pushing:

```bash
packwiz refresh --build
git diff -- mods/ index.toml pack.toml
```

## Testing the pack locally

In terminal 1, from the repository root:

```bash
packwiz serve
```

The development server runs at `http://127.0.0.1:8080` and refreshes the index
automatically. To use another port:

```bash
packwiz serve -p 9000
```

In terminal 2, install only `server` and `both` mods into a test directory:

```bash
mkdir -p /tmp/createandspace-server
cd /tmp/createandspace-server
curl -fsSL -o packwiz-installer-bootstrap.jar \
  https://github.com/packwiz/packwiz-installer-bootstrap/releases/latest/download/packwiz-installer-bootstrap.jar
java -jar packwiz-installer-bootstrap.jar -g -s server \
  http://127.0.0.1:8080/pack.toml
```

If you selected another port, change `8080` in the URL.

## Exporting the client pack

Refresh the index before exporting:

```bash
packwiz refresh --build
packwiz modrinth export -o CreateAndSpace-3.4.mrpack
packwiz curseforge export -o CreateAndSpace-3.4-curseforge.zip
```

CurseForge exports can also be filtered explicitly:

```bash
packwiz curseforge export -s client -o CreateAndSpace-3.4-client.zip
```

Generated files are ignored by Git and should not be committed.

## Exporting the server pack

### Server-side mod ZIP

```bash
packwiz refresh --build
packwiz curseforge export -s server \
  -o CreateAndSpace-3.4-server-mods.zip
```

This is a side-filtered modpack export; it is not a runnable Fabric server.

### Runnable Fabric server

The GitHub Actions workflow creates the complete ZIP. To test the same process
locally:

```bash
curl -fsSL -o fabric-installer.jar \
  https://maven.fabricmc.net/net/fabricmc/fabric-installer/1.1.2/fabric-installer-1.1.2.jar
curl -fsSL -o packwiz-installer-bootstrap.jar \
  https://github.com/packwiz/packwiz-installer-bootstrap/releases/latest/download/packwiz-installer-bootstrap.jar

packwiz serve &
SERVE_PID=$!
trap 'kill "$SERVE_PID" 2>/dev/null || true' EXIT

mkdir -p server-pack
cd server-pack
java -jar ../fabric-installer.jar server \
  -mcversion 1.20.1 \
  -loader 0.16.9 \
  -downloadMinecraft
java -jar ../packwiz-installer-bootstrap.jar -g -s server \
  http://127.0.0.1:8080/pack.toml
```

Start the server with:

```bash
java -Xmx4G -jar fabric-server-launch.jar nogui
```

Accept the EULA before starting the server in production.

## Publishing a release

The `.github/workflows/release.yml` workflow runs for `v*` tags:

```bash
# First add a matching `## [version]` section to CHANGELOG.md.
git add .
git commit -m "Update modpack"
git push origin main

git tag v3.4-1.20.1
git push origin v3.4-1.20.1
```

The workflow extracts the matching `CHANGELOG.md` section and uses it as the
version description on Modrinth, CurseForge, and GitHub Releases. It fails if
the section for the pushed tag is missing.

It then builds and publishes the `.mrpack`, CurseForge ZIP, and server pack.
The workflow requires the variables and secrets documented in the main README.

## Troubleshooting

- **`packwiz: command not found`**: add `$(go env GOPATH)/bin` to `PATH`.
- **Port already in use**: run `packwiz serve -p 9000` and use the same port in
  the installer URL.
- **A mod appears on the server by mistake**: check that its `.pw.toml` has
  `side = "client"`, run `packwiz refresh --build`, and install again.
- **An export requests a manual download**: use a direct HTTPS URL in the
  `[download]` section, keep the file hash, and run the export again.
