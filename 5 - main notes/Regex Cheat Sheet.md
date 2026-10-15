2026/09/28  -  12:05
status: 

Tags: [[js]] 


# REGEX CHEAT SHEET

**Flavor:** JS / PCRE-ish. Most patterns work across languages.

---

## ANCHORS

- `^` — start of string (or line with `m` flag)
- `$` — end of string (or line with `m` flag)
- `\b` — word boundary
- `\B` — not a word boundary
- `\A` — start of string (PCRE)
- `\z` — end of string (PCRE)
- `\Z` — end of string or before final newline (PCRE)

---

## CHARACTER CLASSES

- `.` — any char except newline (add `s` flag to include newline)
- `\d` — digit `[0-9]`
- `\D` — not a digit
- `\w` — word char `[A-Za-z0-9_]`
- `\W` — not a word char
- `\s` — whitespace `[ \t\n\r\f\v]`
- `\S` — not whitespace
- `[abc]` — any of a, b, c
- `[^abc]` — any char except a, b, c
- `[a-z]` — range
- `[0-9a-fA-F]` — hex digit
- `[\d\s]` — digit or whitespace
- `\p{L}` — Unicode letter (needs `u` flag)
- `\p{N}` — Unicode number (needs `u` flag)

---

## QUANTIFIERS

- `*` — 0 or more
- `+` — 1 or more
- `?` — 0 or 1
- `{n}` — exactly n
- `{n,}` — n or more
- `{n,m}` — between n and m
- `*?` — 0 or more, lazy
- `+?` — 1 or more, lazy
- `??` — 0 or 1, lazy
- `{n,m}?` — between n and m, lazy

---

## GROUPS & CAPTURES

- `(...)` — capture group
- `(?:...)` — non-capturing group
- `(?<name>...)` — named capture (JS, PCRE)
- `\1` — backreference to group 1
- `\k<name>` — backreference to named group
- `(?|...)` — branch reset (PCRE)

---

## ALTERNATION

- `|` — OR
- `(a|b)` — a or b

---

## LOOKAROUNDS

- `(?=...)` — positive lookahead
- `(?!...)` — negative lookahead
- `(?<=...)` — positive lookbehind
- `(?<!...)` — negative lookbehind

---

## FLAGS (JS)

- `g` — global — find all matches
- `i` — case-insensitive
- `m` — multiline — `^` and `$` match line breaks
- `s` — dotall — `.` matches newline
- `u` — unicode — full Unicode mode
- `y` — sticky — match from `lastIndex` only
- `d` — indices — match indices available

---

## ESCAPING

- `\.`
- `\*`
- `\+`
- `\?`
- `\(`
- `\)`
- `\[`
- `\]`
- `\{`
- `\}`
- `\|`
- `\^`
- `\$`
- `\\`
- `\/`

Escape any metacharacter with a backslash.

---

## COMMON RECIPES

### Email (simple)

```regex
^[^\s@]+@[^\s@]+\.[^\s@]+$
```

### URL (simplified)

```regex
https?:\/\/[^\s/$.?#].[^\s]*
```

### IPv4

```regex
^((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)\.){3}(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)$
```

### Date YYYY-MM-DD

```regex
^\d{4}-\d{2}-\d{2}$
```

### Time HH:MM:SS (24h)

```regex
^([01]\d|2[0-3]):[0-5]\d:[0-5]\d$
```

### US Phone

```regex
^\+?1?\s?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}$
```

### Hex color

```regex
^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$
```

### Slug

```regex
^[a-z0-9]+(?:-[a-z0-9]+)*$
```

### Username (3–16 alnum + underscore)

```regex
^[a-zA-Z0-9_]{3,16}$
```

### Strong password (8+, upper, lower, digit, special)

```regex
^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*]).{8,}$
```

### HTML tag (simple, with `s` flag)

```regex
<([a-z]+)([^>]*)>(.*?)<\/\1>
```

### Trim whitespace

```regex
^\s+|\s+$
```

### Duplicate words

```regex
\b(\w+)\s+\1\b
```

### Markdown link

```regex
\[([^\]]+)\]\(([^)]+)\)
```

### Number with thousands separators

```regex
\B(?=(\d{3})+(?!\d))
```

### Extract query param value

```regex
[?&]key=([^&]*)
```

### Remove JS line comments

```regex
\/\/.*$
```

### Remove JS block comments

```regex
\/\*[\s\S]*?\*\/
```

### CSV field (simple)

```regex
(?:^|,)(?:"([^"]*)"|([^,]*))
```

---

## JS USAGE

```javascript
const re = /pattern/flags;
re.test(str)                // boolean
str.match(re)               // matches
str.matchAll(re)            // iterator of all matches (g flag)
str.replace(re, '$1')       // replace with capture
str.split(re)               // split by pattern
re.exec(str)                // next match with lastIndex
```

---

## PERFORMANCE & GOTCHAS

- Avoid nested quantifiers like `(a+)+` — catastrophic backtracking.
- Anchor when possible: `^...$` cuts work.
- Prefer `[^"]*` over `.*?` for quoted strings.
- `.` does not match newline unless `s` flag.
- `\d` is ASCII in JS; use `\p{N}` with `u` for Unicode digits.
- Lookbehind not supported in all engines (Safari < 16.4 lacked it).
- Always escape user input when building dynamic regex.
- Test with real data; regex is easy to get 95% right and 5% wrong.

# References
