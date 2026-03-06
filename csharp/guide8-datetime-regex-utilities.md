# C# DateTime, Regex & Common Utilities Quick Reference

---

## Part 1: DateTime & Time Management

### 1. DateTime Struct

#### Creating DateTime

```csharp
// Constructor
DateTime dt1 = new DateTime(2024, 12, 17);                    // Date only
DateTime dt2 = new DateTime(2024, 12, 17, 14, 30, 0);         // Date and time
DateTime dt3 = new DateTime(2024, 12, 17, 14, 30, 0, 500);    // With milliseconds

// Current date/time
DateTime now = DateTime.Now;              // Local time
DateTime utcNow = DateTime.UtcNow;        // UTC time
DateTime today = DateTime.Today;          // Today at midnight (00:00:00)

// Parsing
DateTime dt4 = DateTime.Parse("12/17/2024");
DateTime dt5 = DateTime.Parse("2024-12-17T14:30:00");

// Safe parsing
if (DateTime.TryParse("12/17/2024", out DateTime result))
{
    Console.WriteLine(result);
}

// Exact format
DateTime dt6 = DateTime.ParseExact(
    "17-12-2024", 
    "dd-MM-yyyy", 
    CultureInfo.InvariantCulture);

// Try parse exact
if (DateTime.TryParseExact(
    "17-12-2024", 
    "dd-MM-yyyy", 
    CultureInfo.InvariantCulture,
    DateTimeStyles.None,
    out DateTime exactResult))
{
    Console.WriteLine(exactResult);
}

// Min and Max values
DateTime min = DateTime.MinValue;  // 01/01/0001 00:00:00
DateTime max = DateTime.MaxValue;  // 12/31/9999 23:59:59
```

#### DateTime Properties

```csharp
DateTime dt = new DateTime(2024, 12, 17, 14, 30, 45, 500);

// Date components
int year = dt.Year;          // 2024
int month = dt.Month;        // 12
int day = dt.Day;            // 17
DayOfWeek dayOfWeek = dt.DayOfWeek;  // Tuesday
int dayOfYear = dt.DayOfYear;        // 352

// Time components
int hour = dt.Hour;          // 14 (24-hour)
int minute = dt.Minute;      // 30
int second = dt.Second;      // 45
int millisecond = dt.Millisecond;  // 500

// Date and time separately
DateTime dateOnly = dt.Date;          // 12/17/2024 00:00:00
TimeSpan timeOnly = dt.TimeOfDay;     // 14:30:45.5000000

// Ticks (100-nanosecond intervals since 01/01/0001)
long ticks = dt.Ticks;

// Kind (Local, Utc, or Unspecified)
DateTimeKind kind = dt.Kind;
```

#### DateTime Methods

```csharp
DateTime dt = new DateTime(2024, 12, 17, 14, 30, 0);

// Adding time
DateTime tomorrow = dt.AddDays(1);
DateTime nextWeek = dt.AddDays(7);
DateTime nextMonth = dt.AddMonths(1);
DateTime nextYear = dt.AddYears(1);
DateTime later = dt.AddHours(2);
DateTime plusMinutes = dt.AddMinutes(30);
DateTime plusSeconds = dt.AddSeconds(45);
DateTime plusMilliseconds = dt.AddMilliseconds(500);
DateTime plusTicks = dt.AddTicks(10000);

// Subtracting time
DateTime yesterday = dt.AddDays(-1);
DateTime lastMonth = dt.AddMonths(-1);

// TimeSpan operations
TimeSpan span = TimeSpan.FromDays(5);
DateTime future = dt.Add(span);
DateTime past = dt.Subtract(span);

// Difference between dates
DateTime date1 = new DateTime(2024, 12, 17);
DateTime date2 = new DateTime(2024, 12, 25);
TimeSpan difference = date2.Subtract(date1);  // or date2 - date1
Console.WriteLine(difference.Days);  // 8

// Comparison
bool isEqual = dt1.Equals(dt2);
int comparison = dt1.CompareTo(dt2);  // -1, 0, or 1
bool isBefore = dt1 < dt2;
bool isAfter = dt1 > dt2;

// Static comparison
int compare = DateTime.Compare(dt1, dt2);

// Time zone conversion
DateTime local = dt.ToLocalTime();
DateTime utc = dt.ToUniversalTime();

// String representations
string shortDate = dt.ToShortDateString();    // 12/17/2024
string longDate = dt.ToLongDateString();      // Tuesday, December 17, 2024
string shortTime = dt.ToShortTimeString();    // 2:30 PM
string longTime = dt.ToLongTimeString();      // 2:30:00 PM
```

### DateTime Quick Reference Table

| Property/Method | Description | Example |
|----------------|-------------|---------|
| **DateTime.Now** | Current local time | `DateTime.Now` |
| **DateTime.UtcNow** | Current UTC time | `DateTime.UtcNow` |
| **DateTime.Today** | Today at midnight | `DateTime.Today` |
| **Year, Month, Day** | Date components | `dt.Year` → 2024 |
| **Hour, Minute, Second** | Time components | `dt.Hour` → 14 |
| **DayOfWeek** | Day of week enum | `dt.DayOfWeek` → Tuesday |
| **DayOfYear** | Day number (1-366) | `dt.DayOfYear` → 352 |
| **AddDays(n)** | Add days | `dt.AddDays(7)` |
| **AddMonths(n)** | Add months | `dt.AddMonths(1)` |
| **AddYears(n)** | Add years | `dt.AddYears(1)` |
| **Subtract(date)** | Time difference | `date2 - date1` → TimeSpan |
| **CompareTo(date)** | Compare dates | Returns -1, 0, or 1 |

