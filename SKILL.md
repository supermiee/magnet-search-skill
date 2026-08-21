# Magnet Search Skill

## Trigger

Activate this skill **only when the user message starts with `mag `** (the word `mag` followed by a space).

Examples:

```text
mag SSIS-123
mag ADN-081
mag 完整资源名称
```

Do not activate for ordinary conversation, an unprefixed resource name, a casual question, a standalone magnet/torrent URL, or `/magnet`.

Remove the leading `mag ` before searching. Treat everything after `mag ` as the user's resource name.

## Assumption

The resource name supplied by the user is assumed to be accurate.

Do not rewrite, correct, reinterpret, translate, or clean up the resource name before searching. Use the supplied query exactly as the primary search query.

Only allow a minimal technical variant when necessary for search compatibility, for example `ADN-081` → `ADN081`. Do not invent titles, actors, years, identifiers, or translations.

## Workflow

```text
mag <query>
  -> identify category
  -> choose the best 1-2 reachable sources
  -> search using the supplied query
  -> enough good results?
       yes -> stop
       no  -> use the next fallback source(s)
  -> validate Torrent/Magnet metadata
  -> deduplicate by InfoHash
  -> rank
  -> return best results
```

For personal use, keep the search narrow. Do not query every configured source unless the earlier sources are insufficient.

## Categories and source preference

The exact source list is maintained in `config/sources.yaml`.

### JAV / Japanese adult video

Prefer sources tagged `jav`, with specialist sources before general sources.

Search order should normally start with:

1. Sukebei Nyaa
2. OneJAV
3. U9A9
4. BTSOW

### Chinese / Asian NSFW

Prefer sources tagged `asian` or `chinese`.

Search order should normally start with:

1. U9A9
2. BTSOW
3. Sukebei Nyaa
4. Bitsearch

### Adult anime / doujin / 2D

Prefer sources tagged `hentai`, `anime`, or `doujin`.

Search order should normally start with:

1. Sukebei Nyaa
2. Nyaa
3. BTSOW
4. Bitsearch

### Fallback

Use the next configured source with a suitable category only when specialist searches are insufficient. Do not automatically crawl a long list of general torrent sites.

## Source availability

Use `config/sources.yaml` as the source registry.

- `active`: usable source.
- `redirect`: prefer `canonical_url`.
- `inconclusive`: availability was not established; re-check before relying on it.

If a source is unreachable, skip it and move on. Do not repeatedly retry one failed source.

## Result validation

A result counts as a **usable Magnet result** only when the source provides either:

- a complete Magnet URI containing an InfoHash, or
- a verifiable InfoHash associated with a Torrent result.

Do not treat movie/episode pages, subtitle pages, ordinary search-engine results, screenshots, preview pages, or title-only pages as Magnet results.

Never construct or guess a Magnet URI from incomplete information.

Metadata-only matches may be reported separately as `元数据线索`, but never as a usable Magnet.

## Ranking and deduplication

Prefer:

1. exact match to the user's supplied query
2. valid Torrent/Magnet metadata
3. reasonable file size/completeness
4. higher seed count
5. newer result

Use InfoHash as the primary deduplication key. If the same InfoHash appears on multiple sources, merge them into one result.

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

Use `精确匹配` for strong matches to the supplied query and `疑似匹配` for weaker matches.

If no usable Magnet is found, say so briefly and optionally report useful metadata-only clues separately. Do not claim that the resource does not exist solely because the configured sources returned no usable Magnet.

## Maintenance

Keep source names, URLs, categories, priorities, status, canonical URLs, and notes in `config/sources.yaml`.

Do not hard-code source URLs in this file.
