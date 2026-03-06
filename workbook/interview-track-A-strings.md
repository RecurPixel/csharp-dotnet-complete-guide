# 💼 INTERVIEW TRACK - TRACK A: STRING ALGORITHMS
## 12 Problems with Guidance Only (NO Solutions)

**Purpose**: Master string manipulation for technical interviews  
**Difficulty**: ⭐ Easy → ⭐⭐⭐⭐ Hard  
**Time per problem**: 20-45 minutes  

---

## Problem 151: Reverse String (3 Methods) ⭐⭐

**Problem Statement:**

Write a function that reverses a string. Implement THREE different approaches:
1. Using iteration
2. Using recursion
3. Using built-in methods

**Examples:**
```
Input: "hello"
Output: "olleh"

Input: "C#"
Output: "#C"

Input: "a"
Output: "a"

Input: ""
Output: ""
```

**Constraints:**
- 0 ≤ string.length ≤ 10⁴
- String contains ASCII characters

---

**Approach 1: Two Pointers (Iteration)**

**Concept:**
- Use two pointers: one at start, one at end
- Swap characters and move pointers toward center
- Convert string to char array first (strings are immutable)

**Complexity:**
- Time: O(n)
- Space: O(n) - for char array

**Hints:**
```csharp
char[] chars = str.ToCharArray();
int left = 0, right = chars.Length - 1;
// Swap chars[left] and chars[right]
// Move pointers
return new string(chars);
```

---

**Approach 2: Recursion**

**Concept:**
- Base case: empty or single character
- Recursive case: first char + reverse(rest of string)

**Complexity:**
- Time: O(n)
- Space: O(n) - recursion stack + string concatenation

**Hints:**
```csharp
if (str.Length <= 1) return str;
return str[str.Length - 1] + Reverse(str.Substring(0, str.Length - 1));
```

---

**Approach 3: Built-in Methods**

**Concept:**
- Use Array.Reverse() or LINQ

**Hints:**
```csharp
// Method 1: Array.Reverse
char[] arr = str.ToCharArray();
Array.Reverse(arr);

// Method 2: LINQ
new string(str.Reverse().ToArray());
```

---

**Test Cases:**
```csharp
"hello" → "olleh"
"C#" → "#C"
"a" → "a"
"" → ""
"racecar" → "racecar" (palindrome)
"12345" → "54321"
```

**Interview Tips:**
- Show you know multiple approaches
- Mention string immutability
- Discuss which method is most efficient
- Production code: use built-in methods
- Interview: show you can implement from scratch

---

## Problem 152: Check Anagrams ⭐⭐

**Problem Statement:**

Given two strings `s` and `t`, return true if `t` is an anagram of `s`, and false otherwise.

An anagram is a word formed by rearranging the letters of another word, using all original letters exactly once.

**Examples:**
```
Input: s = "anagram", t = "nagaram"
Output: true

Input: s = "rat", t = "car"
Output: false

Input: s = "listen", t = "silent"
Output: true
```

**Constraints:**
- 1 ≤ s.length, t.length ≤ 5 × 10⁴
- s and t consist of lowercase English letters

---

**Approach 1: Sorting**

**Concept:**
- Sort both strings
- Compare if sorted versions are equal

**Complexity:**
- Time: O(n log n)
- Space: O(1) or O(n) depending on sorting

**Hints:**
```csharp
// Convert to char arrays, sort, compare
char[] sChars = s.ToCharArray();
char[] tChars = t.ToCharArray();
Array.Sort(sChars);
Array.Sort(tChars);
// Compare arrays
```

---

**Approach 2: Frequency Map (Optimal)**

**Concept:**
- Count frequency of each character
- Both strings should have same character frequencies

**Complexity:**
- Time: O(n)
- Space: O(1) - at most 26 letters

**Hints:**
```csharp
// Use Dictionary<char, int> or int[26]
var freq = new Dictionary<char, int>();
// Count chars in s (add)
// Count chars in t (subtract)
// Check if all frequencies are 0
```

---

**Approach 3: Single Pass (Optimized)**

**Concept:**
- Use array of size 26 for lowercase letters
- Increment for first string, decrement for second
- Check if all values are zero