---

### 2. DateTimeOffset (C# 2.0+)

**Purpose:** DateTime with time zone offset

```csharp
// Create with offset
DateTimeOffset dto1 = new DateTimeOffset(2024, 12, 17, 14, 30, 0, TimeSpan.FromHours(-5));
DateTimeOffset dto2 = DateTimeOffset.Now;              // Local time with offset
DateTimeOffset dto3 = DateTimeOffset.UtcNow;           // UTC time

// Properties
DateTime dateTime = dto1.DateTime;    // DateTime component
TimeSpan offset = dto1.Offset;        // Time zone offset (-05:00)
DateTime utc = dto1.UtcDateTime;      // Converted to UTC

// Convert between time zones
DateTimeOffset local = dto1.ToLocalTime();
DateTimeOffset utc = dto1.ToUniversalTime();
```

**DateTime vs DateTimeOffset:**

| Feature | DateTime | DateTimeOffset |
|---------|----------|----------------|
| **Time zone info** | Kind property only | Explicit offset |
| **Precision** | Local/UTC/Unspecified | Exact moment in time |
| **Use when** | Local times, dates only | API responses, logging, distributed systems |
| **Best for** | UI, user input | Backend, databases, APIs |

**When to use DateTimeOffset:**

- ✅ Storing timestamps in databases
- ✅ API responses/requests
- ✅ Logging across time zones
- ✅ Distributed systems

---

### 3. DateOnly & TimeOnly (.NET 6.0+)

```csharp
// DateOnly - date without time
DateOnly date = new DateOnly(2024, 12, 17);
DateOnly today = DateOnly.FromDateTime(DateTime.Now);

int year = date.Year;
int month = date.Month;
int day = date.Day;
DayOfWeek dayOfWeek = date.DayOfWeek;

DateOnly tomorrow = date.AddDays(1);
DateOnly nextMonth = date.AddMonths(1);

// TimeOnly - time without date
TimeOnly time = new TimeOnly(14, 30, 0);  // 2:30:00 PM
TimeOnly now = TimeOnly.FromDateTime(DateTime.Now);

int hour = time.Hour;
int minute = time.Minute;
int second = time.Second;

TimeOnly later = time.AddHours(2);
TimeOnly earlier = time.AddMinutes(-30);

// Combine
DateTime combined = date.ToDateTime(time);
```

**Use Cases:**

- `DateOnly` - Birthdays, holidays, appointment dates
- `TimeOnly` - Business hours, schedules, alarms

---

### 4. TimeSpan Struct

#### Creating TimeSpan

```csharp
// Constructor
TimeSpan ts1 = new TimeSpan(1, 2, 3);           // 1 hour, 2 min, 3 sec
TimeSpan ts2 = new TimeSpan(2, 1, 30, 45);      // 2 days, 1 hour, 30 min, 45 sec
TimeSpan ts3 = new TimeSpan(5, 12, 30, 45, 500); // days, hours, min, sec, ms

// Factory methods
TimeSpan day = TimeSpan.FromDays(1);
TimeSpan hour = TimeSpan.FromHours(1);
TimeSpan minute = TimeSpan.FromMinutes(30);
TimeSpan second = TimeSpan.FromSeconds(45);
TimeSpan millisecond = TimeSpan.FromMilliseconds(500);
TimeSpan tick = TimeSpan.FromTicks(10000);

// From difference
DateTime start = new DateTime(2024, 12, 17, 10, 0, 0);
DateTime end = new DateTime(2024, 12, 17, 14, 30, 0);
TimeSpan duration = end - start;  // 4 hours, 30 minutes

// Parse
TimeSpan ts4 = TimeSpan.Parse("1:30:00");  // 1 hour, 30 minutes
```

#### TimeSpan Properties

```csharp
TimeSpan ts = new TimeSpan(2, 14, 30, 45, 500);

// Component properties
int days = ts.Days;              // 2
int hours = ts.Hours;            // 14
int minutes = ts.Minutes;        // 30
int seconds = ts.Seconds;        // 45
int milliseconds = ts.Milliseconds;  // 500

// Total properties (as double)
double totalDays = ts.TotalDays;            // 2.604225694...
double totalHours = ts.TotalHours;          // 62.5014583...
double totalMinutes = ts.TotalMinutes;      // 3750.0875
double totalSeconds = ts.TotalSeconds;      // 225005.25
double totalMilliseconds = ts.TotalMilliseconds;  // 225005250

// Ticks
long ticks = ts.Ticks;
```

#### TimeSpan Operations

