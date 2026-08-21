# Magnet Search Skill

A small reusable AI-agent skill for explicitly triggered Torrent/Magnet search.

## Trigger

Use:

```text
mag <resource name>
```

Examples:

```text
mag ADN-081
mag SSIS-123
mag 某资源名称
```

Ordinary conversation, unprefixed resource names, standalone Magnet URLs, and `/magnet` do not trigger the skill.

## Design

The user-provided resource name is assumed accurate and is searched as-is. The skill uses a small personal-use source set and **site adapters** stored in `config/sources.yaml`.

Each adapter predefines:

- how to search the site
- where the matching Torrent result is found
- how to extract a Magnet URI or InfoHash
- the one-step fallback when the search result does not expose the Magnet

This prevents the agent from repeatedly spending tokens reasoning about how each website works.

## Source set

- Sukebei Nyaa — https://sukebei.nyaa.si/
- OneJAV — https://onejav.com/
- U9A9 — https://u9a9.com/
- BTSOW — https://btsow.live/
- Nyaa — https://nyaa.si/
- Bitsearch — https://bitsearch.eu/

The authoritative machine-readable registry is [`config/sources.yaml`](config/sources.yaml).

## Search behavior

1. Classify the query.
2. Pick the best 1-2 reachable specialist sources.
3. Follow their adapter instructions.
4. Validate a real Magnet URI or InfoHash.
5. Stop when enough exact results are found.
6. Otherwise move to the next source.
7. Deduplicate by InfoHash.

Do not crawl every source by default and do not invent site-specific scraping methods during a request.

## Magnet validation

A usable result must expose either:

- a complete `magnet:?xt=urn:btih:...` URI, or
- a clearly verifiable InfoHash attached to a Torrent result.

A movie page, subtitle page, ordinary search result, screenshot, or title-only page is not a Magnet result.

Never guess or synthesize a Magnet URI from incomplete data.

## Verification

The latest source verification pass is dated **2026-08-21**. `inconclusive` means the checker could not establish availability; it does not mean the site is confirmed dead.

Use only for resources the user is legally entitled to access. Respect copyright, local law, site terms, and access controls.
