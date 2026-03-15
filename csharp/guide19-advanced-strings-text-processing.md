# C# Advanced Strings and Text Processing Quick Reference

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                  STRING PROCESSING DECISION MAP                          │
│                                                                          │
│  Goal                        Preferred Approach                          │
│  ────────────────────────────────────────────────────────────────────── │
│  Compare strings (equals)    StringComparison.Ordinal(IgnoreCase)        │
│  Compare for sorting (UI)    StringComparer.CurrentCulture               │
│  Split / parse text          ReadOnlySpan<char> / string.Split           │
│  Pattern match               Regex (precompiled / source generated)      │
│  Build large strings         StringBuilder / string.Create               │
│  Scan/slice without alloc    ReadOnlySpan<char>                          │
│  Cross-language text         Normalize to NFC before compare             │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Unicode Basics for Developers

### Key Concepts

| Term | Meaning | Example |
|------|---------|---------|
| Code point | A Unicode number (U+XXXX) | U+0041 = 'A' |
| Char (C# `char`) | UTF-16 code unit (16-bit) | `'A'` = one char |
| Surrogate pair | Two chars for code points > U+FFFF | 😀 = `\uD83D\uDE00` |
| Grapheme cluster | What a user perceives as a "character" | `é` may be 1 or 2 code points |
| Rune (C# `Rune`) | A single Unicode code point (.NET 5+) | `new Rune(0x1F600)` |

```csharp
// ⚠️ "character count" depends on what you mean
string emoji = "😀";
Console.WriteLine(emoji.Length);          // 2 (two UTF-16 chars / surrogates)
Console.WriteLine(emoji.EnumerateRunes().Count()); // 1 (one code point)

// Counting grapheme clusters — for display / user-facing length
// Install: Microsoft.Globalization
using System.Globalization;
var ei = StringInfo.GetTextElementEnumerator("café");
int graphemes = 0;
while (ei.MoveNext()) graphemes++;
// café may be 4 or 5 chars internally depending on normalization

// Rune — iterate code points safely (handles surrogates)
foreach (Rune rune in "hello 😀".EnumerateRunes())
    Console.Write($"U+{rune.Value:X4} ");
```

---

## 2. Normalization and Comparison Correctness

```csharp
// "é" can be ONE precomposed char (NFC) or TWO chars: 'e' + combining accent (NFD)
string nfc = "\u00E9";          // é  (precomposed)
string nfd = "e\u0301";         // é  (decomposed)

Console.WriteLine(nfc == nfd);  // FALSE — different bytes!
Console.WriteLine(nfc.Length);  // 1
Console.WriteLine(nfd.Length);  // 2

// Normalize before comparing cross-system text
string Normalize(string s) => s.Normalize(NormalizationForm.FormC);  // NFC
Console.WriteLine(Normalize(nfc) == Normalize(nfd));  // TRUE

// Normalization forms
// NFC  — precomposed (most common, used by Windows/Web)
// NFD  — decomposed (used by macOS file system)
// NFKC — compatibility + precomposed (folding ligatures: ﬁ → fi)
// NFKD — compatibility + decomposed
```

---

## 3. Culture-Aware vs Ordinal Comparisons

```csharp
// ❌ Anti-pattern: implicit CurrentCulture on a server
bool eq = str1 == str2;                    // CurrentCulture on some runtimes
bool eq2 = str1.Equals(str2);             // same issue — avoid for identifiers

// ✅ Ordinal — fast, byte-by-byte, no culture rules
bool same    = str1.Equals(str2, StringComparison.Ordinal);
bool sameCI  = str1.Equals(str2, StringComparison.OrdinalIgnoreCase);
int  cmp     = string.Compare(a, b, StringComparison.Ordinal);

// ✅ Culture — for user-visible sorting / display
int uiCmp = string.Compare(a, b, StringComparison.CurrentCulture);
var sorted = list.OrderBy(s => s, StringComparer.CurrentCulture);

// Decision table
// Use case                         → StringComparison
// --------------------------------------------------
// Identifiers, URLs, file paths   → Ordinal
// Case-insensitive IDs            → OrdinalIgnoreCase
// Sorting displayed to users      → CurrentCulture
// Locale-neutral file comparison  → InvariantCulture
// Password hash input check       → Ordinal
```

### Anti-Pattern: ToLower / ToUpper for Comparison

```csharp
// ❌ Wrong — allocates a new string, culture issues with Turkish 'i'
if (input.ToLower() == "admin") { }
if (input.ToUpper() == "ADMIN") { }

// ✅ Correct — zero allocation, ordinal
if (input.Equals("admin", StringComparison.OrdinalIgnoreCase)) { }
if (input.StartsWith("sys_", StringComparison.OrdinalIgnoreCase)) { }

// Turkish problem: in tr-TR culture, 'i'.ToUpper() == 'İ' not 'I'
// ToLowerInvariant() / ToUpperInvariant() avoid this — but still allocate
```

---

## 4. Efficient Text Processing with Span<T> / ReadOnlySpan<char>

```csharp
// ReadOnlySpan<char> — a view into string memory, zero allocation
string text = "Hello, World!";
ReadOnlySpan<char> span = text.AsSpan();

ReadOnlySpan<char> hello = span[..5];       // "Hello" — no allocation
ReadOnlySpan<char> world = span[7..12];     // "World" — no allocation

// Slice and check without allocating
bool startsWithHttp = span.StartsWith("http", StringComparison.OrdinalIgnoreCase);
int  commaIdx       = span.IndexOf(',');

// ✅ Process CSV fields without splitting into string[]
static void ParseCsvLine(ReadOnlySpan<char> line)
{
    int start = 0;
    while (true)
    {
        int comma = line[start..].IndexOf(',');
        if (comma < 0)
        {
            ProcessField(line[start..]);
            break;
        }
        ProcessField(line[start..(start + comma)]);
        start += comma + 1;
    }
}

static void ProcessField(ReadOnlySpan<char> field) { /* parse/use */ }

// Convert span to string only when needed (e.g., to store)
string stored = hello.ToString();
```

### string.Create — Efficient String Construction

```csharp
// Avoid intermediate char[] or StringBuilder for fixed-format strings
static string FormatId(int prefix, int id)
    => string.Create(10, (prefix, id), static (span, state) =>
    {
        span[0] = (char)('A' + state.prefix);
        state.id.TryFormat(span[1..], out _);
    });
```

---

## 5. StringBuilder — Efficient String Building

```csharp
// ✅ Use when concatenating in a loop
var sb = new StringBuilder(capacity: 1024);  // pre-size when possible

foreach (var item in items)
{
    sb.Append(item.Name)
      .Append('\t')
      .AppendLine(item.Value.ToString("F2"));
}

string result = sb.ToString();

// ✅ Reuse StringBuilder with Clear()
sb.Clear();   // resets length, keeps buffer — better than new StringBuilder()

// ReadOnlySpan overloads (avoids string copies)
sb.Append(someSpan);

// AppendFormat is slower than multiple Append calls — prefer explicit Append chains
```

---

## 6. Tokenizer / Split / Parser Patterns

```csharp
// string.Split — simple, allocates array of strings
string[] parts = "a,b,,c".Split(',');                 // ["a","b","","c"]
string[] noBlanks = "a,b,,c".Split(',',
    StringSplitOptions.RemoveEmptyEntries);            // ["a","b","c"]
string[] limited = "a:b:c:d".Split(':', 3);           // ["a","b","c:d"]

// Span-based split — zero allocation, .NET 8+
foreach (Range range in "a,b,c".AsSpan().Split(','))
{
    ReadOnlySpan<char> token = "a,b,c".AsSpan()[range];
    Console.WriteLine(token.ToString());
}

// Manual tokenizer for complex grammars
static IEnumerable<string> Tokenize(string input, char delimiter = ' ')
{
    int start = 0;
    for (int i = 0; i <= input.Length; i++)
    {
        if (i == input.Length || input[i] == delimiter)
        {
            if (i > start)
                yield return input[start..i];
            start = i + 1;
        }
    }
}
```

---

## 7. Regex Safety and Performance Tips

```csharp
using System.Text.RegularExpressions;

// ✅ Precompile — compile once, use many times
private static readonly Regex EmailRegex = new(
    @"^[^@\s]+@[^@\s]+\.[^@\s]+$",
    RegexOptions.Compiled | RegexOptions.IgnoreCase,
    matchTimeout: TimeSpan.FromMilliseconds(100)  // always set timeout
);

bool isEmail = EmailRegex.IsMatch(input);

// ✅ Source-generated Regex (.NET 7+) — fastest, AOT-safe
[GeneratedRegex(@"^\d{4}-\d{2}-\d{2}$", RegexOptions.None)]
private static partial Regex DatePattern();

bool isDate = DatePattern().IsMatch("2024-03-15");

// Span-based matching — avoids string allocation
bool match = EmailRegex.IsMatch(input.AsSpan());

// Named captures
var match2 = Regex.Match("John Doe", @"(?<first>\w+)\s(?<last>\w+)");
string first = match2.Groups["first"].Value;
string last  = match2.Groups["last"].Value;

// Replace with span/lambda
string clean = Regex.Replace(input, @"\s+", " ");
```

### Regex Pitfalls — Catastrophic Backtracking

```csharp
// ❌ DANGER: nested quantifiers — O(2^n) on certain inputs
var bad = new Regex(@"(a+)+$");
bad.IsMatch("aaaaaaaaaaaaaaab");  // hangs!

// ✅ FIX: atomic groups or possessive quantifiers (where supported)
// ✅ Always set matchTimeout to prevent ReDoS
var safe = new Regex(@"a+$",
    RegexOptions.Compiled,
    TimeSpan.FromMilliseconds(50));

// ✅ Validate regex patterns that accept user-defined patterns
```

---

## 8. UTF-8 String Literals (.NET 7+)

```csharp
// UTF-8 byte literals — avoid encoding cost for known ASCII/UTF-8 content
ReadOnlySpan<byte> header = "Content-Type: application/json\r\n"u8;

// Useful for HTTP headers, fixed byte sequences, protocol parsing
// The suffix u8 creates a ReadOnlySpan<byte>, no heap allocation
static ReadOnlySpan<byte> Separator => ": "u8;
```

---

## 9. Practical Text Processing Recipes

### Recipe 1: Truncate to N Display Characters (Grapheme-Safe)

```csharp
static string TruncateGrapheme(string text, int maxChars)
{
    var info = new StringInfo(text);
    if (info.LengthInTextElements <= maxChars) return text;
    return info.SubstringByTextElements(0, maxChars) + "…";
}
```

### Recipe 2: Slugify a String

```csharp
static string Slugify(string input)
{
    string normalized = input.Normalize(NormalizationForm.FormD);
    var sb = new StringBuilder();
    foreach (char c in normalized)
    {
        if (CharUnicodeInfo.GetUnicodeCategory(c) == UnicodeCategory.NonSpacingMark)
            continue;
        if (char.IsLetterOrDigit(c))
            sb.Append(char.ToLowerInvariant(c));
        else if (c is ' ' or '-' or '_')
            sb.Append('-');
    }
    return sb.ToString().Trim('-');
}
// "Héllo Wörld!" → "hello-world"
```

### Recipe 3: Count Words (Ordinal, No Regex)

```csharp
static int CountWords(ReadOnlySpan<char> text)
{
    int count   = 0;
    bool inWord = false;
    foreach (char c in text)
    {
        if (char.IsWhiteSpace(c)) { inWord = false; }
        else if (!inWord)         { inWord = true; count++; }
    }
    return count;
}
```

### Recipe 4: Levenshtein Distance (Edit Distance)

```csharp
static int Levenshtein(ReadOnlySpan<char> s, ReadOnlySpan<char> t)
{
    int m = s.Length, n = t.Length;
    int[] prev = new int[n + 1], curr = new int[n + 1];
    for (int j = 0; j <= n; j++) prev[j] = j;
    for (int i = 1; i <= m; i++)
    {
        curr[0] = i;
        for (int j = 1; j <= n; j++)
            curr[j] = s[i - 1] == t[j - 1]
                ? prev[j - 1]
                : 1 + Math.Min(prev[j - 1], Math.Min(prev[j], curr[j - 1]));
        (prev, curr) = (curr, prev);
    }
    return prev[n];
}
```

---

## Anti-Patterns

```
❌ Using ToLower() / ToUpper() for case-insensitive comparison — allocates + culture issues
❌ string == other without StringComparison — implicit culture on some runtimes
❌ string.Length to count user-visible characters — ignores surrogates/graphemes
❌ Regex without matchTimeout — vulnerability to ReDoS
❌ string concatenation in a loop — use StringBuilder
❌ Splitting to string[] when only scanning — use Span-based split
❌ Forgetting to Normalize() before comparing multilingual text
❌ Using Regex.IsMatch() with a new Regex per call in hot paths — precompile
```

---

## Best Practices

```
✅ Use StringComparison.Ordinal for identifiers, keys, paths, tokens
✅ Use StringComparison.CurrentCulture for UI-displayed text sorting
✅ Normalize strings to NFC before cross-system comparison
✅ Use ReadOnlySpan<char> for scan/slice operations in hot paths
✅ Precompile Regex or use [GeneratedRegex] — always set timeout
✅ Use string.Create() for structured fixed-format string construction
✅ Use StringBuilder.Clear() to reuse builders in loops
✅ Use "..."u8 UTF-8 literals for protocol-level byte sequences
✅ Count grapheme clusters (StringInfo) for display length, not .Length
```

---

## Quick Reference Summary

| Task | API |
|------|-----|
| Compare strings | `Equals(s, StringComparison.Ordinal)` |
| Case-insensitive match | `Equals(s, StringComparison.OrdinalIgnoreCase)` |
| Sort for UI | `StringComparer.CurrentCulture` |
| Normalize Unicode | `s.Normalize(NormalizationForm.FormC)` |
| Slice without alloc | `s.AsSpan()[start..end]` |
| Split without alloc | `span.Split(delimiter)` (.NET 8+) |
| Build strings | `StringBuilder` / `string.Create` |
| Precompiled regex | `new Regex(pattern, RegexOptions.Compiled, timeout)` |
| Generated regex | `[GeneratedRegex(...)]` attribute |
| Grapheme length | `new StringInfo(s).LengthInTextElements` |
| Code point iteration | `s.EnumerateRunes()` |
| UTF-8 literal | `"text"u8` → `ReadOnlySpan<byte>` |

---

**Guide Complete!** Mastering Unicode normalization, ordinal comparisons, and Span-based text processing will make your code both correct and allocation-efficient. 📘
