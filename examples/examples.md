# Usage examples

## Triggered

```text
/magnet SSIS-123
```

Expected behavior:

- Classify as JAV.
- Search the best 1-2 currently usable specialist sources first.
- Try the exact catalog number and a normalized variant.
- Stop when enough high-confidence results are found.

## Triggered: Chinese / Asian

```text
/magnet <resource title>
```

Expected behavior:

- Determine whether the metadata indicates Chinese/Asian content.
- Search Asian-oriented sources first.
- Broaden only if the first results are insufficient.

## Triggered: adult anime / doujin

```text
/magnet <anime title>
```

Expected behavior:

- Prefer original Japanese or English title.
- Search the specialist anime/NSFW sources first.
- Deduplicate identical InfoHashes.

## Not triggered

These must **not** activate the skill:

```text
SSIS-123
帮我看看这个资源是什么
magnet:?xt=urn:btih:...
今天天气怎么样
```

The user must explicitly use the `/magnet` prefix.

## Duplicate results

If multiple sources return the same InfoHash, show one result and merge the source names.
