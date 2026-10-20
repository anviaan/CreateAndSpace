# Changelog

## [Unreleased] - 2026-10-20

### Changed

- Restored the nine platform-specific mod files and preserved their original CurseForge or Modrinth metadata.
- Kept exact-byte linking with the [`andre-carbajal/packwiz`](https://github.com/andre-carbajal/packwiz) fork v0.2.0 while enabling `--allow-overrides` for the CurseForge and Modrinth client exports.
- Updated release validation to confirm every client mod is present through a platform reference or an override.
- Added reproducible Fabric server-pack generation with the client-only mods excluded.
- Updated the player README and development documentation for the alternate Packwiz fork.

## [3.7] - 2026-09-26

### Added

- YUNG's Bridges, YUNG's Extras, YUNG's Better Mineshafts, YUNG's Better Caves, YUNG's Cave Biomes, Better Archeology, and Traveler's Titles.
- Required libraries: YUNG's API, TerraBlender, GeckoLib, and SuperMartijn642's Config Lib.

### Changed

- Disabled C2ME threaded world generation to avoid YUNG's Better Caves 1.20.1 compatibility issues.

## [3.6.1] - 2026-09-20

### Removed

- Absolute Order

### Fixed

- Added CurseForge metadata for Fabric API so the exported modpack includes it in manifest.json.

## [3.6] - 2026-09-20

### Added

- Absolute Order [#7](https://github.com/anviaan/CreateAndSpace/issues/7)

### Removed

- LambDynamicLights
- Create: Dynamic Lights
- Create Chunkloading
- Create Enchantment Industry Legacy

### Changed

- All mods have been updated to their latest versions.

### Fixed

- Farmers Delight mod has been updated to the last version [#5](https://github.com/anviaan/CreateAndSpace/issues/5), [#4](https://github.com/anviaan/CreateAndSpace/issues/4)
