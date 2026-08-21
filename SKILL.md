# Magnet Search Skill

## Trigger

Activate this skill **only when the user message starts with `/magnet`**.

Examples:

```text
/magnet SSIS-123
/magnet 某资源名称
/magnet "完整资源标题"
```

Do not activate for ordinary conversation, casual mentions of resource names, or a standalone magnet/torrent URL.

Remove `/magnet` before searching.

## Purpose

When triggered, identify the resource type, choose the most relevant configured sources, search in stages, validate that a result is actually a torrent/magnet result, deduplicate, and rank.

Use only for content the user is legally entitled to access. Do not bypass authentication, paywalls, CAPTCHAs, DRM, or access controls.

## Workflow

```text
/magnet <query>
  -> identify type
  -> normalize query
  -> choose 1-2 best sources
  -> search
  -> validate real torrent metadata
  -> enough good results?
       yes -> stop
       no  -> broaden to fallback sources
  -> deduplicate by InfoHash
  -> rank
  -> return Top 5
```

Do not search every source by default.

## Resource types

### JAV / Japanese adult video

Detect catalog-number patterns or clear JAV metadata.

Preferred sources:

1. Sukebei Nyaa
2. OneJAV
3. JAVJunkies
4. U9A9
5. BTSOW

For catalog numbers, try:

```text
exact number
-> normalized number
-> number + title/actor
```

### Chinese / Asian NSFW

Preferred sources:

1. U9A9
2. BTSOW
3. Sehuatang
4. Bitsearch
5. SolidTorrents

Try the supplied title first, then original-language or English variants when useful.

### Adult anime / doujin / 2D

Preferred sources:

1. Sukebei Nyaa
2. Nyaa
3. U9A9
4. BTSOW
5. Bitsearch

Prefer original Japanese or English titles when available.

### General fallback

Use only when specialist searches are insufficient:

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
- `inconclusive`: last check did not establish availability; re-check before relying on it.

If a source is unreachable, skip it. Do not repeatedly retry one failed source.

## Query handling

Extract when present:

- catalog number / work ID
- title
- original-language title
- English title
- Chinese title
- actor / creator
- year

Normalize whitespace and obvious punctuation variants, but never silently change identifiers.

For catalog numbers, try hyphenated and compact forms when appropriate.

## Torrent-result validation

This is mandatory. **Do not classify a normal webpage as a torrent/magnet result.**

A result is a **valid magnet candidate** only when the source exposes at least one of:

- a complete `magnet:` URI; or
- a verified InfoHash associated with a torrent/magnet record.

Prefer results that also expose torrent metadata such as:

- title
- size
- seeders/leechers
- file count or file list
- publication/update time

The following are **metadata only**, not magnet results:

- movie/episode detail pages
- subtitle pages
- reviews
- screenshots/image pages
- ordinary search-engine results
- pages that merely mention an InfoHash without an actual torrent/magnet record

If only metadata is found, report it as `元数据线索`, not as a downloadable magnet result.

Never invent or reconstruct a Magnet URI from incomplete information.

## Result ranking

Prefer:

1. exact identifier/title match
2. verified torrent/magnet metadata
3. matching actor/creator
4. complete and plausible file metadata
5. higher seed count
6. newer result

Use InfoHash as the primary deduplication key. If the same InfoHash appears on multiple sources, merge them into one result.

A weakly matching high-seed result must not outrank a strong exact match merely because it has more seeds.

## Output

Return up to 5 unique **verified magnet candidates**.

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

Use:

- `精确匹配` = strong identifier/title match
- `疑似匹配` = weaker match
- `元数据线索` = useful metadata found, but no verified magnet/torrent record

If no verified magnet candidate is found, say so clearly and briefly state which source groups were checked. Do not turn metadata-only pages into magnet results.

## Maintenance

Keep source names, URLs, categories, priorities, status, canonical URLs, and notes in `config/sources.yaml`.

Do not hard-code source URLs in this file.
