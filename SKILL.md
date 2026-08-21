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
  -> execute the source adapter exactly as configured
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

Each executable source adapter should explicitly define:

```yaml
adapter:
  request:
    method: GET | POST
    url: "https://..."
    query: {}
    headers: {}

  response:
    type: html | json

  results:
    item_selector: "..."
    fields:
      title: {selector: "...", type: text}
      detail_url: {selector: "...", type: url}

  magnet:
    strategy: direct | detail_page | json_field
    selector: "..."
    href_attribute: href
    infohash_regex: "..."

  pagination:
    enabled: false
    next_selector: null
    max_pages: 2

  fallback:
    on_empty_results: next_source
    on_parse_failure: next_source
    max_detail_pages: 3
```

### Adapter execution

1. Put the user's exact query into the configured request template.
2. Call the configured `method` and `url` with the configured query/body parameters.
3. Use the configured response parser (`html` or `json`).
4. Extract result items with the configured `item_selector`.
5. Extract `title` and `detail_url` using the configured field selectors.
6. Apply the configured Magnet strategy:
   - `direct`: extract Magnet directly from the result item.
   - `detail_page`: open the configured detail URL once, then extract Magnet.
   - `json_field`: read the configured JSON field.
7. Validate the Magnet/InfoHash.
8. Stop after the configured page/detail limits.
9. On an empty result or parse failure, follow `fallback` instead of inventing another endpoint.

If an adapter lacks a required request URL, parameter, selector, or extraction rule, mark the source `configuration_missing` and move to the next source.

Do not invent API paths, URL parameters, CSS selectors, JSON fields, or hidden endpoints at runtime when the adapter is present.

Use only normal, non-protected access. Do not bypass CAPTCHA, login, paywall, anti-bot protection, DRM, or other access controls.

## Generic extraction rule

When an adapter uses an `anchor` selector, inspect the configured result/detail DOM for links whose `href` starts with:

```text
magnet:?xt=urn:btih:
```

Extract the complete URI and parse the InfoHash from the `xt=urn:btih:` value.

When an adapter provides an InfoHash field, accept the explicitly exposed InfoHash as metadata. Do not fabricate a Magnet URI unless the source itself exposes one.

Never infer a Magnet from a title, filename, size, or unrelated webpage.

## Resource categories

### JAV / Japanese adult video

Prefer sources tagged `jav`.

### Chinese / Asian NSFW

Prefer sources tagged `asian` or `chinese`.

### Adult anime / doujin / 2D

Prefer sources tagged `hentai`, `anime`, or `doujin`.

Use the priorities and adapter definitions in `config/sources.yaml` rather than hard-coding site behavior here.

## Search breadth

For personal use, start with the single highest-priority reachable specialist source.

Only try the next source when:

- no usable results were returned;
- parsing failed;
- or all returned candidates are weak matches.

Do not query every source by default.

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

All site-specific URLs, priorities, availability status, and executable adapter instructions belong in `config/sources.yaml`.

The generic adapter field specification is documented in `config/adapter-schema.yaml`.

When a site's search flow or Magnet extraction changes, update its adapter in `config/sources.yaml` instead of adding ad-hoc instructions here.
