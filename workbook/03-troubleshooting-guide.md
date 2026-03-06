# Troubleshooting Guide

**Purpose:** Get unstuck fast when things go wrong

**Remember:** Every developer gets stuck. This is normal and part of learning!

---

## 🚨 Emergency Quick Fixes

### **Problem: My code won't run at all!**

**Try these in order:**

1. **Check for red squiggles** - Compilation errors shown as red underlines
2. **Read error message** - It tells you what's wrong!
3. **Build the project** - Press `Ctrl+Shift+B` (Visual Studio)
4. **Clean and rebuild** - Build → Clean Solution, then Build → Rebuild
5. **Restart IDE** - Close and reopen Visual Studio/VS Code
6. **Check .NET version** - Run `dotnet --version` in terminal

### **Problem: Program runs but closes immediately!**

**Solution:**
Add this at the end of your code:
```csharp
Console.WriteLine("Press any key to exit...");
Console.ReadKey();
```

Or run with `Ctrl+F5` (without debugging) in Visual Studio.

### **Problem: "Object reference not set to an instance of an object"**

**This is a NullReferenceException - most common error!**

```csharp
// ❌ WRONG
string name = null;
int length = name.Length;  // CRASH!

// ✅ RIGHT
string name = null;
if (name != null)
{
    int length = name.Length;
}

// ✅ BETTER
string name = null;
int? length = name?.Length;  // Safe navigation
```

---

## Compilation Errors (Red Squiggles)

### **CS0103: The name 'xyz' does not exist in the current context**

**What it means:** You're using a variable/method that doesn't exist or isn't in scope.

**Common causes:**
1. Typo in variable name
2. Variable declared in different scope
3. Missing `using` statement
4. Variable not declared at all

**Example:**
```csharp
// ❌ WRONG
void Method1()
{
    int x = 5;
}

void Method2()
{
    Console.WriteLine(x);  // ERROR! x doesn't exist here
}

// ✅ RIGHT
int x = 5;  // Declare at class level

void Method1() { }

void Method2()
{
    Console.WriteLine(x);  // Now it works
}
```

---

### **CS1002: ; expected**

**What it means:** Missing semicolon.

**Where to look:** Usually the line BEFORE the error!

```csharp
// ❌ WRONG
int x = 5    // Missing semicolon
int y = 10;  // Error shown here!

// ✅ RIGHT
int x = 5;
int y = 10;
```

---

### **CS0029: Cannot implicitly convert type 'X' to 'Y'**

**What it means:** Type mismatch - trying to assign wrong type.

```csharp
// ❌ WRONG
int x = "hello";  // Can't assign string to int

// ✅ RIGHT - Parse string to int
int x = int.Parse("42");

// ✅ BETTER - Safe parse
if (int.TryParse("42", out int x))
{
    // Use x
}
```

---

### **CS0161: Not all code paths return a value**

**What it means:** Method should return value but doesn't always.

```csharp
// ❌ WRONG
int GetValue(bool flag)
{
    if (flag)
        return 5;
    // What if flag is false? No return!
}

// ✅ RIGHT
int GetValue(bool flag)
{
    if (flag)
        return 5;
    else
        return 0;  // Cover all cases
}

// ✅ BETTER
int GetValue(bool flag)
{
    return flag ? 5 : 0;  // Ternary operator
}
```

---

### **CS1061: 'Type' does not contain a definition for 'Method'**

**What it means:** Trying to call method that doesn't exist on that type.

**Common causes:**
1. Typo in method name
2. Wrong object type
3. Missing `using` statement for extension method
4. Method is in different class

```csharp
// ❌ WRONG
List<int> numbers = new List<int>();
numbers.Length;  // ERROR! Lists have Count, not Length

// ✅ RIGHT
numbers.Count;

// ❌ WRONG
var result = numbers.Where(...);  // ERROR if missing using
// Need: using System.Linq;
```

---

## Runtime Errors (Program Crashes)

