# Magnet Search Skill

## Trigger

This skill is **explicitly triggered only when the user message starts with `/magnet`**.

Examples:

```text
/magnet SSIS-123
/magnet 某资源名称
/magnet "完整资源标题"
```

Do **not** activate this skill for ordinary conversation, ordinary resource names, casual questions, or a standalone magnet/torrent URL.

Remove the `/magnet` prefix before searching.

## Purpose

When triggered, identify the resource type, search the most relevant configured sources, broaden the search only when needed, deduplicate results, and return the best matches.

Use only for resources the user is legally entitled to access. Do not bypass authentication, paywalls, CAPTCHAs, DRM, or other access controls.

## Simple workflow

```text
/magnet <query>
  -> identify type
  -> normalize query
  -> choose 1-2 best sources
  -> search
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

Look for catalog-number patterns or clear JAV metadata.

Preferred sources, in configured priority order:

1. Sukebei Nyaa
2. OneJAV
3. JAVJunkies
4. U9A9
5. BTSOW

For catalog numbers, search:

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

Prefer original Japanese or English titles over translated titles when available.

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

If a source is unreachable, skip it and continue. Do not repeatedly retry one failed source.

## Query handling

Extract, when present:

- catalog number / work ID
- title
- original-language title
- English title
- Chinese title
- actor / creator
- year

Normalize whitespace and obvious punctuation variants, but never silently change identifiers.

For catalog numbers, try both hyphenated and compact forms when appropriate.

## Result ranking

Prefer:

1. exact identifier/title match
2. matching actor/creator
3. complete and plausible file metadata
4. higher seed count
5. newer result

Use InfoHash as the primary deduplication key. If the same InfoHash appears on multiple sources, merge them into one result.

A weakly matching high-seed result should not outrank a strong exact match merely because it has more seeds.

## Output

Return up to 5 unique results.

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

Use `精确匹配` for strong identifier/title matches and `疑似匹配` for weaker matches.

If nothing reliable is found, briefly state which source groups were checked. Do not claim that the resource does not exist solely because the configured sources returned no result.

## Maintenance

Keep source names, URLs, categories, priorities, status, canonical URLs, and notes in `config/sources.yaml`.

Do not hard-code source URLs in this file.