```csharp
TimeSpan ts1 = TimeSpan.FromHours(2);
TimeSpan ts2 = TimeSpan.FromMinutes(30);

// Arithmetic
TimeSpan sum = ts1.Add(ts2);          // or ts1 + ts2
TimeSpan diff = ts1.Subtract(ts2);    // or ts1 - ts2
TimeSpan doubled = ts1.Multiply(2);   // or ts1 * 2  (C# 7.0+)
TimeSpan halved = ts1.Divide(2);      // or ts1 / 2  (C# 7.0+)

// Negate
TimeSpan negative = ts1.Negate();     // or -ts1

// Absolute value
TimeSpan absolute = ts1.Duration();

// Comparison
bool isEqual = ts1.Equals(ts2);
int compare = ts1.CompareTo(ts2);
bool isLonger = ts1 > ts2;

// Zero and min/max
TimeSpan zero = TimeSpan.Zero;
TimeSpan min = TimeSpan.MinValue;
TimeSpan max = TimeSpan.MaxValue;
```

---

### 5. DateTime Formatting

#### Standard Format Strings

```csharp
DateTime dt = new DateTime(2024, 12, 17, 14, 30, 45);

Console.WriteLine(dt.ToString("d"));   // 12/17/2024 (short date)
Console.WriteLine(dt.ToString("D"));   // Tuesday, December 17, 2024 (long date)
Console.WriteLine(dt.ToString("t"));   // 2:30 PM (short time)
Console.WriteLine(dt.ToString("T"));   // 2:30:45 PM (long time)
Console.WriteLine(dt.ToString("f"));   // Tuesday, December 17, 2024 2:30 PM
Console.WriteLine(dt.ToString("F"));   // Tuesday, December 17, 2024 2:30:45 PM
Console.WriteLine(dt.ToString("g"));   // 12/17/2024 2:30 PM (general short)
Console.WriteLine(dt.ToString("G"));   // 12/17/2024 2:30:45 PM (general long)
Console.WriteLine(dt.ToString("M"));   // December 17 (month day)
Console.WriteLine(dt.ToString("Y"));   // December 2024 (year month)
Console.WriteLine(dt.ToString("o"));   // 2024-12-17T14:30:45.0000000 (round-trip)
Console.WriteLine(dt.ToString("R"));   // Tue, 17 Dec 2024 14:30:45 GMT (RFC1123)
Console.WriteLine(dt.ToString("s"));   // 2024-12-17T14:30:45 (sortable)
Console.WriteLine(dt.ToString("u"));   // 2024-12-17 14:30:45Z (universal)
```

#### Standard Format Strings Table

| Format | Name | Example Output |
|--------|------|----------------|
| **d** | Short date | 12/17/2024 |
| **D** | Long date | Tuesday, December 17, 2024 |
| **t** | Short time | 2:30 PM |
| **T** | Long time | 2:30:45 PM |
| **f** | Full (short time) | Tuesday, December 17, 2024 2:30 PM |
| **F** | Full (long time) | Tuesday, December 17, 2024 2:30:45 PM |
| **g** | General (short) | 12/17/2024 2:30 PM |
| **G** | General (long) | 12/17/2024 2:30:45 PM |
| **M** or **m** | Month day | December 17 |
| **Y** or **y** | Year month | December 2024 |
| **o** or **O** | Round-trip (ISO 8601) | 2024-12-17T14:30:45.0000000 |
| **R** or **r** | RFC1123 | Tue, 17 Dec 2024 14:30:45 GMT |
| **s** | Sortable | 2024-12-17T14:30:45 |
| **u** | Universal sortable | 2024-12-17 14:30:45Z |

#### Custom Format Strings

```csharp
DateTime dt = new DateTime(2024, 12, 17, 14, 30, 45, 123);

// Year
dt.ToString("yyyy")  // "2024" (4-digit)
dt.ToString("yy")    // "24" (2-digit)

// Month
dt.ToString("MM")    // "12" (2-digit)
dt.ToString("M")     // "12" (1 or 2 digit)
dt.ToString("MMM")   // "Dec" (abbreviated)
dt.ToString("MMMM")  // "December" (full name)

// Day
dt.ToString("dd")    // "17" (2-digit)
dt.ToString("d")     // "17" (1 or 2 digit)
dt.ToString("ddd")   // "Tue" (abbreviated)
dt.ToString("dddd")  // "Tuesday" (full name)

// Hour (24-hour)
dt.ToString("HH")    // "14" (2-digit)
dt.ToString("H")     // "14" (1 or 2 digit)

// Hour (12-hour)
dt.ToString("hh")    // "02" (2-digit)
dt.ToString("h")     // "2" (1 or 2 digit)

// Minute
dt.ToString("mm")    // "30" (2-digit)
dt.ToString("m")     // "30" (1 or 2 digit)

// Second
dt.ToString("ss")    // "45" (2-digit)
dt.ToString("s")     // "45" (1 or 2 digit)

// Milliseconds
dt.ToString("fff")   // "123" (3 digits)
dt.ToString("ff")    // "12" (2 digits)
dt.ToString("f")     // "1" (1 digit)

// AM/PM
dt.ToString("tt")    // "PM"
dt.ToString("t")     // "P"

// Combined custom formats
dt.ToString("yyyy-MM-dd")              // "2024-12-17"
dt.ToString("dd/MM/yyyy")              // "17/12/2024"
dt.ToString("MMMM dd, yyyy")           // "December 17, 2024"
dt.ToString("yyyy-MM-dd HH:mm:ss")     // "2024-12-17 14:30:45"
dt.ToString("hh:mm tt")                // "02:30 PM"
dt.ToString("dddd, MMMM dd, yyyy")     // "Tuesday, December 17, 2024"
```

