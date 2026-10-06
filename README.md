# RetroBoxDB PokemonMini

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Nintendo Pokémon Mini. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 101 source ZIPs, 8.9 MiB (No-Intro 50, RetroAchievements sets 51); 101 ROM files, 35.9 MiB uncompressed |
| Stored size | populated database 4.9 MiB; public Catalog 2.1 MiB (no ROM data) |
| Ratio | 55.4% of the source ZIPs, 13.8% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 256 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 32 MiB (32 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (46 files, each checked against the DAT hashes): 47.6 MiB/s, 10 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 0.074 s, TorrentZip 0.118 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.PokemonMini.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-PokemonMini/releases/latest/download/RetroBoxDB.PokemonMini.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-pokemini-games.csv) / [summary](reports/ra-pokemini.json), [build report](reports/pokemini-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

8 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-pokemini.json`): smallest 512 KiB / 32 MiB at 1.53 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 256 KiB / 32 MiB at 1.54 MiB. ZIPs 8.91 MiB, per-file LZMA 5.40 MiB.

- ROMs are 4–512 KiB; the whole collection deduplicates to 23 MiB of unique blocks and compresses to about 1.5 MiB in one group, where 256 KiB blocks need the least metadata within the rule.
- Header at 0x2100: `MN`, the `NINTENDO` string, a four-character game code (its last letter is the region), a 12-byte title and the `2P` flag, stored in `pokemini_hardware`.
- RetroAchievements console 24 (whole-file MD5). The RA folder holds 35 files that No-Intro does not list (homebrew and other dumps); they are stored with block deduplication.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 83 / 22 / 46 |
| DAT coverage per version | 20260529-125415: 46/46 |
| Local ROMs in no DAT | 37 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 16, RA only 35, hash not in the latest RA snapshot 0 ([list](reports/ra-pokemini-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-pokemini-missing.csv) |
| No-Intro DB Export + Dump Log unknown | 46 archives, 49 file identities, 20 documented hardware assertions; Dump Log Verified 9 |
| RetroAchievements (console 24) | 40 games with achievements: 39 with a local ROM (51 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 1 without a No-Intro counterpart |
| Chinese names | 43 of 44 rows translated (17 unique); 43 local ROMs have a Chinese name |
| Populated-database audit | 86 objects, 1 groups, 88 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.PokemonMini.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.PokemonMini.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.PokemonMini.sqlite --discover --ra --catalog RetroBoxDB.PokemonMini.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