### **NullReferenceException**

**Most common error! Happens when accessing null object.**

```csharp
// ❌ WRONG
string name = null;
Console.WriteLine(name.Length);  // CRASH!

// ✅ RIGHT - Check for null
if (name != null)
{
    Console.WriteLine(name.Length);
}

// ✅ BETTER - Null conditional operator
Console.WriteLine(name?.Length ?? 0);

// ✅ BEST - Initialize properly
string name = ""; // Empty string, not null
```

**How to debug:**
1. Look at line number in error
2. Check which object is null
3. Trace back where it comes from
4. Add null check or initialize properly

---

### **IndexOutOfRangeException**

**Trying to access array/list element that doesn't exist.**

```csharp
// ❌ WRONG
int[] numbers = { 1, 2, 3 };
int x = numbers[3];  // CRASH! Index 0-2 only

// ✅ RIGHT
if (3 < numbers.Length)
{
    int x = numbers[3];
}

// ✅ BETTER - Use Length/Count
for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i]);
}
```

**Common causes:**
- Using `<=` instead of `<` in loop
- Forgetting arrays start at 0
- Not checking collection size

---

### **DivideByZeroException**

**Dividing by zero.**

```csharp
// ❌ WRONG
int result = 10 / 0;  // CRASH!

// ✅ RIGHT
int divisor = 0;
if (divisor != 0)
{
    int result = 10 / divisor;
}
else
{
    Console.WriteLine("Cannot divide by zero!");
}
```

---

### **FormatException**

**Usually happens with Parse methods when input is invalid.**

```csharp
// ❌ WRONG
int x = int.Parse("abc");  // CRASH!

// ✅ RIGHT
if (int.TryParse("abc", out int x))
{
    // x has valid value
}
else
{
    Console.WriteLine("Invalid number!");
}

// ALWAYS use TryParse for user input!
```

---

### **FileNotFoundException**

**File doesn't exist at specified path.**

```csharp
// ❌ WRONG
string content = File.ReadAllText("data.txt");  // Might crash!

// ✅ RIGHT
if (File.Exists("data.txt"))
{
    string content = File.ReadAllText("data.txt");
}
else
{
    Console.WriteLine("File not found!");
}

// ✅ BETTER - Use try-catch
try
{
    string content = File.ReadAllText("data.txt");
}
catch (FileNotFoundException)
{
    Console.WriteLine("File not found!");
}
```

**File path issues:**
- Use full path: `C:\Users\Name\file.txt`
- Or relative to exe: `bin\Debug\net8.0\file.txt`
- Check current directory: `Console.WriteLine(Directory.GetCurrentDirectory());`

---

## Logic Errors (Wrong Results)

### **Problem: Loop runs forever (infinite loop)**

```csharp
// ❌ WRONG
int i = 0;
while (i < 10)
{
    Console.WriteLine(i);
    // Forgot to increment i!
}

// ✅ RIGHT
int i = 0;
while (i < 10)
{
    Console.WriteLine(i);
    i++;  // Don't forget this!
}
```

**How to stop infinite loop:**
- Press `Ctrl+C` in console
- Click "Stop" button in IDE
- Close console window

---

### **Problem: Wrong calculation results**

**Common causes:**

**1. Integer division**
```csharp
// ❌ WRONG
int result = 5 / 2;  // Result: 2 (not 2.5!)

// ✅ RIGHT
double result = 5.0 / 2.0;  // Result: 2.5
// Or
double result = (double)5 / 2;  // Cast to double
```

**2. Operator precedence**
```csharp
// ❌ WRONG
int result = 5 + 2 * 3;  // Result: 11 (not 21!)

// ✅ RIGHT
int result = (5 + 2) * 3;  // Result: 21
```

**3. Floating point precision**
```csharp
// ❌ WRONG for money
double price = 0.1 + 0.2;  // Might be 0.30000000004

// ✅ RIGHT
decimal price = 0.1m + 0.2m;  // Exactly 0.3
```

