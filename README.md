# Magnet Search Skill

A reusable AI-agent skill for finding and ranking torrent/magnet resources from multiple search sources.

## What it does

- Identifies the resource category
- Extracts catalog numbers, titles, actors/authors and other identifiers
- Generates multilingual/search-friendly query variants
- Uses specialist sources before broad fallback sources
- Expands the search only when necessary
- Deduplicates results by InfoHash
- Scores results by identity match, metadata quality and availability
- Returns a concise ranked result set with confidence labels

## Supported source groups

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
- SolidTorrents — https://solidtorrents.to/
- Bitsearch — https://bitsearch.to/

### Anime / doujin

- Sukebei Nyaa — https://sukebei.nyaa.si/
- Nyaa — https://nyaa.si/
- U9A9 — https://u9a9.com/
- BTSOW — https://btsow.live/
- SolidTorrents — https://solidtorrents.to/

### General fallback

- SolidTorrents — https://solidtorrents.to/
- Bitsearch — https://bitsearch.to/
- BT4G — https://bt4gprx.com/
- 1337x — https://1337x.to/
- TorrentGalaxy — https://torrentgalaxy.to/
- The Pirate Bay — https://thepiratebay.org/

The authoritative machine-readable source registry is [`config/sources.yaml`](config/sources.yaml).

## Agent integration

Load `SKILL.md` as the agent's search behavior. Keep site/domain data in `config/sources.yaml` so sources can be updated independently.

The agent should not blindly query every source. It should start with the most relevant specialist source, evaluate the returned candidates, and broaden the search when confidence is insufficient.

## Legal and safety notice

This project is a generic torrent/magnet search workflow. Users are responsible for complying with copyright law, local regulations, site terms, and the rights associated with any content they access. Do not use the workflow to bypass authentication, paywalls, CAPTCHAs, DRM, or other access controls.

Source availability and domains can change. Verify a source before relying on it.
