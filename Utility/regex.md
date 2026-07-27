# Regex
 
Regular expressions — a pattern language for searching and matching text. Used everywhere in security work: grep, log analysis, Python scripts, SIEM detection rules, iptables extensions, sed/awk pipelines, nmap NSE scripts.
 
---
 
## Why it matters here
 
- **Log analysis** — pulling IPs, usernames, error patterns out of massive log files
- **Extracting data** — IPs, emails, hashes, URLs, tokens from mixed text
- **Detection rules** — writing patterns for SIEMs and IDS to catch malicious activity
- **Input validation / bypass** — understanding what regexes accept and reject is core to auth bypass, SQLi, XSS
- **Tool output parsing** — nmap, masscan, ffuf, hashcat all produce text you'll want to filter
---
 
## The two-second version
 
- `.` — any single character
- `*` — repeat the previous thing zero or more times
- `.*` — anything, including empty
- `+` — one or more of the previous
- `?` — zero or one (optional)
- `^` — start of line
- `$` — end of line
- `[]` — character class (any one of)
- `()` — group
- `|` — OR
Everything below is elaboration on these building blocks.
 
---
 
## Character shortcuts
 
| Symbol | Meaning | Example | Matches |
|---|---|---|---|
| `.` | Any single character | `c.t` | cat, cot, cut, c9t |
| `\d` | Any digit (0-9) | `\d\d\d` | 123, 456, 789 |
| `\w` | Any word char (letters, digits, `_`) | `\w\w` | ab, a1, _x |
| `\s` | Any whitespace (space, tab, newline) | `hello\sworld` | "hello world" |
| `\D` | NOT a digit | `\D\D` | ab, !@ |
| `\W` | NOT a word character | `\W` | !, @, space |
| `\S` | NOT whitespace | `\S+` | any unbroken word |
| `\t` | Tab |
| `\n` | Newline |
| `\r` | Carriage return |
 
---
 
## Quantifiers
 
| Symbol | Meaning | Example | Matches |
|---|---|---|---|
| `*` | Zero or more | `ab*c` | ac, abc, abbc, abbbc |
| `+` | One or more | `ab+c` | abc, abbc (NOT ac) |
| `?` | Zero or one (optional) | `colou?r` | color, colour |
| `{3}` | Exactly 3 | `\d{3}` | 123, 456 |
| `{2,4}` | Between 2 and 4 | `\d{2,4}` | 12, 123, 1234 |
| `{2,}` | 2 or more | `\d{2,}` | 12, 123, 12345678 |
| `{,4}` | Up to 4 | `\d{,4}` | (empty), 1, 12, 123, 1234 |
 
### Greedy vs non-greedy
 
By default, quantifiers are **greedy** — they match as much as possible.
 
```
<b>hello</b> world <b>bye</b>
```
 
- Greedy: `<b>.*</b>` → matches `<b>hello</b> world <b>bye</b>` (spans across both)
- Non-greedy: `<b>.*?</b>` → matches just `<b>hello</b>`
Add `?` after a quantifier to make it non-greedy: `*?`, `+?`, `??`, `{2,4}?`
 
This trips people up constantly when parsing HTML or logs.
 
---
 
## Anchors — position matching
 
| Symbol | Meaning | Example | Matches |
|---|---|---|---|
| `^` | Start of line | `^Error` | Lines starting with "Error" |
| `$` | End of line | `\.exe$` | Lines ending with ".exe" |
| `\b` | Word boundary | `\bcat\b` | "cat" but NOT "category" |
| `\B` | NOT a word boundary | `\Bcat` | "cat" only when NOT at word start |
 
Word boundaries are critical for accurate matches. `cat` matches inside `category`; `\bcat\b` doesn't.
 
---
 
## Character classes — pick from a set
 
| Syntax | Meaning | Example | Matches |
|---|---|---|---|
| `[abc]` | Any one of a, b, or c | `[cb]at` | cat, bat |
| `[a-z]` | Any lowercase letter | `[a-z]+` | hello, world |
| `[A-Z]` | Any uppercase letter |
| `[0-9]` | Any digit (same as `\d`) | `[0-9]{3}` | 123, 999 |
| `[^abc]` | Any character EXCEPT a, b, c | `[^0-9]` | anything not a digit |
| `[a-zA-Z0-9]` | Any letter or digit | `[a-zA-Z0-9]+` | Hello123 |
| `[a-zA-Z0-9_-]` | Common "identifier" class |
 
> **Note:** `^` inside `[]` means "NOT." Outside `[]` it means "start of line." Context matters.
 
