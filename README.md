# Magnet Search Skill

A small reusable AI-agent skill for **explicitly triggered** torrent/magnet search.

## Trigger

The skill activates only when the message starts with:

```text
/magnet <resource name>
```

Examples:

```text
/magnet SSIS-123
/magnet 某资源名称
```

Ordinary conversation, ordinary resource names, and standalone magnet/torrent URLs must not activate the skill.

## Workflow

1. Identify the resource type.
2. Choose the best 1-2 reachable specialist sources.
3. Search using the user's query as-is.
4. Expand only when results are insufficient.
5. Validate actual Torrent/Magnet metadata.
6. Deduplicate by InfoHash.
7. Return up to 5 ranked results.

## Source set

The personal-use source registry is intentionally small:

- Sukebei Nyaa — https://sukebei.nyaa.si/
- OneJAV — https://onejav.com/
- U9A9 — https://u9a9.com/
- BTSOW — https://btsow.live/
- Nyaa — https://nyaa.si/
- Bitsearch — https://bitsearch.eu/

The authoritative machine-readable registry is [`config/sources.yaml`](config/sources.yaml). It contains source priority, category, availability status, and canonical redirect information.

## Design principles

- The user-provided resource name is assumed to be accurate.
- Do not rewrite or reinterpret the query.
- Keep source count small to reduce latency and maintenance.
- Do not query every source by default.
- A result is usable only when a complete Magnet URI or verifiable InfoHash is available.
- Web pages, subtitle pages, screenshots, and metadata-only pages are not Magnet results.

The latest source verification pass is dated **2026-08-21**. `inconclusive` means availability could not be established by the checker; it does not mean the site is confirmed dead.

Use only for resources the user is legally entitled to access. Respect copyright, local law, site terms, and access controls.