#### Common Scenarios

```csharp
DateTime dt = DateTime.Now;

// ISO 8601 (for APIs)
string iso = dt.ToString("o");  // 2024-12-17T14:30:45.0000000

// File name safe
string fileName = dt.ToString("yyyyMMdd_HHmmss");  // 20241217_143045

// Database (SQL Server)
string sql = dt.ToString("yyyy-MM-dd HH:mm:ss");

// User-friendly
string friendly = dt.ToString("MMMM dd, yyyy");  // December 17, 2024

// Time only
string timeOnly = dt.ToString("HH:mm");  // 14:30
```

---

## Part 2: Regular Expressions

### 6. Regex Class (System.Text.RegularExpressions)

**What are Regular Expressions?** Patterns for matching text.

**When to use Regex:**

- ✅ Validating input (email, phone, etc.)
- ✅ Finding patterns in text
- ✅ Replacing text based on patterns
- ✅ Splitting strings by complex patterns
- ❌ Simple string operations (use string methods)

```csharp
using System.Text.RegularExpressions;

// Basic usage
bool isMatch = Regex.IsMatch("hello123", @"\d+");  // true (contains digits)

// Create Regex object
Regex regex = new Regex(@"\d+");
bool match = regex.IsMatch("hello123");
```

---

### 7. Character Classes

```csharp
// Literal characters
Regex.IsMatch("abc", "abc")  // true

// Any character (.)
Regex.IsMatch("cat", "c.t")  // true (. matches 'a')
Regex.IsMatch("cut", "c.t")  // true (. matches 'u')

// Character class []
Regex.IsMatch("cat", "[cb]at")  // true (c or b)
Regex.IsMatch("bat", "[cb]at")  // true

// Range
Regex.IsMatch("5", "[0-9]")    // true (any digit)
Regex.IsMatch("c", "[a-z]")    // true (lowercase letter)
Regex.IsMatch("C", "[A-Z]")    // true (uppercase letter)

// Negated class [^]
Regex.IsMatch("5", "[^a-z]")   // true (NOT a lowercase letter)

// Predefined character classes
Regex.IsMatch("5", @"\d")      // true (digit) = [0-9]
Regex.IsMatch("a", @"\D")      // true (non-digit)
Regex.IsMatch("a", @"\w")      // true (word char) = [a-zA-Z0-9_]
Regex.IsMatch("@", @"\W")      // true (non-word)
Regex.IsMatch(" ", @"\s")      // true (whitespace)
Regex.IsMatch("a", @"\S")      // true (non-whitespace)
```

### Character Classes Reference

| Pattern | Matches | Example |
|---------|---------|---------|
| `.` | Any character (except newline) | `c.t` → "cat", "cut" |
| `[abc]` | Any of a, b, or c | `[cb]at` → "cat", "bat" |
| `[^abc]` | Not a, b, or c | `[^c]at` → "bat", "hat" |
| `[a-z]` | Any lowercase letter | `[a-z]+` → "hello" |
| `[A-Z]` | Any uppercase letter | `[A-Z]+` → "HELLO" |
| `[0-9]` | Any digit | `[0-9]+` → "123" |
| `\d` | Digit [0-9] | `\d+` → "123" |
| `\D` | Non-digit | `\D+` → "abc" |
| `\w` | Word character [a-zA-Z0-9_] | `\w+` → "hello_123" |
| `\W` | Non-word character | `\W+` → "@#$" |
| `\s` | Whitespace | `\s+` → "   " |
| `\S` | Non-whitespace | `\S+` → "hello" |

---

### 8. Quantifiers

```csharp
// * (zero or more)
Regex.IsMatch("", "a*")       // true (zero a's)
Regex.IsMatch("aaa", "a*")    // true (three a's)

// + (one or more)
Regex.IsMatch("", "a+")       // false (needs at least one)
Regex.IsMatch("aaa", "a+")    // true

// ? (zero or one)
Regex.IsMatch("color", "colou?r")   // true (u is optional)
Regex.IsMatch("colour", "colou?r")  // true

// {n} (exactly n)
Regex.IsMatch("aaa", "a{3}")    // true (exactly 3)
Regex.IsMatch("aa", "a{3}")     // false

// {n,} (n or more)
Regex.IsMatch("aaa", "a{2,}")   // true (2 or more)
Regex.IsMatch("aaaa", "a{2,}")  // true

// {n,m} (between n and m)
Regex.IsMatch("aa", "a{2,4}")    // true
Regex.IsMatch("aaa", "a{2,4}")   // true
Regex.IsMatch("aaaaa", "a{2,4}") // true (matches first 4)

// Greedy vs Lazy
string text = "<div>content</div>";
Regex.Match(text, "<.*>").Value   // "<div>content</div>" (greedy)
Regex.Match(text, "<.*?>").Value  // "<div>" (lazy)
```

### Quantifiers Reference

