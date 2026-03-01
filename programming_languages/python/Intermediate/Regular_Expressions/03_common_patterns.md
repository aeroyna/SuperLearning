# Common Regex Patterns

## Email Validation (Simplified)
`^[\w\.-]+@[\w\.-]+\.\w+$`

## Date (YYYY-MM-DD)
`^\d{4}-\d{2}-\d{2}$`

## IP Address
`^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$`
(Note: Doesn't validate range 0-255)

## Greedy vs Non-Greedy
By default, quantifiers are greedy (match as much as possible). Add `?` to make them non-greedy/lazy.

```python
text = "<div>hello</div><div>world</div>"

# Greedy
re.findall(r'<div>.*</div>', text)
# ['<div>hello</div><div>world</div>'] (Matches everything from first to last div)

# Non-Greedy
re.findall(r'<div>.*?</div>', text)
# ['<div>hello</div>', '<div>world</div>']
```
