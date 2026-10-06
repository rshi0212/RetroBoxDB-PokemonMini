PokemonMini Catalog, storage v4 (256 KiB blocks, 1 solid LZMA2 group of up to 32 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- Naming normalized: platform codes are the Batocera system names, every populated database is `RetroBoxDB.<label>.sqlite`, and `meta.scope` / `meta.storage` are derived from the platform and the current storage parameters.
- One schema for all fifteen platforms: the header tables of every platform (including Master System, 32X, WonderSwan, NeoGeo Pocket and Pokémon Mini) and the provider-information tables exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 101 ZIPs (nointro 50, retroachievements 51), 8.9 MiB (101 ROM files, 35.9 MiB uncompressed). Populated database: 4.9 MiB (55.4% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 83 ROM records, 22 games, 46 releases; DAT versions: 20260529-125415.
- RetroAchievements: 39 of 40 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 47.6 MiB/s (46 files); single file with a cold cache 0.074 s (ROM) / 0.118 s (TorrentZip) on average.
- Full audit of the populated database: 86 objects, 1 group, 88 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-PokemonMini/blob/main/README.zh-CN.md)