**Hints:**
```csharp
int[] counts = new int[26];
// For each char in s: counts[c - 'a']++
// For each char in t: counts[c - 'a']--
// Check if all counts[i] == 0
```

---

**Test Cases:**
```csharp
("anagram", "nagaram") → true
("rat", "car") → false
("listen", "silent") → true
("abc", "abcd") → false (different lengths)
("a", "a") → true
("ab", "ba") → true
```

**Common Mistakes:**
- Not checking lengths first (quick optimization)
- Case sensitivity (problem says lowercase, but ask!)
- Unicode characters (problem says English letters)

**Interview Tips:**
- Always check lengths first (O(1) optimization)
- Mention both approaches
- Sorting is simpler code but slower
- Hash map is optimal

---

## Problem 153: First Non-Repeating Character ⭐⭐

**Problem Statement:**

Given a string `s`, find the first non-repeating character and return its index. If it doesn't exist, return -1.

**Examples:**
```
Input: s = "leetcode"
Output: 0 (character 'l')

Input: s = "loveleetcode"
Output: 2 (character 'v')

Input: s = "aabb"
Output: -1
```

**Constraints:**
- 1 ≤ s.length ≤ 10⁵
- s consists of lowercase English letters

---

**Approach 1: Brute Force**

**Concept:**
- For each character, check if it appears elsewhere
- Return first character that doesn't repeat

**Complexity:**
- Time: O(n²)
- Space: O(1)

**Not recommended for interview!**

---

**Approach 2: Two Pass with Dictionary (Optimal)**

**Concept:**
- First pass: count frequency of each character
- Second pass: find first character with frequency 1

**Complexity:**
- Time: O(n)
- Space: O(1) - at most 26 letters

**Hints:**
```csharp
// First pass: build frequency map
var freq = new Dictionary<char, int>();
foreach (char c in s)
{
    // Count frequencies
}

// Second pass: find first with freq == 1
for (int i = 0; i < s.Length; i++)
{
    if (freq[s[i]] == 1)
        return i;
}
return -1;
```

---

**Approach 3: Array Instead of Dictionary**

**Concept:**
- Use int[26] for lowercase letters
- Faster than Dictionary for this use case

**Hints:**
```csharp
int[] counts = new int[26];
// First pass: count
// Second pass: find first with count == 1
```

---

**Test Cases:**
```csharp
"leetcode" → 0
"loveleetcode" → 2
"aabb" → -1
"z" → 0
"aabbccddeeffgghhiiz" → 18
```

**Interview Tips:**
- Explain why two passes are needed
- Mention array vs Dictionary trade-off
- Array is faster for known small character set
- Dictionary is more flexible for Unicode

---

## Problem 154: Longest Substring Without Repeating Characters ⭐⭐⭐

**Problem Statement:**

Given a string `s`, find the length of the longest substring without repeating characters.

**Examples:**
```
Input: s = "abcabcbb"
Output: 3
Explanation: "abc" is the longest substring

Input: s = "bbbbb"
Output: 1
Explanation: "b"

Input: s = "pwwkew"
Output: 3
Explanation: "wke"
```

**Constraints:**
- 0 ≤ s.length ≤ 5 × 10⁴
- s consists of English letters, digits, symbols, and spaces

---

**Approach 1: Brute Force**

**Concept:**
- Check all possible substrings
- For each, verify if all characters are unique

**Complexity:**
- Time: O(n³)
- Space: O(min(n, m)) where m is character set size

**Too slow!**

---

**Approach 2: Sliding Window with HashSet (Optimal)**

**Key Insight:**
- Use two pointers (left and right)
- Expand window by moving right
- Contract window when duplicate found
- Track maximum length seen

**Concept:**
- HashSet stores characters in current window
- When duplicate found, remove from left until no duplicate
- Update max length at each step

**Complexity:**
- Time: O(n) - each character visited at most twice
- Space: O(min(n, m)) - HashSet size

**Hints:**
```csharp
var seen = new HashSet<char>();
int left = 0, maxLength = 0;

for (int right = 0; right < s.Length; right++)
{
    // While s[right] is in seen:
    //   Remove s[left] from seen
    //   Move left++
    
    // Add s[right] to seen
    // Update maxLength
}
```

---

**Approach 3: Sliding Window with Dictionary (Optimized)**

**Concept:**
- Store character → last seen index
- When duplicate found, jump left pointer directly