---

### **Problem: Condition never true**

```csharp
// ❌ WRONG
if (name = "John")  // Single = is assignment!
{
    // This always runs!
}

// ✅ RIGHT
if (name == "John")  // Double == is comparison
{
    // Correct
}
```

---

## IDE/Environment Issues

### **Problem: IntelliSense not working**

**Solutions:**
1. **Restart IDE** - Fixes 80% of issues
2. **Delete obj/ and bin/ folders**, then rebuild
3. **Check C# extension installed** (VS Code)
4. **Update IDE** to latest version
5. **File → Close Solution, then reopen**

### **Problem: "The type or namespace name 'System' could not be found"**

**Solutions:**
1. Check .NET SDK installed: `dotnet --version`
2. Check project targets correct framework
3. Restore packages: `dotnet restore`
4. Clean and rebuild

### **Problem: Changes not taking effect**

**You're editing wrong file!**

**Check:**
1. Are you editing Program.cs?
2. Are you in correct project folder?
3. Did you save the file? (`Ctrl+S`)
4. Did you rebuild? (`Ctrl+Shift+B`)

### **Problem: Can't create new project**

**Visual Studio:**
1. Ensure .NET SDK installed
2. Run installer again, modify installation
3. Restart Visual Studio
4. Try "Repair" from installer

**VS Code:**
1. Install .NET SDK
2. Restart VS Code
3. Install C# Dev Kit extension
4. Open terminal and try: `dotnet new console -n Test`

---

## Common Beginner Mistakes

### **1. Forgetting variable declaration**

```csharp
// ❌ WRONG
x = 5;  // What type is x?

// ✅ RIGHT
int x = 5;
```

### **2. Using = instead of ==**

```csharp
// ❌ WRONG
if (x = 5)  // Assignment, not comparison!

// ✅ RIGHT
if (x == 5)  // Comparison
```

### **3. Case sensitivity**

```csharp
// ❌ WRONG
String name;  // Capital S
console.WriteLine();  // Lowercase c

// ✅ RIGHT
string name;  // Lowercase s
Console.WriteLine();  // Capital C
```

C# is case-sensitive! `Name` ≠ `name`

### **4. Scope confusion**

```csharp
// ❌ WRONG
if (true)
{
    int x = 5;
}
Console.WriteLine(x);  // ERROR! x doesn't exist here

// ✅ RIGHT
int x = 0;
if (true)
{
    x = 5;
}
Console.WriteLine(x);  // Works now
```

### **5. Missing break in switch**

```csharp
// ❌ WRONG
switch (x)
{
    case 1:
        Console.WriteLine("One");
        // Missing break!
    case 2:
        Console.WriteLine("Two");
        break;
}

// ✅ RIGHT
switch (x)
{
    case 1:
        Console.WriteLine("One");
        break;  // Don't forget!
    case 2:
        Console.WriteLine("Two");
        break;
}
```

---

## Debugging Techniques

### **1. Use Console.WriteLine()**

**Simple but effective!**

```csharp
int x = 5;
Console.WriteLine($"x = {x}");  // See value

// Track execution
Console.WriteLine("Before loop");
for (int i = 0; i < 10; i++)
{
    Console.WriteLine($"i = {i}");
}
Console.WriteLine("After loop");
```

### **2. Use Debugger (Breakpoints)**

**Visual Studio:**
1. Click in left margin (red dot appears)
2. Press F5 to run with debugger
3. Program pauses at breakpoint
4. Hover over variables to see values
5. Press F10 to step through line-by-line
6. Press F5 to continue

**VS Code:**
1. Click in left margin
2. Press F5
3. Use debug controls at top

### **3. Check Variable Values**

```csharp
// Print everything you're unsure about
Console.WriteLine($"name: {name}");
Console.WriteLine($"count: {count}");
Console.WriteLine($"list.Count: {list.Count}");
```

### **4. Simplify Code**