Inside `[]`, most metacharacters lose their special meaning. `[.]` matches a literal dot; you don't need to escape it.
 
---
 
## Groups and alternation
 
| Syntax | Meaning | Example | Matches |
|---|---|---|---|
| `(abc)` | Group — treat as one unit | `(ab)+` | ab, abab, ababab |
| `\|` | OR | `cat\|dog` | cat or dog |
| `(cat\|dog)` | Grouped OR | `I have a (cat\|dog)` | "I have a cat" or "I have a dog" |
| `(?:abc)` | Non-capturing group | `(?:ab)+` | same match, but the group isn't stored |
 
### Capture groups
 
Parentheses **capture** what they match, and you can reference those captures later.
 
```
Regex:  user=(\w+) pass=(\S+)
Input:  user=admin pass=Password123!
```
 
Capture group 1 = `admin`, capture group 2 = `Password123!`
 
Depending on tool:
- **Python:** `m.group(1)`, `m.group(2)`
- **sed:** `\1`, `\2`
- **grep -P:** with `\1` in the same pattern (backreference), or via Perl-compatible tools
### Backreferences — refer back to a capture within the same regex
 
```
(\w+)\s+\1
```
Matches the same word twice in a row: `the the`, `is is`.
 
Useful for finding duplicate words in text, or matched quotes:
```
(['"])[^'"]+\1
```
Matches strings quoted with the same character.
 
---
 
## Lookaround — assertions that don't consume characters
 