**Complexity:**
- Time: O(n) - single pass
- Space: O(min(n, m))

**Hints:**
```csharp
var lastSeen = new Dictionary<char, int>();
int left = 0, maxLength = 0;

for (int right = 0; right < s.Length; right++)
{
    if (lastSeen.ContainsKey(s[right]))
    {
        // Move left to the right of last occurrence
        left = Math.Max(left, lastSeen[s[right]] + 1);
    }
    
    // Update last seen
    lastSeen[s[right]] = right;
    
    // Update max length
    maxLength = Math.Max(maxLength, right - left + 1);
}
```

---

**Test Cases:**
```csharp
"abcabcbb" → 3 ("abc")
"bbbbb" → 1 ("b")
"pwwkew" → 3 ("wke" or "kew")
"" → 0
" " → 1
"au" → 2
"dvdf" → 3 ("vdf")
"abba" → 2 ("ab" or "ba")
```

**Common Mistakes:**
- Not handling empty string
- Off-by-one errors with left pointer
- Forgetting to update maxLength
- Not using Math.Max when moving left pointer

**Interview Tips:**
- Start by explaining brute force (shows understanding)
- Draw the sliding window movement
- Explain why each character is visited at most twice
- Mention the optimization with Dictionary

---

## Problem 155: String Compression (aaabb → a3b2) ⭐⭐

**Problem Statement:**

Implement basic string compression using the counts of repeated characters.

For example: "aabcccccaaa" becomes "a2b1c5a3"

If the compressed string is not smaller than the original, return the original string.

**Examples:**
```
Input: "aabcccccaaa"
Output: "a2b1c5a3"

Input: "abcdef"
Output: "abcdef" (no compression)

Input: "aabbcc"
Output: "aabbcc" (compressed: "a2b2c2" is longer)

Input: "aaaa"
Output: "a4"
```

**Constraints:**
- 1 ≤ s.length ≤ 10⁴
- s consists of lowercase English letters

---

**Approach: Single Pass with StringBuilder**

**Concept:**
- Iterate through string counting consecutive characters
- When character changes, append count to result
- Compare final length with original

**Complexity:**
- Time: O(n)
- Space: O(n) - StringBuilder

**Hints:**
```csharp
var compressed = new StringBuilder();
int count = 1;

for (int i = 0; i < s.Length; i++)
{
    // If next char is same: count++
    // Else: append current char and count
    //       reset count to 1
}

// Don't forget the last character group!

// Return shorter of original or compressed
return compressed.Length < s.Length ? 
    compressed.ToString() : s;
```

---

**Test Cases:**
```csharp
"aabcccccaaa" → "a2b1c5a3"
"abcdef" → "abcdef"
"aabbcc" → "aabbcc"
"aaaa" → "a4"
"a" → "a"
"aabbccddee" → "aabbccddee"
```

**Common Mistakes:**
- Forgetting last character group
- Not comparing lengths before returning
- Using string concatenation instead of StringBuilder (slow!)

**Interview Tips:**
- Mention StringBuilder for performance
- Handle edge case: single character
- Ask: "Should single occurrences show '1'?" (Yes in this version)

---

## Problem 156: Longest Palindromic Substring ⭐⭐⭐⭐

**Problem Statement:**

Given a string `s`, return the longest palindromic substring in `s`.

**Examples:**
```
Input: s = "babad"
Output: "bab" (or "aba")

Input: s = "cbbd"
Output: "bb"

Input: s = "a"
Output: "a"

Input: s = "ac"
Output: "a" (or "c")
```

**Constraints:**
- 1 ≤ s.length ≤ 1000
- s consists of lowercase English letters

---

**Approach 1: Brute Force**

**Concept:**
- Check all possible substrings
- For each, check if palindrome

**Complexity:**
- Time: O(n³)
- Space: O(1)

**Too slow!**

---

**Approach 2: Expand Around Center (Optimal for this problem)**

**Key Insight:**
- Palindromes mirror around center
- Center can be single character (odd length) or between two characters (even length)
- Expand outward from each possible center

**Concept:**
- For each position, try expanding as center
- Track longest palindrome found

**Complexity:**
- Time: O(n²) - n centers × n expansion
- Space: O(1)

