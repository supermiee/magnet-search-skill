# Usage examples

## Example 1: exact catalog number

User:

```text
SSIS-123
```

Expected agent behavior:

1. Classify as JAV.
2. Search the exact catalog number first on specialist sources.
3. Try normalized variants such as `SSIS-123` and `SSIS123` when useful.
4. Prefer exact identifier matches over title-only matches.
5. Stop early if enough high-confidence results are found.

## Example 2: Chinese/Asian title

User:

```text
<resource title>
```

Expected behavior:

1. Classify as Chinese/Asian if the metadata indicates that category.
2. Try the supplied Chinese title, original-language title and English title.
3. Search Asian-oriented sources first.
4. Fall back to general torrent search if necessary.

## Example 3: adult anime

User:

```text
<anime title>
```

Expected behavior:

1. Classify as anime/doujin when metadata indicates it.
2. Prefer the original Japanese title and English title.
3. Search Nyaa/Sukebei-oriented sources first.
4. Deduplicate identical InfoHashes.

## Example 4: insufficient results

If the first source returns no useful candidates:

```text
specialist source
  -> second specialist source
  -> regional/general source
  -> broad fallback source
```

Do not report "not found" until the configured search stages have been exhausted or the user explicitly requests a limited search.

## Example 5: duplicate results

If three websites return the same InfoHash:

```text
Result A
  InfoHash: abc123...
  Sources: Sukebei, U9A9, Bitsearch
```

Show one result rather than three duplicate magnets.
