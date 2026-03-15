# C# Data Conversion and Parsing Cookbook

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                     CONVERSION DECISION TREE                             │
│                                                                          │
│  Input known valid?                                                      │
│     YES → Parse() / explicit cast / Convert.ToX()                       │
│     NO  → TryParse() / as / pattern matching                             │
│                                                                          │
│  Culture-sensitive? (user input / UI)                                    │
│     YES → Pass CultureInfo.CurrentCulture                                │
│     NO  → Pass CultureInfo.InvariantCulture (APIs, files, DBs)          │
│                                                                          │
│  Lossless required?                                                      │
│     YES → Avoid float/double; use decimal or exact types                 │
│     NO  → Widening conversions are safe (int → long → double)            │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. String ↔ Number

### Safe Parsing with TryParse

```csharp
// ✅ TryParse — never throws, always check return value
if (int.TryParse("42", out int value))
    Console.WriteLine(value);  // 42

// Culture-safe: parse "1,234.56" from en-US input
if (double.TryParse("1,234.56",
        NumberStyles.Number,
        CultureInfo.InvariantCulture,
        out double amount))
    Console.WriteLine(amount);  // 1234.56

// Span<char> overload — zero allocation
ReadOnlySpan<char> span = "123".AsSpan();
if (int.TryParse(span, out int result))
    Console.WriteLine(result);
```

### Parse When Input is Guaranteed Valid

```csharp
int n       = int.Parse("42");
long l      = long.Parse("9999999999");
decimal d   = decimal.Parse("19.99", CultureInfo.InvariantCulture);
double dbl  = double.Parse("3.14",   CultureInfo.InvariantCulture);
```

### Number → String

```csharp
int n = 1234567;

n.ToString();                              // "1234567"
n.ToString("N0", CultureInfo.InvariantCulture); // "1,234,567"
n.ToString("X");                           // "12D687" (hex)
n.ToString("D8");                          // "01234567" (zero-padded)

decimal price = 19.99m;
price.ToString("C", CultureInfo.GetCultureInfo("en-US")); // "$19.99"
price.ToString("F2", CultureInfo.InvariantCulture);       // "19.99"
```

### Lossless vs Lossy Conversion Table

| From → To | Lossless? | Method |
|-----------|-----------|--------|
| int → long | ✅ Yes | implicit |
| int → double | ⚠️ Mostly (large ints lose precision) | implicit |
| double → int | ❌ No | explicit cast, truncates |
| double → decimal | ❌ No (precision differs) | `Convert.ToDecimal()` |
| decimal → double | ❌ No | explicit cast |
| float → double | ✅ Yes | implicit |
| long → int | ❌ No | explicit + `checked` |

```csharp
// Checked arithmetic: throw on overflow
checked
{
    int x = int.MaxValue;
    int y = x + 1;  // throws OverflowException
}

// Unchecked (default): silently wraps
int z = unchecked(int.MaxValue + 1);  // -2147483648
```

---

## 2. String ↔ DateTime / DateOnly / TimeOnly

### Parsing Dates

```csharp
using System.Globalization;

// Exact format — safest for APIs and files
string iso = "2024-03-15T10:30:00Z";
DateTime dt = DateTime.ParseExact(iso, "yyyy-MM-ddTHH:mm:ssZ",
                  CultureInfo.InvariantCulture, DateTimeStyles.AdjustToUniversal);

// TryParseExact — never throws
if (DateTime.TryParseExact("15/03/2024", "dd/MM/yyyy",
        CultureInfo.InvariantCulture, DateTimeStyles.None, out DateTime parsed))
    Console.WriteLine(parsed);

// ISO 8601 shortcut
DateTime utc = DateTime.Parse("2024-03-15T10:30:00Z",
                   null, DateTimeStyles.RoundtripKind);

// DateTimeOffset for timezone-aware data
DateTimeOffset dto = DateTimeOffset.Parse("2024-03-15T10:30:00+05:30");
```

### DateOnly / TimeOnly (.NET 6+)

```csharp
// DateOnly — no time component
if (DateOnly.TryParse("2024-03-15", out DateOnly date))
    Console.WriteLine(date.DayOfWeek);  // Friday

DateOnly d = DateOnly.ParseExact("15-03-2024", "dd-MM-yyyy");

// TimeOnly
if (TimeOnly.TryParse("14:30", out TimeOnly time))
    Console.WriteLine(time.Hour);  // 14

TimeOnly t = TimeOnly.ParseExact("02:30 PM", "hh:mm tt",
                 CultureInfo.InvariantCulture);
```