**Hints:**
```csharp
string ExpandAroundCenter(string s, int left, int right)
{
    // While left >= 0 and right < length and s[left] == s[right]:
    //   Expand outward
    // Return substring
}

// For each position:
//   Check odd length palindrome (single center)
//   Check even length palindrome (two centers)
//   Track longest
```

---

**Approach 3: Dynamic Programming**

**Concept:**
- dp[i][j] = true if substring from i to j is palindrome
- dp[i][j] = (s[i] == s[j]) && dp[i+1][j-1]

**Complexity:**
- Time: O(n²)
- Space: O(n²)

**Hints:**
```csharp
bool[,] dp = new bool[n, n];
// All single characters are palindromes
// Check all length-2 substrings
// Check all length-3 substrings
// ... up to length n
```

---

**Test Cases:**
```csharp
"babad" → "bab" or "aba"
"cbbd" → "bb"
"a" → "a"
"ac" → "a" or "c"
"racecar" → "racecar"
"noon" → "noon"
"abacabad" → "abacaba"
```

**Common Mistakes:**
- Forgetting even-length palindromes
- Off-by-one errors in expansion
- Not handling single character

**Interview Tips:**
- Explain expand-around-center approach (most intuitive)
- Mention DP exists but more complex
- Draw example of expansion
- Discuss time-space trade-offs

---

## Problem 157: Valid Parentheses ⭐⭐

**Problem Statement:**

Given a string `s` containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

Valid means:
- Open brackets must be closed by the same type
- Open brackets must be closed in the correct order
- Every close bracket has a corresponding open bracket

**Examples:**
```
Input: s = "()"
Output: true

Input: s = "()[]{}"
Output: true

Input: s = "(]"
Output: false

Input: s = "([)]"
Output: false

Input: s = "{[]}"
Output: true
```

**Constraints:**
- 1 ≤ s.length ≤ 10⁴
- s consists of parentheses only '()[]{}'

---

**Approach: Stack**

**Key Insight:**
- Most recent open bracket must match next close bracket
- This is LIFO (Last In, First Out) → Stack!

**Concept:**
- Push open brackets onto stack
- For close bracket: check if matches top of stack
- At end, stack should be empty

**Complexity:**
- Time: O(n)
- Space: O(n) - stack size

**Hints:**
```csharp
var stack = new Stack<char>();

foreach (char c in s)
{
    // If opening bracket: push
    if (c == '(' || c == '{' || c == '[')
    {
        stack.Push(c);
    }
    // If closing bracket: check match
    else
    {
        if (stack.Count == 0) return false;
        
        char top = stack.Pop();
        // Check if top matches c
        if ((c == ')' && top != '(') ||
            (c == '}' && top != '{') ||
            (c == ']' && top != '['))
        {
            return false;
        }
    }
}

return stack.Count == 0;
```

---

**Optimization: Use Dictionary for Matching**

```csharp
var pairs = new Dictionary<char, char>
{
    {')', '('},
    {'}', '{'},
    {']', '['}
};
```

---

**Test Cases:**
```csharp
"()" → true
"()[]{}" → true
"(]" → false
"([)]" → false
"{[]}" → true
"(((" → false
")))" → false
"" → true (edge case)
"{[()]}" → true
```

**Common Mistakes:**
- Not checking if stack is empty before popping
- Not checking if stack is empty at the end
- Forgetting edge case: empty string

**Interview Tips:**
- Stack is THE data structure for matching problems
- Explain LIFO property
- Walk through example visually
- Mention variations: min stack, next greater element, etc.

---

## Problem 158: String Permutations ⭐⭐⭐

**Problem Statement:**

Given a string `s`, return all possible permutations of its characters.

**Examples:**
```
Input: s = "abc"
Output: ["abc", "acb", "bac", "bca", "cab", "cba"]

Input: s = "ab"
Output: ["ab", "ba"]

Input: s = "a"
Output: ["a"]
```

**Constraints:**
- 1 ≤ s.length ≤ 8
- s consists of unique lowercase English letters

---

**Approach: Backtracking**

**Key Insight:**
- For each position, try every remaining character
- Recurse for rest of positions
- Backtrack when done

**Concept:**
- Fix first character, permute rest
- Swap characters to try different first characters
- Recursively build permutations

