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
2. Choose the best 1-2 specialist sources.
3. Search and evaluate the results.
4. Expand to fallback sources only when necessary.
5. Deduplicate by InfoHash.
6. Return up to 5 ranked results.

## Source groups

### JAV / Japanese adult video

- Sukebei Nyaa — https://sukebei.nyaa.si/
- OneJAV — https://onejav.com/
- JAVJunkies — https://javjunkies.com/
- U9A9 — https://u9a9.com/
- BTSOW — https://btsow.live/

### Chinese / Asian

- U9A9 — https://u9a9.com/
- BTSOW — https://btsow.live/
- Sehuatang — https://www.sehuatang.net/
- Bitsearch — https://bitsearch.eu/
- SolidTorrents — https://solidtorrents.to/

### Adult anime / doujin

- Sukebei Nyaa — https://sukebei.nyaa.si/
- Nyaa — https://nyaa.si/
- U9A9 — https://u9a9.com/
- BTSOW — https://btsow.live/
- Bitsearch — https://bitsearch.eu/

### General fallback

- Bitsearch — https://bitsearch.eu/
- BT4G — https://bt4gprx.com/
- SolidTorrents — https://solidtorrents.to/
- 1337x — https://1337x.to/
- TorrentGalaxy — https://torrentgalaxy.to/
- The Pirate Bay — https://thepiratebay.org/

## Source registry

The authoritative machine-readable registry is [`config/sources.yaml`](config/sources.yaml). It contains source priority, category, availability status, and canonical redirect information.

The latest registry verification pass is dated **2026-08-21**. `inconclusive` means availability could not be established by the checker; it does not mean the site is confirmed dead.

## Notes

Do not query every source by default. Prefer the most relevant reachable source, then broaden only when the results are insufficient.

Use only for resources the user is legally entitled to access. Respect copyright, local law, site terms, and access controls.