### Formatting Dates

```csharp
DateTime now = DateTime.UtcNow;

now.ToString("yyyy-MM-dd");              // "2024-03-15"
now.ToString("o");                       // ISO 8601 round-trip: "2024-03-15T10:30:00.0000000Z"
now.ToString("R");                       // RFC 1123: "Fri, 15 Mar 2024 10:30:00 GMT"
now.ToString("dd/MM/yyyy HH:mm");        // "15/03/2024 10:30"

// Span-based formatting — avoids allocation
Span<char> buf = stackalloc char[20];
now.TryFormat(buf, out int written, "yyyy-MM-dd");
```

---

## 3. String ↔ Enum

```csharp
public enum Status { Pending, Active, Closed }

// Parse — throws if invalid
Status s = Enum.Parse<Status>("Active");

// TryParse — safe, case-insensitive option
if (Enum.TryParse<Status>("active", ignoreCase: true, out Status status))
    Console.WriteLine(status);  // Active

// From integer
Status fromInt = (Status)1;   // Active
int asInt = (int)Status.Active; // 1

// Check validity (integers can be out of range)
int raw = 99;
if (Enum.IsDefined(typeof(Status), raw))
    Status safe = (Status)raw;

// All values / names
Status[] all   = Enum.GetValues<Status>();
string[] names = Enum.GetNames<Status>();

// Flags enums
[Flags] public enum Permission { None = 0, Read = 1, Write = 2, Execute = 4 }
Permission p = Permission.Read | Permission.Write;
bool canRead = p.HasFlag(Permission.Read);  // true
```

---

## 4. Object ↔ String and Null-Safe Helpers

```csharp
// Object to string — null-safe
static string ToStringSafe(object? obj, string fallback = "")
    => obj?.ToString() ?? fallback;

// Convert.ToString handles DBNull and null gracefully
string s = Convert.ToString(someObject) ?? string.Empty;

// Pattern: Convert.ToX with fallback
static T ConvertOrDefault<T>(object? value, T fallback = default!)
{
    if (value is null || value is DBNull) return fallback;
    try   { return (T)Convert.ChangeType(value, typeof(T)); }
    catch { return fallback; }
}

// Usage
int age = ConvertOrDefault<int>(reader["Age"], 0);
```

---

## 5. Bytes ↔ Hex / Base64

### Hex

```csharp
// Bytes → Hex
byte[] data = [0xDE, 0xAD, 0xBE, 0xEF];
string hex  = Convert.ToHexString(data);           // "DEADBEEF"
string hexL = Convert.ToHexString(data).ToLower(); // "deadbeef"

// Hex → Bytes
byte[] back = Convert.FromHexString("DEADBEEF");

// Manual (if targeting older .NET)
static string BytesToHex(byte[] bytes)
    => string.Concat(bytes.Select(b => b.ToString("x2")));

static byte[] HexToBytes(string hex)
{
    byte[] result = new byte[hex.Length / 2];
    for (int i = 0; i < result.Length; i++)
        result[i] = Convert.ToByte(hex.Substring(i * 2, 2), 16);
    return result;
}
```

### Base64

```csharp
// Bytes → Base64
byte[] data   = [1, 2, 3, 4, 5];
string b64    = Convert.ToBase64String(data);      // standard
string b64Url = Convert.ToBase64String(data)
                    .Replace("+", "-")
                    .Replace("/", "_")
                    .TrimEnd('=');                 // URL-safe

// Base64 → Bytes
byte[] decoded    = Convert.FromBase64String(b64);

// ✅ TryFromBase64 — avoids exception on invalid input
byte[] buf = new byte[1024];
if (Convert.TryFromBase64String(userInput, buf, out int written))
{
    Span<byte> result = buf.AsSpan(0, written);
}
```

---

## 6. Char[] / String ↔ Byte[] (UTF-8)

```csharp
using System.Text;

// String → UTF-8 bytes
string text    = "Hello, Мир";
byte[] utf8    = Encoding.UTF8.GetBytes(text);

// UTF-8 bytes → string
string decoded = Encoding.UTF8.GetString(utf8);

// Span-based — no intermediate array allocation
ReadOnlySpan<char> chars = "Hello".AsSpan();
byte[] bytes = new byte[Encoding.UTF8.GetByteCount(chars)];
int written  = Encoding.UTF8.GetBytes(chars, bytes);

// Stackalloc for short strings
Span<byte> stack = stackalloc byte[256];
if (Encoding.UTF8.TryGetBytes("Short text".AsSpan(), stack, out int n))
{
    // Use stack[..n]
}

// char[] <-> string
char[] chars2 = "hello".ToCharArray();
string str    = new string(chars2);
string sub    = new string(chars2, 1, 3);  // "ell"
```

