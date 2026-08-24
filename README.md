# marvel-lcg-corpus

The frozen replay corpus for [marvel-lcg](https://github.com/mggarofalo/marvel-lcg):
the behavioral oracle the C# engine is validated against.

**Do not edit anything here.** This repository is content-addressed by a manifest
that lives in the main repo. A shard that changes is a shard that no longer
matches, and the check that notices is the whole point.

## What is here

58 gzipped shards, one per scenario, holding **1,773 replay scenes**.

Each shard maps a corpus-relative scene path to that scene's **exact text**, so
expanding a shard reproduces the bytes the manifest hashes. Storing the parsed
document instead costs ~0.3% of each scene's length to separator and key-order
differences, and every expanded scene then fails its hash.

| | |
|---|---|
| Root hash | `b4e3946150a02dc6f77b2c9513d3ed5eaff209827734163b40c905d2c6fa212d` |
| Engine | [`3446f61`](https://github.com/mggarofalo/marvel-lcg/commit/3446f61) |
| Manifest | `datasets/corpus/manifest.json` in the main repo |
| Frozen | 2026-08-24 |

## Using it

From a checkout of the main repo, with this repository cloned alongside:

```bash
cd py_src
# everything, verified scene by scene and by root hash
python -m tools.corpus.expand ../../marvel-lcg-corpus/ \
    --out ./corpus/ --manifest ../datasets/corpus/manifest.json

# or one scenario, which is why the shards are split this way
python -m tools.corpus.expand ../../marvel-lcg-corpus/ \
    --out ./corpus/ --manifest ../datasets/corpus/manifest.json --only rhino
```

`--manifest` is what makes it a verification rather than a decompression: every
scene is hashed as it lands, and the root hash over the whole set is recomputed
so that an omission cannot pass as success.

## Why the shards are here and the manifest is not

MARVEL-4. Git's problem is repeated churn on large files, not one-time storage,
so a dedicated repository of compressed, never-modified shards is fine at this
scale and avoids both LFS friction and object-storage infrastructure.

The manifest stays in the main repo so that **integrity is checkable without
fetching 579 MB**. It carries a hash per scene and a root hash over all of
them; this repository carries only the bytes those hashes describe.

## What is not here

**27 quarantined scenes.** Each reproduced every recorded step with every
digest matching and then made the engine log an error — so they are not
divergences, and they are not corpus. They are recorded by name in the
manifest under `quarantine`, and they are the C# engine's acceptance list:
cases the new engine should handle correctly *because* the Python oracle
could not.