| Quantifier | Matches | Example |
|------------|---------|---------|
| `*` | 0 or more | `a*` → "", "a", "aaa" |
| `+` | 1 or more | `a+` → "a", "aaa" |
| `?` | 0 or 1 | `colou?r` → "color", "colour" |
| `{n}` | Exactly n | `a{3}` → "aaa" |
| `{n,}` | n or more | `a{2,}` → "aa", "aaa" |
| `{n,m}` | Between n and m | `a{2,4}` → "aa", "aaa", "aaaa" |
| `*?` | Lazy 0 or more | `<.*?>` → shortest match |
| `+?` | Lazy 1 or more | `.+?` → shortest match |
| `??` | Lazy 0 or 1 | `colou??r` → shortest match |

---

### 9. Anchors

```csharp
// ^ (start of string)
Regex.IsMatch("hello", "^h")      // true (starts with h)
Regex.IsMatch("hello", "^e")      // false

// $ (end of string)
Regex.IsMatch("hello", "o$")      // true (ends with o)
Regex.IsMatch("hello", "l$")      // false

// Combined
Regex.IsMatch("hello", "^hello$") // true (exact match)

// \b (word boundary)
Regex.IsMatch("cat dog", @"\bcat\b")   // true
Regex.IsMatch("catalog", @"\bcat\b")   // false (no boundary)

// \B (non-word boundary)
Regex.IsMatch("catalog", @"cat\B")     // true (cat followed by non-boundary)
```

---

### 10. Groups

```csharp
// Capturing groups ()
Match match = Regex.Match("John Doe", @"(\w+) (\w+)");
string firstName = match.Groups[1].Value;  // "John"
string lastName = match.Groups[2].Value;   // "Doe"

// Named groups (?<name>pattern)
Match match2 = Regex.Match("John Doe", @"(?<first>\w+) (?<last>\w+)");
string first = match2.Groups["first"].Value;  // "John"
string last = match2.Groups["last"].Value;    // "Doe"

// Non-capturing groups (?:pattern)
Regex.IsMatch("color", @"(?:col(?:o|ou)r)")  // Groups don't capture

// Backreferences
Regex.IsMatch("hello", @"(\w)\1")  // true (\1 refers to first group)
Regex.IsMatch("abc", @"(\w)\1")    // false (no repeated character)
```

---

### 11. Alternation

```csharp
// | (OR operator)
Regex.IsMatch("cat", "cat|dog")   // true
Regex.IsMatch("dog", "cat|dog")   // true
Regex.IsMatch("bird", "cat|dog")  // false

// With groups
Regex.IsMatch("color", "col(o|ou)r")    // true
Regex.IsMatch("colour", "col(o|ou)r")   // true
```

---

### 12. Regex Methods

```csharp
string text = "My phone is 555-1234 and my other is 555-5678";
string pattern = @"\d{3}-\d{4}";

// IsMatch - test if pattern matches
bool hasPhone = Regex.IsMatch(text, pattern);  // true

// Match - get first match
Match match = Regex.Match(text, pattern);
if (match.Success)
{
    Console.WriteLine(match.Value);  // "555-1234"
    Console.WriteLine(match.Index);  // 14 (position)
}

// Matches - get all matches
MatchCollection matches = Regex.Matches(text, pattern);
foreach (Match m in matches)
{
    Console.WriteLine(m.Value);  // "555-1234", "555-5678"
}

// Replace - replace matches
string replaced = Regex.Replace(text, pattern, "XXX-XXXX");
// "My phone is XXX-XXXX and my other is XXX-XXXX"

// Replace with function
string masked = Regex.Replace(text, pattern, m => 
{
    return "***-" + m.Value.Substring(4);
});

// Split - split by pattern
string csv = "a,b,c;d,e";
string[] parts = Regex.Split(csv, "[,;]");
// ["a", "b", "c", "d", "e"]

// Static vs Instance
// Static: Regex.IsMatch(...) - convenient
// Instance: new Regex(...).IsMatch(...) - better for reuse
```

---

### 13. RegexOptions Enum

```csharp
// IgnoreCase
bool match = Regex.IsMatch("Hello", "hello", RegexOptions.IgnoreCase);  // true

// Multiline (^ and $ match each line)
string multiline = "line1\nline2";
bool hasStart = Regex.IsMatch(multiline, "^line2", RegexOptions.Multiline);  // true

// Singleline (. matches newline)
bool matchAll = Regex.IsMatch("a\nb", "a.b", RegexOptions.Singleline);  // true

// IgnorePatternWhitespace (verbose mode)
var regex = new Regex(@"
    \d{3}   # area code
    -       # separator
    \d{4}   # number
", RegexOptions.IgnorePatternWhitespace);

// Compiled (better performance for repeated use)
var compiled = new Regex(@"\d+", RegexOptions.Compiled);

// Combine options
var options = RegexOptions.IgnoreCase | RegexOptions.Multiline;
```

---

### 14. Common Regex Patterns