---

## 7. Numeric Conversions Between Types

```csharp
// Widening — always safe, implicit
int i     = 42;
long l    = i;       // int → long
double d  = i;       // int → double
decimal m = i;       // int → decimal

// Narrowing — requires explicit cast, may lose data
double dbl = 3.99;
int narrow = (int)dbl;        // 3 (truncates, no rounding)
int rounded = (int)Math.Round(dbl); // 4

// Safe narrowing with boundary check
static int ToInt32Safe(long value)
    => value is >= int.MinValue and <= int.MaxValue
       ? (int)value
       : throw new OverflowException($"Value {value} out of Int32 range.");

// BitConverter — binary layout interpretation
byte[] floatBytes = BitConverter.GetBytes(3.14f);
float  restored   = BitConverter.ToSingle(floatBytes, 0);
```

---

## 8. Parsing Edge Cases and Pitfalls

```csharp
// ❌ Pitfall: culture-sensitive parsing on a server
double.Parse("1.234");  // Works in en-US, fails in de-DE (comma decimal separator)

// ✅ Fix: always pass InvariantCulture for machine-to-machine data
double.Parse("1.234", CultureInfo.InvariantCulture);

// ❌ Pitfall: DateTime.Parse may interpret ambiguously
DateTime.Parse("01/02/03");  // Is it Jan 2 2003? Feb 1 2003?

// ✅ Fix: use ParseExact
DateTime.ParseExact("01/02/03", "MM/dd/yy", CultureInfo.InvariantCulture);

// ❌ Pitfall: overflow silently wraps
byte b = (byte)300;  // 44 — silent data corruption

// ✅ Fix: checked cast
byte safe = checked((byte)300);  // throws OverflowException

// ❌ Pitfall: float equality after conversion
float f = 0.1f;
double fd = (double)f;  // 0.10000000149011612 — not 0.1!

// ✅ Fix: use decimal for financial data
decimal exact = 0.1m;
```

---

## Anti-Patterns

```
❌ Parsing user-facing numbers without CultureInfo
❌ Using Parse() on untrusted input (throw risk) — use TryParse()
❌ Using == to compare floats after conversion
❌ Converting double to decimal directly for money
❌ Using unchecked int casts when overflow is a concern
❌ Ignoring Convert.ChangeType failures — wrap in try/catch
❌ Using DateTime.Now for cross-timezone serialization (use UtcNow)
```

---

## Best Practices

```
✅ Always use TryParse() for user or external input
✅ Pass CultureInfo.InvariantCulture for all non-UI parsing
✅ Use decimal for monetary / financial values, not double
✅ Use DateOnly/TimeOnly when time or date component is irrelevant
✅ Prefer Convert.ToHexString() over manual hex formatting
✅ Use Encoding.UTF8.GetBytes(ReadOnlySpan<char>) to avoid string copies
✅ Validate Enum.IsDefined() before casting integers to enums
✅ Use the "o" (round-trip) format when serializing DateTime to strings
```

---

## Quick Reference Summary

| Conversion | Best Method |
|-----------|------------|
| string → int | `int.TryParse(s, out int n)` |
| string → decimal | `decimal.TryParse(s, NumberStyles.Number, CultureInfo.InvariantCulture, out decimal d)` |
| string → DateTime | `DateTime.TryParseExact(s, fmt, CultureInfo.InvariantCulture, ...)` |
| string → DateOnly | `DateOnly.TryParse(s, out DateOnly d)` |
| string → Enum | `Enum.TryParse<T>(s, true, out T e)` |
| bytes → hex | `Convert.ToHexString(bytes)` |
| hex → bytes | `Convert.FromHexString(hex)` |
| bytes → base64 | `Convert.ToBase64String(bytes)` |
| string → UTF-8 | `Encoding.UTF8.GetBytes(str)` |
| object → T | `ConvertOrDefault<T>(obj, fallback)` |

---

**Guide Complete!** These recipes handle 95% of real-world data conversion scenarios in production C# code. Always validate, always specify culture, always prefer TryParse. 📘
