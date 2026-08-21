# Magnet Search Skill

## Purpose

Use this skill when the user provides a resource name, title, catalog number, actor, author, or other identifying information and asks the agent to locate matching torrent/magnet resources.

The skill is designed as a **search-and-rank workflow**, not a single-site lookup. The agent should identify the resource type, generate useful search variants, select the most relevant search sources, verify source availability when needed, search in stages, deduplicate results by InfoHash, score matches, and return the strongest results.

> Use only for resources the user is legally entitled to access. Respect copyright, local law, site terms, and robots/access restrictions. Do not bypass authentication, paywalls, CAPTCHAs, DRM, or access controls.

## Core workflow

```text
User request
  -> identify resource type
  -> extract identifiers
  -> normalize title / identifier
  -> generate search variants
  -> select source group
  -> verify candidate source availability
  -> first-pass search
  -> evaluate result quality
  -> expand search only if necessary
  -> deduplicate by InfoHash
  -> rank results
  -> return Top N with confidence
```

Do not stop after one website returns no results. A failed search on one index is not evidence that the resource does not exist.

## Source availability is mandatory

The machine-readable source registry is `config/sources.yaml`. Each source has a `status` recorded by the latest verification pass.

Use these rules:

- `active`: safe to attempt, subject to normal request/access restrictions.
- `redirect`: use `canonical_url` when present; otherwise follow the redirect and record the final domain.
- `inconclusive`: the last checker could not establish availability. Do not describe the source as dead; perform a fresh check before relying on it.

Do not blindly trust an old status. If the user asks for current availability, or a source is important to the search, perform a fresh HTTPS check first.

If a configured source is unreachable, do not waste repeated attempts. Move to the next source and record the failure.

## Resource groups

### 1. JAV / Japanese adult video

When the input clearly contains a Japanese catalog number or JAV-specific metadata, prioritize sources tagged `jav` in `config/sources.yaml`.

Recommended order by configured priority and current availability:

1. Sukebei Nyaa
2. OneJAV
3. JAVJunkies
4. U9A9
5. BTSOW
6. General torrent search sources

For a catalog number, search the identifier first. Examples of identifier patterns include `IPX-123`, `SSIS-123`, `STARS-123`, `SIVR-123`, `MIDE-123`, `MIDV-123`, `JUQ-123`, `FSDSS-123`, `SONE-123`, `DASS-123`, `ADN-123`, and `ABW-123`.

Search order:

```text
catalog number
-> catalog number + actor
-> catalog number + title
-> original title
-> English title
-> actor + title
```

### 2. Chinese / Asian NSFW

Use sources tagged `asian` or `chinese`.

Recommended order:

1. U9A9
2. BTSOW
3. Sehuatang
4. SolidTorrents
5. Bitsearch
6. General torrent search sources

Try Chinese title, original-language title, English title, actor, actor + title, and title + year.

### 3. Adult anime / hentai / doujin / 2D

Use sources tagged `hentai`, `anime`, or `doujin`.

Recommended order:

1. Sukebei Nyaa
2. Nyaa
3. U9A9
4. BTSOW
5. SolidTorrents
6. Bitsearch

Prefer the original Japanese title, then English title, romanization, Chinese title, work number, author, or circle.

### 4. General torrent fallback

If specialist sources do not produce reliable matches, use the general sources tagged `general`:

1. SolidTorrents
2. Bitsearch
3. BT4G
4. 1337x
5. TorrentGalaxy
6. The Pirate Bay

The exact source list is configuration-driven; do not hard-code source URLs in the decision logic.

## Query normalization

Before searching, extract as many of these fields as possible:

- catalog number / work ID
- title
- original-language title
- English title
- Chinese title
- actor / performer
- author / creator
- circle / group
- year
- edition / release information

Normalize obvious punctuation and whitespace variants, but do not silently alter identifiers.

For Japanese catalog numbers, try both hyphenated and compact variants when appropriate, e.g. `SSIS-123` and `SSIS123`.

Do not rely on a Chinese translation when an original-language title or catalog number is available.

## Adaptive search strategy

Do not blindly query every source.

Start with the most likely specialist source that is currently reachable. If it returns enough high-confidence results, stop. If it returns no results or only weak matches, broaden the search.

Suggested thresholds:

- High confidence: exact identifier/title match and usable torrent metadata.
- Medium confidence: strong title/actor match but identifier is absent.
- Low confidence: only partial keyword overlap.

A practical staged strategy is:

```text
Stage 1: 1-2 specialist sources
Stage 2: 2-3 regional/general sources if Stage 1 is insufficient
Stage 3: broader general search if still insufficient
```

Do not claim that a resource is unavailable merely because all searched sites returned no result. Say which sources and query variants were checked.

## Result extraction

For every candidate, extract where available:

- title
- magnet URI
- InfoHash
- torrent URL
- source site
- size
- seeders
- leechers
- published/updated time
- file count
- filename/file list

Prefer metadata supplied by the source over assumptions derived from the title.

## Deduplication

Use InfoHash as the primary deduplication key.

If the same InfoHash appears on multiple sites, merge the sources into one result instead of displaying duplicates.

If InfoHash is unavailable, use a conservative normalized key based on title + size; do not incorrectly merge resources merely because titles are similar.

## Ranking

Score candidates using the following baseline:

| Signal | Score |
|---|---:|
| Exact catalog number match | +50 |
| Exact title match | +30 |
| Actor/creator match | +20 |
| Year match | +10 |
| Core keywords present in filename | +10 |
| Seeders > 20 | +10 |
| Seeders > 100 | +20 |
| Reasonable file size | +10 |
| Clearly unrelated result | -30 |
| Sample/trailer-only result | -30 |
| Suspected fake/advertising torrent | -50 |
| Clearly anomalous file structure | -30 |

The score is a ranking aid, not a guarantee of quality.

Prefer, in order:

1. content identity match
2. completeness / file structure
3. availability (seeders)
4. size plausibility
5. freshness

Do not rank a high-seed but weakly matching result above a low-seed exact match merely because of seed count.

## Output format

Default to the best 5 unique results.

For each result show:

```text
Title:
Size:
Seed / Leech:
Source:
Match confidence:
InfoHash:
Magnet:
```

If several sources point to the same InfoHash, show one result and list the sources together.

Clearly label uncertain results as `疑似匹配` rather than presenting them as exact matches.

If nothing reliable is found, report:

- resource type inferred
- query variants attempted
- source groups checked
- source availability status where relevant
- whether results were exact, partial, or absent

## Source maintenance

All source names, URLs, categories, priorities, status, canonical URLs, and notes belong in `config/sources.yaml`.

When a source changes domain or becomes unavailable, update the configuration rather than rewriting this skill.

Run a fresh source verification before changing a source from `active` to `inconclusive` or removing it. A checker failure is not by itself proof of a permanent outage.
