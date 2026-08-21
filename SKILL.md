# Magnet Search Skill

## Trigger

Activate this skill **only when the user message starts with `/magnet`**.

Examples:

```text
/magnet SSIS-123
/magnet ADN-081
/magnet 完整资源名称
```

Do not activate for ordinary conversation, an unprefixed resource name, a casual question, or a standalone magnet/torrent URL.

Remove `/magnet` before searching.

## Assumption

The resource name supplied by the user is assumed to be accurate.

**Do not rewrite, correct, reinterpret, translate, or "clean up" the resource name before searching.** Preserve the user's query exactly as the primary search query.

Only generate a minimal technical variant when necessary for search compatibility, such as:

```text
ADN-081 -> ADN081
```

Do not invent titles, actors, years, or identifiers that the user did not provide.

## Workflow

```text
/magnet <query>
  -> identify resource category
  -> search the best 1-2 configured sources using the supplied query
  -> if insufficient, search fallback sources
  -> validate actual Torrent/Magnet metadata
  -> deduplicate by InfoHash
  -> rank results
  -> return best matches
```

Do not search every source by default.

## Resource categories

### JAV / Japanese adult video

Preferred sources:

1. Sukebei Nyaa
2. OneJAV
3. JAVJunkies
4. U9A9
5. BTSOW

If the supplied query is a catalog number, search that exact identifier first.

### Chinese / Asian NSFW

Preferred sources:

1. U9A9
2. BTSOW
3. Sehuatang
4. Bitsearch
5. SolidTorrents

Use the supplied query as-is first.

### Adult anime / doujin / 2D

Preferred sources:

1. Sukebei Nyaa
2. Nyaa
3. U9A9
4. BTSOW
5. Bitsearch

Use the supplied query as-is first.

### General fallback

Only use when specialist sources are insufficient:

1. Bitsearch
2. BT4G
3. SolidTorrents
4. 1337x
5. TorrentGalaxy
6. The Pirate Bay

## Source availability

Use `config/sources.yaml` as the source registry.

- `active`: usable source; still subject to normal access restrictions.
- `redirect`: prefer `canonical_url` when present.
- `inconclusive`: availability was not established; re-check before relying on it.

If a source is unreachable, skip it. Do not repeatedly retry one failed source.

## Result validation

A result counts as a **usable Magnet result** only when the source provides either:

- a complete Magnet URI containing an InfoHash, or
- a verifiable InfoHash associated with a Torrent result.

Do **not** treat the following as Magnet results:

- movie/episode detail pages
- subtitle pages
- ordinary search-engine results
- screenshots or preview pages
- pages that only contain a title without Torrent metadata

Never construct or guess a Magnet URI from incomplete information.

Metadata-only matches may be reported separately as `元数据线索`, but must never be presented as a usable Magnet.

## Ranking and deduplication

Prefer, in order:

1. exact match to the user's supplied query
2. complete Torrent metadata
3. higher seed count
4. newer result

Use InfoHash as the primary deduplication key. If the same InfoHash appears on multiple sources, merge the sources into one result.

Do not let a high-seed but weakly matching result outrank a strong exact match merely because it has more seeds.

## Output

Return up to 5 unique usable Magnet results.

For each result show:

```text
Title:
Size:
Seed / Leech:
Source:
Match:
InfoHash:
Magnet:
```

Use `精确匹配` when the result matches the user's supplied resource name/identifier. Use `疑似匹配` only when the result differs from the supplied name but is otherwise plausibly related.

If no usable Magnet is found, say so briefly and optionally report useful metadata-only clues separately. Do not claim that the resource does not exist merely because the configured sources returned no usable Magnet.

## Maintenance

Keep source names, URLs, categories, priorities, status, canonical URLs, and notes in `config/sources.yaml`.

Do not hard-code source URLs in this file.
