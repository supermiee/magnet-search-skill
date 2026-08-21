# Magnet Search Skill

## Trigger

Activate only when the user message starts with `mag `.

```text
mag SSIS-123
mag ADN-081
mag 完整资源名称
```

Do not activate for normal conversation, an unprefixed resource name, a standalone Magnet URL, or `/magnet`.

Everything after `mag ` is the resource name. Assume it is accurate. Search it as-is. Do not rewrite, translate, correct, or invent metadata. A minimal technical variant such as `ADN-081` → `ADN081` is allowed only when needed by a source.

## Goal

Find real Torrent/Magnet results efficiently.

The source-specific extraction method is defined in `config/sources.yaml`. **Follow that adapter; do not invent a new scraping strategy during each request.**

## Deterministic workflow

```text
mag <query>
  -> classify resource
  -> select 1-2 best available sources
  -> use the source adapter
  -> extract candidate Magnet/InfoHash
  -> validate
  -> enough exact results?
       yes -> stop
       no  -> next source
  -> deduplicate by InfoHash
  -> rank
  -> return up to 5
```

Do not crawl all sources by default.

## Source adapter protocol

Each source adapter defines four things:

1. `search`: how to submit the user's exact query to the site's normal search.
2. `result`: where to find the matching Torrent result.
3. `magnet`: how to extract the Magnet/InfoHash.
4. `fallback`: what to do when no usable Magnet is exposed.

Use the site normally. Do not bypass CAPTCHA, login, paywall, anti-bot protection, DRM, or other access controls.

### Generic extraction rule

When an adapter says `anchor`, inspect the result/detail HTML/DOM for links whose `href` starts with:

```text
magnet:?xt=urn:btih:
```

Extract the complete URI. Parse the InfoHash from the `xt=urn:btih:` value.

When an adapter says `infohash`, accept a clearly identified InfoHash from the Torrent result and construct nothing unless the source explicitly exposes a Magnet URI. Never guess a Magnet from a title alone.

### Page navigation rule

If the search result itself does not contain a Magnet, open the matching Torrent/detail page once and apply the adapter's `magnet` rule there.

Do not recursively crawl unrelated pages.

## Resource categories

### JAV / Japanese adult video

Prefer sources tagged `jav`.

Order:

1. Sukebei Nyaa
2. OneJAV
3. U9A9
4. BTSOW

### Chinese / Asian NSFW

Prefer sources tagged `asian` or `chinese`.

Order:

1. U9A9
2. BTSOW
3. Sukebei Nyaa
4. Bitsearch

### Adult anime / doujin / 2D

Prefer sources tagged `hentai`, `anime`, or `doujin`.

Order:

1. Sukebei Nyaa
2. Nyaa
3. BTSOW
4. Bitsearch

## Source availability

Read `config/sources.yaml` before searching.

- `active`: use normally.
- `redirect`: use `canonical_url`.
- `inconclusive`: perform one fresh availability check before relying on it.

If the source fails, skip it and continue. Do not repeatedly retry the same source.

## Result validation

A usable result must contain:

- a complete Magnet URI with an InfoHash, or
- a clearly verifiable InfoHash attached to a Torrent result.

Do not treat movie pages, subtitle pages, ordinary web-search results, screenshots, preview pages, or title-only pages as Magnet results.

Never construct or guess a Magnet URI from incomplete data.

## Ranking and deduplication

Priority order:

1. exact match to the user's query
2. valid Torrent/Magnet metadata
3. plausible file size/completeness
4. higher seed count
5. newer result

Use InfoHash as the primary deduplication key. Merge duplicate InfoHashes from different sources.

## Output

Return up to 5 unique usable results.

```text
Title:
Size:
Seed / Leech:
Source:
Match:
InfoHash:
Magnet:
```

Use `精确匹配` or `疑似匹配` only; do not present metadata-only clues as usable Magnets.

If nothing usable is found, briefly state which source adapters were attempted and stop.

## Maintenance

All site-specific URLs, priorities, availability status, and adapter instructions belong in `config/sources.yaml`.

When a site's search flow or Magnet extraction changes, update its adapter there instead of adding ad-hoc instructions here.
