# Magnet Search Skill

A reusable AI-agent skill for finding and ranking torrent/magnet resources from multiple search sources.

## What it does

- Identifies the resource category
- Extracts catalog numbers, titles, actors/authors and other identifiers
- Generates multilingual/search-friendly query variants
- Verifies source availability when freshness matters
- Uses specialist sources before broad fallback sources
- Expands the search only when necessary
- Deduplicates results by InfoHash
- Scores results by identity match, metadata quality and availability
- Returns a concise ranked result set with confidence labels

## Source verification

The source registry is [`config/sources.yaml`](config/sources.yaml).

A direct HTTPS verification pass was performed on **2026-08-21**. The registry records `active`, `redirect`, and `inconclusive` status for every source. `inconclusive` means the checker could not establish availability; it does **not** mean the site is confirmed dead.

Current checks established direct reachability for U9A9, OneJAV, BTSOW, Sehuatang and BT4G. Bitsearch's configured URL redirected to `https://bitsearch.eu/`. Several other domains were inconclusive with the checker and should be re-checked immediately before use.

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
- Bitsearch — https://bitsearch.eu/

### Anime / doujin

- Sukebei Nyaa — https://sukebei.nyaa.si/
- Nyaa — https://nyaa.si/
- U9A9 — https://u9a9.com/
- BTSOW — https://btsow.live/
- SolidTorrents — https://solidtorrents.to/

### General fallback

- SolidTorrents — https://solidtorrents.to/
- Bitsearch — https://bitsearch.eu/
- BT4G — https://bt4gprx.com/
- 1337x — https://1337x.to/
- TorrentGalaxy — https://torrentgalaxy.to/
- The Pirate Bay — https://thepiratebay.org/

## Agent integration

Load `SKILL.md` as the agent's search behavior. Keep site/domain data in `config/sources.yaml` so sources can be updated independently.

The agent should not blindly query every source. It should start with the most relevant currently reachable specialist source, evaluate the returned candidates, and broaden the search when confidence is insufficient.

## Legal and safety notice

This project is a generic torrent/magnet search workflow. Users are responsible for complying with copyright law, local regulations, site terms, and the rights associated with any content they access. Do not use the workflow to bypass authentication, paywalls, CAPTCHAs, DRM, or other access controls.

Source availability and domains can change. Re-check sources before relying on them.