**Complexity:**
- Time: O(n × n!) - n! permutations, n to build each
- Space: O(n!) - storing all permutations

**Hints:**
```csharp
void Backtrack(List<string> result, char[] chars, int start)
{
    // Base case: reached end
    if (start == chars.Length)
    {
        result.Add(new string(chars));
        return;
    }
    
    // Try each character at current position
    for (int i = start; i < chars.Length; i++)
    {
        // Swap to try this character
        Swap(chars, start, i);
        
        // Recurse
        Backtrack(result, chars, start + 1);
        
        // Backtrack (undo swap)
        Swap(chars, start, i);
    }
}
```

---

**Test Cases:**
```csharp
"abc" → 6 permutations
"ab" → 2 permutations
"a" → 1 permutation
"abcd" → 24 permutations
```

**Interview Tips:**
- Draw the recursion tree
- Explain backtracking concept
- Mention time complexity (factorial!)
- Discuss duplicate handling (if chars not unique)

---

## Problem 159: String Rotation Check ⭐⭐

**Problem Statement:**

Check if one string is a rotation of another.

Example: "waterbottle" is a rotation of "erbottlewat"

**Examples:**
```
Input: s1 = "waterbottle", s2 = "erbottlewat"
Output: true

Input: s1 = "hello", s2 = "llohe"
Output: true

Input: s1 = "hello", s2 = "world"
Output: false
```

---

**Approach 1: Brute Force**

**Concept:**
- Try all possible rotations of s1
- Check if any matches s2

**Complexity:**
- Time: O(n²)
- Space: O(n)

---

**Approach 2: Clever Trick (Optimal)**

**Key Insight:**
- If s2 is rotation of s1, then s2 is substring of s1+s1!
- Example: "waterbottle" + "waterbottle" = "waterbottlewaterbottle"
- "erbottlewat" is substring of above!

**Concept:**
- Check if lengths are equal (must be for rotation)
- Check if s2 is substring of s1 + s1

**Complexity:**
- Time: O(n)
- Space: O(n)

**Hints:**
```csharp
if (s1.Length != s2.Length) return false;
string doubled = s1 + s1;
return doubled.Contains(s2);
```

---

**Test Cases:**
```csharp
("waterbottle", "erbottlewat") → true
("hello", "llohe") → true
("hello", "world") → false
("abc", "bca") → true
("abc", "acb") → false (not rotation, different order)
```

**Interview Tips:**
- Start with brute force to show understanding
- Then present the clever trick
- Explain why it works with example
- Mention this is a common interview trick

---

## Problem 160: Longest Common Prefix ⭐⭐

**Problem Statement:**

Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string "".

**Examples:**
```
Input: strs = ["flower", "flow", "flight"]
Output: "fl"

Input: strs = ["dog", "racecar", "car"]
Output: ""

Input: strs = ["interspecies", "interstellar", "interstate"]
Output: "inters"
```

**Constraints:**
- 1 ≤ strs.length ≤ 200
- 0 ≤ strs[i].length ≤ 200
- strs[i] consists of lowercase English letters

---

**Approach 1: Vertical Scanning**

**Concept:**
- Compare characters at same position across all strings
- Stop when mismatch found or any string ends

**Complexity:**
- Time: O(S) where S is sum of all characters
- Space: O(1)

**Hints:**
```csharp
if (strs.Length == 0) return "";

for (int i = 0; i < strs[0].Length; i++)
{
    char c = strs[0][i];
    
    // Check this character in all strings
    for (int j = 1; j < strs.Length; j++)
    {
        // If reached end or mismatch
        if (i >= strs[j].Length || strs[j][i] != c)
        {
            return strs[0].Substring(0, i);
        }
    }
}

return strs[0]; // First string is the prefix
```

---

**Approach 2: Horizontal Scanning**

**Concept:**
- Start with first string as prefix
- For each subsequent string, reduce prefix until it matches

**Hints:**
```csharp
string prefix = strs[0];

for (int i = 1; i < strs.Length; i++)
{
    while (!strs[i].StartsWith(prefix))
    {
        prefix = prefix.Substring(0, prefix.Length - 1);
        if (prefix == "") return "";
    }
}

return prefix;
```

---