**If complex code doesn't work:**
1. Comment out most of it
2. Test simple version
3. Gradually add back parts
4. Find where it breaks

### **5. Rubber Duck Debugging**

**Explain your code to:**
- Rubber duck
- Friend
- Cat
- Yourself

Often you'll spot the error while explaining!

---

## When to Ask for Help

### **Try These First (30 minutes):**
1. Read error message carefully
2. Check this troubleshooting guide
3. Add Console.WriteLine() to debug
4. Try debugger with breakpoints
5. Google the exact error message
6. Check Stack Overflow

### **Still Stuck? Ask for Help!**

**Where to ask:**
- Stack Overflow (stackoverflow.com)
- Reddit r/csharp
- C# Discord servers
- Programming forums

**How to ask (get better answers!):**

**Good question format:**
```
Title: "NullReferenceException when accessing list element"

I'm trying to access a list element but getting NullReferenceException.

Code:
[paste relevant code - not entire program]

Error:
[paste exact error message]

What I've tried:
- Checked if list is null (it's not)
- Verified index is within bounds
- ...

Expected: Should print first element
Actual: Crashes with NullReferenceException
```

**❌ Bad questions:**
- "My code doesn't work, please fix"
- [Entire 500-line program pasted]
- No error message included
- "It worked yesterday!"

---

## Resources for Getting Unstuck

### **Official Documentation:**
- Microsoft C# Docs: https://docs.microsoft.com/dotnet/csharp/
- .NET API Browser: https://docs.microsoft.com/dotnet/api/

### **Q&A Sites:**
- Stack Overflow: https://stackoverflow.com/questions/tagged/c%23
- Reddit r/csharp: https://reddit.com/r/csharp
- Microsoft Q&A: https://docs.microsoft.com/answers/

### **Search Tips:**
- Include "C#" in search
- Include exact error code (e.g., "CS0103")
- Add "2024" or "NET 8" for recent answers

---

## Preventing Problems

### **Best Practices:**

**1. Write small, testable code**
```csharp
// ❌ BAD - Too much at once
void DoEverything() { /* 100 lines */ }

// ✅ GOOD - Small, focused methods
void LoadData() { /* 10 lines */ }
void ProcessData() { /* 10 lines */ }
void SaveData() { /* 10 lines */ }
```

**2. Test as you go**
- Don't write 100 lines then test
- Write 5-10 lines, run it, test it
- Add more only when current part works

**3. Use meaningful names**
```csharp
// ❌ BAD
int x;
string s;

// ✅ GOOD
int userAge;
string firstName;
```

**4. Handle errors gracefully**
```csharp
// Always validate user input
if (int.TryParse(input, out int number))
{
    // Use number
}
else
{
    Console.WriteLine("Invalid number!");
}
```

**5. Keep code simple**
- Simple code = fewer bugs
- Don't try to be clever
- Clear is better than concise

---

## Emergency Checklist

**When completely stuck, work through this:**

- [ ] Read error message
- [ ] Check line number in error
- [ ] Look for red squiggles
- [ ] Add Console.WriteLine() to debug
- [ ] Try debugger with breakpoint
- [ ] Simplify code to minimal version
- [ ] Google exact error message
- [ ] Check this troubleshooting guide
- [ ] Take a break (seriously!)
- [ ] Ask for help with good question

**Remember:** Being stuck is temporary. Every problem has a solution!

---

## Final Tips

### **When Frustrated:**
1. **Take a break** - Walk away for 15 minutes
2. **Start fresh** - Sometimes easier to rewrite
3. **Sleep on it** - Morning brain works better
4. **Ask for help** - No shame in asking!

### **Build Resilience:**
- **Every error is a learning opportunity**
- **Debugging is a skill** - you'll get better
- **Everyone struggles** - even experts
- **Persistence pays off** - keep going!

**You've got this!** 💪

Every developer you admire has been exactly where you are now. The only difference? They didn't give up.

**Keep coding!** 🚀

