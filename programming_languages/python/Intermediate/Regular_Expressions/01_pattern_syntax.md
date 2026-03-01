# Regex Syntax

Regular Expressions (regex) are strings defining search patterns.

## Metacharacters
Special characters with specific meanings:
`. ^ $ * + ? { } [ ] \ | ( )`

| Symbol | Matching |
|--------|----------|
| `.` | Any character (except newline) |
| `^` | Start of string |
| `$` | End of string |
| `*` | 0 or more repetitions |
| `+` | 1 or more repetitions |
| `?` | 0 or 1 repetition |
| `\d` | Digit [0-9] |
| `\w` | Word char [a-zA-Z0-9_] |
| `\s` | Whitespace |

## Quantifiers
*   `{n}`: Exactly n times
*   `{n,}`: At least n times
*   `{n,m}`: n to m times

## Groups
*   `(abc)`: Capture group
*   `(?:abc)`: Non-capturing group
*   `(?P<name>...)`: Named group