**Test Cases:**
```csharp
["flower", "flow", "flight"] → "fl"
["dog", "racecar", "car"] → ""
["interspecies", "interstellar", "interstate"] → "inters"
["a"] → "a"
["", "b"] → ""
["abc", "abc", "abc"] → "abc"
```

**Interview Tips:**
- Both approaches are valid
- Vertical scanning is more intuitive
- Handle edge cases: empty array, empty strings
- Mention early termination optimization

---

## Problem 161: Implement atoi (String to Integer) ⭐⭐⭐

**Problem Statement:**

Implement the `myAtoi(string s)` function, which converts a string to a 32-bit signed integer.

Algorithm:
1. Read in and ignore leading whitespace
2. Check if next character is '-' or '+' (determines sign)
3. Read digits until non-digit or end of input
4. Convert to integer
5. Clamp to [−2³¹, 2³¹ − 1]

**Examples:**
```
Input: s = "42"
Output: 42

Input: s = "   -42"
Output: -42

Input: s = "4193 with words"
Output: 4193

Input: s = "words and 987"
Output: 0 (no digits at start)

Input: s = "-91283472332"
Output: -2147483648 (clamped to int.MinValue)
```

---

**Approach: State Machine**

**Concept:**
- Trim whitespace
- Handle sign
- Build number digit by digit
- Check for overflow

**Complexity:**
- Time: O(n)
- Space: O(1)

**Hints:**
```csharp
int i = 0;
int sign = 1;
long result = 0; // Use long to detect overflow

// Skip whitespace
while (i < s.Length && s[i] == ' ') i++;

// Check sign
if (i < s.Length && (s[i] == '+' || s[i] == '-'))
{
    sign = s[i] == '-' ? -1 : 1;
    i++;
}

// Build number
while (i < s.Length && char.IsDigit(s[i]))
{
    result = result * 10 + (s[i] - '0');
    
    // Check overflow
    if (result * sign > int.MaxValue) return int.MaxValue;
    if (result * sign < int.MinValue) return int.MinValue;
    
    i++;
}

return (int)(result * sign);
```

---

**Test Cases:**
```csharp
"42" → 42
"   -42" → -42
"4193 with words" → 4193
"words and 987" → 0
"-91283472332" → -2147483648
"2147483648" → 2147483647 (overflow)
"+1" → 1
"+-12" → 0 (invalid)
```

**Common Mistakes:**
- Not handling overflow correctly
- Not stopping at first non-digit
- Not handling '+' sign
- Not handling leading zeros

**Interview Tips:**
- This tests edge case handling
- Use long to detect overflow
- Be thorough with test cases
- Ask about leading zeros, multiple signs, etc.

---

## Problem 162: Regex Email Validator ⭐⭐

**Problem Statement:**

Validate if a string is a valid email address using basic rules:
- Local part (before @): letters, digits, dots, hyphens, underscores
- Domain part (after @): letters, digits, dots
- At least one dot in domain
- No consecutive dots

**Examples:**
```
Input: "user@example.com"
Output: true

Input: "user.name@example.co.uk"
Output: true

Input: "user@"
Output: false

Input: "@example.com"
Output: false

Input: "user..name@example.com"
Output: false
```

---

**Approach 1: Manual Validation**

**Concept:**
- Check for single @
- Validate local part
- Validate domain part

**Hints:**
```csharp
// Split by @
string[] parts = email.Split('@');
if (parts.Length != 2) return false;

string local = parts[0];
string domain = parts[1];

// Validate local: not empty, valid chars
// Validate domain: contains dot, valid format
```

---

**Approach 2: Regex Pattern**

**Concept:**
- Use regular expression pattern

**Hints:**
```csharp
var pattern = @"^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$";
return Regex.IsMatch(email, pattern);
```

---

**Test Cases:**
```csharp
"user@example.com" → true
"user.name@example.co.uk" → true
"user@" → false
"@example.com" → false
"user..name@example.com" → false
"user@example" → false (no dot in domain)
"user name@example.com" → false (space)
```

**Interview Tips:**
- Start with manual approach
- Then show regex (if you know it)
- Mention this is simplified version
- Real email validation is VERY complex
- In production: use libraries

---

## ✅ Track A Complete!

You've covered 12 essential string algorithm problems. Practice these until you can solve them confidently without hints!

**Next**: Track B - Array Algorithms (15 problems)

