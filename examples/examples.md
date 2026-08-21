# Usage examples

## 1. Triggering

Only messages beginning with `/magnet` activate this skill.

```text
/magnet SSIS-123
```

These do not activate it:

```text
SSIS-123
帮我看看这个资源
magnet:?xt=urn:btih:...
```

## 2. Exact catalog number

```text
/magnet ADN-081
```

Expected behavior:

1. Classify as JAV.
2. Search the exact catalog number first.
3. Try a normalized identifier such as `ADN081` when useful.
4. Check specialist sources first.
5. Validate that returned items are actual torrent/magnet records.
6. Do not treat subtitle pages, detail pages, or search snippets as magnet results.

## 3. Metadata-only result

If a web result identifies the title or actor but does not expose a complete Magnet URI or verified InfoHash:

```text
状态: 元数据线索
```

Do not manufacture a Magnet URI from that page.

## 4. Duplicate results

If multiple sources return the same InfoHash:

```text
InfoHash: abc123...
Sources: Source A, Source B, Source C
```

Return one result, not three duplicates.

## 5. Insufficient results

```text
specialist source
  -> second specialist source
  -> regional/general source
  -> broad fallback source
```

Stop early when enough high-confidence, verified torrent results are available.