```csharp
// Email (basic)
string email = @"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$";
bool isEmail = Regex.IsMatch("user@example.com", email);

// Phone (US)
string phone = @"^\d{3}-\d{3}-\d{4}$";
bool isPhone = Regex.IsMatch("555-123-4567", phone);

// Alternative phone format
string phone2 = @"^\(\d{3}\) \d{3}-\d{4}$";  // (555) 123-4567

// URL
string url = @"https?://[^\s]+";
bool isUrl = Regex.IsMatch("https://example.com", url);

// IP Address (basic)
string ip = @"^(\d{1,3}\.){3}\d{1,3}$";
bool isIp = Regex.IsMatch("192.168.1.1", ip);

// ZIP Code (US)
string zip = @"^\d{5}(-\d{4})?$";
bool isZip = Regex.IsMatch("12345", zip);  // or 12345-6789

// Credit Card (basic)
string card = @"^\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}$";

// Hex Color
string hex = @"^#[0-9A-Fa-f]{6}$";
bool isColor = Regex.IsMatch("#FF5733", hex);

// Username (alphanumeric, 3-16 chars)
string username = @"^[a-zA-Z0-9_]{3,16}$";

// Password strength (min 8 chars, 1 upper, 1 lower, 1 digit)
string password = @"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$";

// Date (YYYY-MM-DD)
string date = @"^\d{4}-\d{2}-\d{2}$";

// Time (HH:MM 24-hour)
string time = @"^([01]\d|2[0-3]):([0-5]\d)$";

// Social Security Number (US)
string ssn = @"^\d{3}-\d{2}-\d{4}$";

// MAC Address
string mac = @"^([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})$";
```

### Common Patterns Reference

| Pattern Type | Regex | Matches |
|-------------|-------|---------|
| **Email** | `^[\w.+-]+@[\w.-]+\.\w{2,}$` | user@example.com |
| **Phone (US)** | `^\d{3}-\d{3}-\d{4}$` | 555-123-4567 |
| **URL** | `https?://[^\s]+` | https://example.com |
| **IP Address** | `^(\d{1,3}\.){3}\d{1,3}$` | 192.168.1.1 |
| **ZIP Code** | `^\d{5}(-\d{4})?$` | 12345 or 12345-6789 |
| **Hex Color** | `^#[0-9A-Fa-f]{6}$` | #FF5733 |
| **Date (ISO)** | `^\d{4}-\d{2}-\d{2}$` | 2024-12-17 |

---

## Part 3: Math & Random

### 15. Math Class

```csharp
using System;

// Constants
double pi = Math.PI;      // 3.14159265358979
double e = Math.E;        // 2.71828182845905

// Trigonometry (radians)
double sin = Math.Sin(Math.PI / 2);      // 1.0
double cos = Math.Cos(Math.PI);          // -1.0
double tan = Math.Tan(Math.PI / 4);      // 1.0

// Inverse trig
double asin = Math.Asin(1);              // π/2
double acos = Math.Acos(0);              // π/2
double atan = Math.Atan(1);              // π/4
double atan2 = Math.Atan2(1, 1);         // π/4 (y, x)

// Power and root
double pow = Math.Pow(2, 3);             // 8.0 (2³)
double sqrt = Math.Sqrt(16);             // 4.0
double cbrt = Math.Cbrt(27);             // 3.0 (cube root)

// Rounding
double round = Math.Round(3.5);          // 4.0 (banker's rounding)
double floor = Math.Floor(3.9);          // 3.0 (round down)
double ceiling = Math.Ceiling(3.1);      // 4.0 (round up)
double truncate = Math.Truncate(3.9);    // 3.0 (remove decimal)

// Absolute value
int abs = Math.Abs(-5);                  // 5
double absD = Math.Abs(-3.14);           // 3.14

// Min/Max
int min = Math.Min(5, 10);               // 5
int max = Math.Max(5, 10);               // 10

// Sign (-1, 0, or 1)
int sign = Math.Sign(-5);                // -1
int sign2 = Math.Sign(0);                // 0
int sign3 = Math.Sign(5);                // 1

// Logarithms
double log = Math.Log(Math.E);           // 1.0 (natural log)
double log10 = Math.Log10(100);          // 2.0 (base 10)
double log2 = Math.Log2(8);              // 3.0 (base 2)

// Exponential
double exp = Math.Exp(1);                // e¹ = 2.718...

// Clamp (limit to range)
int clamped = Math.Clamp(15, 0, 10);     // 10 (max limit)
int clamped2 = Math.Clamp(-5, 0, 10);    // 0 (min limit)
int clamped3 = Math.Clamp(5, 0, 10);     // 5 (within range)
```

### Math Methods Reference

| Method | Description | Example |
|--------|-------------|---------|
| `Abs(x)` | Absolute value | `Math.Abs(-5)` → 5 |
| `Sqrt(x)` | Square root | `Math.Sqrt(16)` → 4 |
| `Pow(x, y)` | x to power y | `Math.Pow(2, 3)` → 8 |
| `Round(x)` | Round to nearest | `Math.Round(3.5)` → 4 |
| `Floor(x)` | Round down | `Math.Floor(3.9)` → 3 |
| `Ceiling(x)` | Round up | `Math.Ceiling(3.1)` → 4 |
| `Min(x, y)` | Minimum | `Math.Min(5, 10)` → 5 |
| `Max(x, y)` | Maximum | `Math.Max(5, 10)` → 10 |
| `Clamp(x, min, max)` | Limit to range | `Math.Clamp(15, 0, 10)` → 10 |

---

### 16. Rounding Comparison