Advanced but worth learning. Lookarounds check that something is (or isn't) present without including it in the match.
 
| Syntax | Meaning |
|---|---|
| `(?=...)` | Lookahead — followed by `...` |
| `(?!...)` | Negative lookahead — NOT followed by `...` |
| `(?<=...)` | Lookbehind — preceded by `...` |
| `(?<!...)` | Negative lookbehind — NOT preceded by `...` |
 
### Examples
 
**Match a number only if followed by "USD":**
```
\d+(?=USD)
```
On input `100USD 200EUR`, matches `100` (but doesn't include "USD" in the match).
 
**Match a word only if NOT followed by "test":**
```
\bpassword\b(?!.*test)
```
 
**Match a value preceded by "user=" without including "user=" in the match:**
```
(?<=user=)\w+
```
On `user=admin`, matches `admin`.
 
**Extract just the hash from a common /etc/shadow format:**
```
(?<=:)\$[16]\$[^:]+
```
 
Different regex engines have different lookaround support — PCRE (grep -P, Python) supports all four; basic POSIX regex doesn't support lookarounds at all.
 
---
 
## Practical patterns
 
### IPv4 address
 
```regex
\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b
```
 
Strict version (rejects 999.999.999.999):
```regex
\b(?:(?:25[0-5]|2[0-4]\d|1?\d?\d)\.){3}(?:25[0-5]|2[0-4]\d|1?\d?\d)\b
```
 
### Email
 
```regex
[\w.+-]+@[\w.-]+\.\w+
```
 
RFC-strict email regex is famously monstrous; the above catches ~99% of what you actually encounter.
 
### URL
 
```regex
https?://[\w./%?=&#-]+
```
 
### MAC address
 
```regex
([0-9a-fA-F]{2}:){5}[0-9a-fA-F]{2}
```
 
### MD5 hash (32 hex chars)
 
```regex
\b[a-fA-F0-9]{32}\b
```
 
### SHA-1 (40 hex chars) / SHA-256 (64) / SHA-512 (128)
 
```regex
\b[a-fA-F0-9]{40}\b
\b[a-fA-F0-9]{64}\b
\b[a-fA-F0-9]{128}\b
```
 
### CVE identifier
 
```regex
CVE-\d{4}-\d{4,}
```
 
### JWT (3 base64 chunks separated by dots)
 
```regex
eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+
```
 
### Base64 blob
 
```regex
[A-Za-z0-9+/]{20,}={0,2}
```
 
### Windows path
 
```regex
[A-Za-z]:\\(?:[^\\/:*?"<>|\r\n]+\\)*[^\\/:*?"<>|\r\n]*
```
 
### UUID / GUID
 
```regex
[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}
```
 
---
 
## Using regex with grep
 
```bash
# Basic regex (POSIX BRE) — some metacharacters need escaping
grep 'error' /var/log/syslog
 
# Extended regex — cleaner syntax, most metacharacters work as expected
grep -E 'error|warning' /var/log/syslog
 
# Perl-compatible regex — supports lookaround, \d, \w, non-greedy, etc.
grep -P '(?<=user=)\w+' /var/log/auth.log
 
# Case-insensitive
grep -Ei 'error' /var/log/syslog
 
# Show only the match, not the whole line
grep -oE '\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b' /var/log/access.log
 
# Invert — show lines that DON'T match
grep -v 'success' /var/log/auth.log
 
# Show line numbers
grep -n 'error' file.log
 
# Count matches
grep -c 'Failed password' /var/log/auth.log
 
# Recursive
grep -rE 'password' /etc/
```
 
### Common analysis one-liners
 
**Find failed SSH login attempts:**
```bash
grep -E "Failed password.*from \d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}" /var/log/auth.log
```
 
**Extract all unique IPs from a log:**
```bash
grep -oE "\b[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\b" access.log | sort -u
```
 
**Top 10 attacking IPs from auth.log:**
```bash
grep "Failed password" /var/log/auth.log \
    | grep -oE "\b[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\b" \
    | sort | uniq -c | sort -rn | head
```
 
**Find URLs in a suspicious file:**
```bash
grep -oE 'https?://[^ "]+' suspicious.txt
```
 
**Find all email addresses:**
```bash
grep -oE '[[:alnum:]._+-]+@[[:alnum:].-]+\.[[:alpha:]]{2,}' /home/*/mail
```
 
---
 
## Using regex with Python (`re`)
 
The `re` module supports full PCRE-style regex.
 
```python
import re
 
# findall — return all non-overlapping matches
ips = re.findall(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b", log_text)
 
# search — first match anywhere
m = re.search(r"password=(\S+)", config_text)
if m:
    print(m.group(1))     # captured password
 
# match — only at the start of the string
m = re.match(r"user=(\w+)", "user=admin&pass=x")
 
# sub — substitute
sanitized = re.sub(r"password=\S+", "password=REDACTED", text)
 
# Compile for reuse (measurable speedup when reusing many times)
ip_re = re.compile(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b")
for line in log_lines:
    for ip in ip_re.findall(line):
        ...
 
# Named groups
m = re.search(r"user=(?P<user>\w+).*pass=(?P<pw>\S+)", text)
if m:
    print(m.group("user"), m.group("pw"))
    # or m.groupdict() → {"user": "...", "pw": "..."}
```
 
### Flags
 
| Flag | Meaning |
|---|---|
| `re.IGNORECASE` (`re.I`) | Case-insensitive |
| `re.MULTILINE` (`re.M`) | `^` and `$` match start/end of each line |
| `re.DOTALL` (`re.S`) | `.` also matches newlines |
| `re.VERBOSE` (`re.X`) | Allow whitespace and comments in the regex |
 
```python
# Case-insensitive
re.findall(r"error", text, re.I)
 
# Multiline
re.findall(r"^user=(\S+)", multi_line_text, re.M)
 
# Verbose — makes complex regex readable
pattern = re.compile(r"""
    \b                              # word boundary
    (?:\d{1,3}\.){3}                # three "xxx." groups
    \d{1,3}                         # final "xxx"
    \b
""", re.VERBOSE)
```
 
---
 
## Using regex with sed
 
Sed uses a slightly different flavor by default. Use `-E` for extended regex (recommended).
 
```bash
# Replace first occurrence per line
sed -E 's/password=\S+/password=REDACTED/' file.txt
 
# Replace ALL occurrences per line (g flag)
sed -E 's/password=\S+/password=REDACTED/g' file.txt
 
# Edit in place
sed -Ei 's/foo/bar/g' file.txt
 
# Backreferences
echo "John Smith" | sed -E 's/(\w+) (\w+)/\2, \1/'
# → Smith, John
 
# Delete matching lines
sed -E '/^#/d' config.conf              # delete comment lines
sed -E '/^$/d' file.txt                 # delete empty lines
```
 
---
 
## Using regex with awk
 
```bash
# Print lines matching a pattern
awk '/error/' /var/log/syslog
 
# Print field 1 where field 3 matches a pattern
awk '$3 ~ /Failed/ {print $1}' /var/log/auth.log
 
# Use regex on a specific field
awk -F: '$1 ~ /^root|^admin/' /etc/passwd
```
 
---
 
## Testing regex
 
Interactive testers save massive amounts of time when building complex patterns:
 
- **regex101.com** — the gold standard. Explains what each part of your regex does, shows matches in real time, supports Python/PCRE/JS/Go flavors.
- **regexr.com** — similar, slightly friendlier UI.
- **rubular.com** — Ruby-flavored but useful.
For quick command-line testing:
```bash
echo "test input" | grep -Po 'your.*pattern'
```
 
---