```csharp
// MidpointRounding options
double value = 2.5;

// ToEven (banker's rounding) - default
Math.Round(2.5, MidpointRounding.ToEven);  // 2
Math.Round(3.5, MidpointRounding.ToEven);  // 4

// AwayFromZero
Math.Round(2.5, MidpointRounding.AwayFromZero);  // 3
Math.Round(3.5, MidpointRounding.AwayFromZero);  // 4
Math.Round(-2.5, MidpointRounding.AwayFromZero); // -3

// ToZero
Math.Round(2.5, MidpointRounding.ToZero);   // 2
Math.Round(-2.5, MidpointRounding.ToZero);  // -2

// ToNegativeInfinity (like Floor)
Math.Round(2.5, MidpointRounding.ToNegativeInfinity);  // 2
Math.Round(-2.5, MidpointRounding.ToNegativeInfinity); // -3

// ToPositiveInfinity (like Ceiling)
Math.Round(2.5, MidpointRounding.ToPositiveInfinity);  // 3
Math.Round(-2.5, MidpointRounding.ToPositiveInfinity); // -2
```

### Rounding Behavior Table

| Value | ToEven | AwayFromZero | ToZero | Floor | Ceiling |
|-------|--------|--------------|--------|-------|---------|
| 2.5 | 2 | 3 | 2 | 2 | 3 |
| 3.5 | 4 | 4 | 3 | 3 | 4 |
| -2.5 | -2 | -3 | -2 | -3 | -2 |
| -3.5 | -4 | -4 | -3 | -4 | -3 |

---

### 17. Random Class

```csharp
// Create Random
Random random = new Random();            // Time-based seed
Random seeded = new Random(12345);       // Fixed seed (repeatable)

// Random integer (non-negative)
int value1 = random.Next();              // 0 to int.MaxValue

// Random integer (0 to max-1)
int value2 = random.Next(10);            // 0 to 9

// Random integer (min to max-1)
int value3 = random.Next(1, 7);          // 1 to 6 (dice roll)

// Random double (0.0 to 1.0)
double value4 = random.NextDouble();     // 0.0 to 0.999...

// Random bytes
byte[] buffer = new byte[10];
random.NextBytes(buffer);

// Random.Shared (.NET 6.0+) - thread-safe
int shared = Random.Shared.Next(1, 100);
```

**Thread Safety:**

- `Random` - NOT thread-safe (use separate instance per thread)
- `Random.Shared` (.NET 6.0+) - Thread-safe

**Common Patterns:**
```csharp
// Random boolean
bool randomBool = random.Next(2) == 0;

// Random from array
string[] names = { "Alice", "Bob", "Charlie" };
string randomName = names[random.Next(names.Length)];

// Random double in range
double randomDouble = random.NextDouble() * (max - min) + min;

// Random color
Color randomColor = Color.FromArgb(
    random.Next(256),
    random.Next(256),
    random.Next(256));
```

---

### 18. Guid Struct

```csharp
// Create new GUID (globally unique)
Guid guid1 = Guid.NewGuid();
// Example: 3f2504e0-4f89-11d3-9a0c-0305e82c3301

// Empty GUID
Guid empty = Guid.Empty;  // 00000000-0000-0000-0000-000000000000

// Parse
Guid guid2 = Guid.Parse("3f2504e0-4f89-11d3-9a0c-0305e82c3301");

// Try parse
if (Guid.TryParse("3f2504e0-4f89-11d3-9a0c-0305e82c3301", out Guid result))
{
    Console.WriteLine(result);
}

// String formats
Guid guid = Guid.NewGuid();
string n = guid.ToString("N");  // 32 digits: 00000000000000000000000000000000
string d = guid.ToString("D");  // Hyphens: 00000000-0000-0000-0000-000000000000
string b = guid.ToString("B");  // Braces: {00000000-0000-0000-0000-000000000000}
string p = guid.ToString("P");  // Parentheses: (00000000-0000-0000-0000-000000000000)
string x = guid.ToString("X");  // Hex: {0x00000000,0x0000,0x0000,{0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00}}

// Default (same as D)
string str = guid.ToString();

// Common use case: database primary keys
```

---

## Part 4: String Utilities

### 19. String.Format() and Composite Formatting

```csharp
// Positional parameters
string result = String.Format("Hello, {0}!", "World");
// "Hello, World!"

string result2 = String.Format("{0} + {1} = {2}", 2, 3, 5);
// "2 + 3 = 5"

// Format specifiers
String.Format("{0:N2}", 1234.567);     // "1,234.57" (number, 2 decimals)
String.Format("{0:C}", 123.45);        // "$123.45" (currency)
String.Format("{0:P}", 0.123);         // "12.30%" (percent)
String.Format("{0:D5}", 42);           // "00042" (decimal, 5 digits)
String.Format("{0:X}", 255);           // "FF" (hexadecimal)
String.Format("{0:E2}", 1234.56);      // "1.23E+003" (scientific)

// Alignment
String.Format("|{0,10}|", "right");    // "|     right|" (right-aligned)
String.Format("|{0,-10}|", "left");    // "|left      |" (left-aligned)

// Combine alignment and format
String.Format("{0,10:C}", 123.45);     // "   $123.45" (right-aligned currency)

// DateTime formatting
DateTime dt = new DateTime(2024, 12, 17);
String.Format("{0:yyyy-MM-dd}", dt);   // "2024-12-17"
String.Format("{0:MMMM dd, yyyy}", dt);// "December 17, 2024"

// String interpolation (modern alternative)
string name = "World";
string result3 = $"Hello, {name}!";    // Same as Format
```

---

### 20. StringBuilder (Brief Reference)

**When to use:**

- ✅ Building strings in loops
- ✅ Many concatenations (>5)
- ✅ Dynamic string construction
- ❌ Few concatenations (string + is fine)

```csharp
using System.Text;

// Create
StringBuilder sb = new StringBuilder();
StringBuilder sb2 = new StringBuilder("Initial");
StringBuilder sb3 = new StringBuilder(100);  // Initial capacity

// Append
sb.Append("Hello");
sb.Append(" ");
sb.Append("World");
sb.AppendLine();  // Append with newline
sb.AppendLine("Next line");

// Insert
sb.Insert(0, "Start: ");

// Remove
sb.Remove(0, 7);  // Remove 7 chars from position 0

// Replace
sb.Replace("World", "Universe");

// Clear
sb.Clear();

// Get result
string result = sb.ToString();

// Performance: StringBuilder vs String
// String concatenation in loop
string s = "";
for (int i = 0; i < 1000; i++)
{
    s += i;  // ❌ Creates 1000 string objects
}

// StringBuilder in loop
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);  // ✅ Efficient
}
string result = sb.ToString();
```

---

### 21. String Comparison

```csharp
string s1 = "Hello";
string s2 = "hello";

// Ordinal (byte-by-byte comparison)
bool equal1 = String.Equals(s1, s2, StringComparison.Ordinal);  // false

// OrdinalIgnoreCase (case-insensitive)
bool equal2 = String.Equals(s1, s2, StringComparison.OrdinalIgnoreCase);  // true

// CurrentCulture (culture-specific)
bool equal3 = String.Equals(s1, s2, StringComparison.CurrentCulture);  // false

// CurrentCultureIgnoreCase
bool equal4 = String.Equals(s1, s2, StringComparison.CurrentCultureIgnoreCase);  // true

// InvariantCulture (culture-independent)
bool equal5 = String.Equals(s1, s2, StringComparison.InvariantCulture);

// InvariantCultureIgnoreCase
bool equal6 = String.Equals(s1, s2, StringComparison.InvariantCultureIgnoreCase);

// With methods
bool startsWith = s1.StartsWith("He", StringComparison.OrdinalIgnoreCase);
bool endsWith = s1.EndsWith("LO", StringComparison.OrdinalIgnoreCase);
bool contains = s1.Contains("ell", StringComparison.OrdinalIgnoreCase);

// Sorting with comparison
List<string> names = new List<string> { "alice", "Bob", "CHARLIE" };
names.Sort(StringComparer.OrdinalIgnoreCase);
```

### StringComparison Options

| Option | Case-Sensitive | Culture-Specific | Use When |
|--------|----------------|------------------|----------|
| **Ordinal** | Yes | No | File paths, IDs, exact matching |
| **OrdinalIgnoreCase** | No | No | File names, configuration keys |
| **CurrentCulture** | Yes | Yes | User-facing text |
| **CurrentCultureIgnoreCase** | No | Yes | User-facing search |
| **InvariantCulture** | Yes | No | Persistent storage |
| **InvariantCultureIgnoreCase** | No | No | Case-insensitive storage keys |

**Best Practice:**

- Use **Ordinal** or **OrdinalIgnoreCase** for non-linguistic comparisons
- Use **CurrentCulture** for user-facing text

---

## Quick Reference Summary

### DateTime

- `DateTime.Now` / `DateTime.UtcNow` / `DateTime.Today`
- Add/subtract: `AddDays()`, `AddMonths()`, `Subtract()`
- Format: `ToString("yyyy-MM-dd")` or standard formats (d, D, t, T, etc.)
- Use `DateTimeOffset` for time zones
- Use `DateOnly` / `TimeOnly` (.NET 6+) for date/time only

### TimeSpan

- Create: `TimeSpan.FromDays()`, `FromHours()`, etc.
- Properties: `Days`, `Hours`, `TotalDays`, `TotalHours`
- Operations: `+`, `-`, `*`, `/`

### Regex

- Test: `Regex.IsMatch(text, pattern)`
- Find: `Regex.Match(text, pattern)`
- Replace: `Regex.Replace(text, pattern, replacement)`
- Patterns: `\d` (digit), `\w` (word), `\s` (space), `[a-z]` (range)
- Quantifiers: `*` (0+), `+` (1+), `?` (0-1), `{n}` (exactly n)

### Math

- Rounding: `Round()`, `Floor()`, `Ceiling()`, `Truncate()`
- Power: `Pow()`, `Sqrt()`, `Cbrt()`
- Min/Max: `Min()`, `Max()`, `Clamp()`

### Random

- `Next()` - random int
- `NextDouble()` - random double (0.0-1.0)
- `Random.Shared` (.NET 6+) - thread-safe

### String Utilities

- Format: `String.Format("{0:N2}", value)`
- StringBuilder: Use for many concatenations
- Comparison: Use `StringComparison.OrdinalIgnoreCase` for case-insensitive

---

**Guide Complete!** These utilities are essential for everyday C# programming! 📘