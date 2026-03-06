# Setup Guide: Getting Started in 30 Minutes

**Goal:** Get you writing and running C# code as quickly as possible!

---

## What You'll Need

### **Hardware Requirements:**

**Minimum:**
- Windows 10/11, macOS 10.15+, or Linux
- 4 GB RAM
- 10 GB free disk space
- Dual-core processor

**Recommended:**
- 8 GB+ RAM
- 20 GB+ free disk space
- Quad-core processor
- SSD (faster compilation)

**Good News:** C# runs on almost any modern computer!

---

## Option 1: Visual Studio 2022 (Recommended for Windows)

### **Best For:**
- Windows users
- Want full-featured IDE
- Building large projects
- Complete beginner experience

### **Installation Steps:**

**1. Download Visual Studio 2022 Community Edition**
- Go to: https://visualstudio.microsoft.com/downloads/
- Click "Free Download" under Community edition
- **It's completely FREE!**
- File size: ~3-5 GB

**2. Run the Installer**
- Run the downloaded `vs_community.exe`
- The Visual Studio Installer will open

**3. Select Workloads**
On the "Workloads" tab, check:
- ✅ **.NET desktop development** (ESSENTIAL)
- ✅ **ASP.NET and web development** (recommended)

Click "Install" (bottom right)

**4. Wait for Installation**
- Takes 15-30 minutes
- Good time for coffee! ☕

**5. Launch Visual Studio**
- First launch will ask you to sign in (optional, can skip)
- Choose theme (Light/Dark/Blue) - purely cosmetic
- Click "Start Visual Studio"

### **Create Your First Program**

**1. Create New Project**
- Click "Create a new project"
- Search for "Console App"
- Select "Console App" (C#) - NOT .NET Framework version
- Click "Next"

**2. Configure Project**
- Project name: `HelloWorld`
- Location: Choose where to save
- Click "Next"

**3. Additional Information**
- Framework: **.NET 8.0 (Long-term support)**
- Click "Create"

**4. You Should See:**
```csharp
// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello, World!");
```

**5. Run the Program**
- Press `F5` (or click green "▶ Start" button)
- A console window appears with "Hello, World!"
- **Congratulations! You're a programmer!** 🎉

### **Visual Studio Keyboard Shortcuts**

| Action | Shortcut |
|--------|----------|
| Run program | `F5` |
| Run without debugging | `Ctrl + F5` |
| Stop debugging | `Shift + F5` |
| Build solution | `Ctrl + Shift + B` |
| Save all | `Ctrl + Shift + S` |
| Comment/Uncomment | `Ctrl + K, Ctrl + C` / `Ctrl + K, Ctrl + U` |
| Format document | `Ctrl + K, Ctrl + D` |
| IntelliSense | `Ctrl + Space` |

---

## Option 2: Visual Studio Code (Cross-Platform)

### **Best For:**
- macOS or Linux users
- Lightweight editor preference
- Already familiar with VS Code
- Want flexibility

### **Installation Steps:**

**1. Install .NET 8 SDK**
- Go to: https://dotnet.microsoft.com/download
- Download .NET 8.0 SDK
- Run installer
- Verify installation:
  ```bash
  dotnet --version
  ```
  Should show: `8.0.x`

**2. Install Visual Studio Code**
- Go to: https://code.visualstudio.com/
- Download for your OS
- Install

**3. Install C# Extension**
- Open VS Code
- Click Extensions icon (left sidebar) or press `Ctrl+Shift+X`
- Search: "C# Dev Kit"
- Click "Install" on "C# Dev Kit" by Microsoft
- Wait for installation

**4. Restart VS Code**

### **Create Your First Program**

**1. Open Terminal in VS Code**
- Press `` Ctrl + ` `` (backtick)
- Or: View → Terminal

**2. Create New Console Project**
```bash
dotnet new console -n HelloWorld
cd HelloWorld
```

**3. Open the Project**
```bash
code .
```
(Or: File → Open Folder → select HelloWorld)

**4. You Should See:**
```csharp
// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello, World!");
```

**5. Run the Program**
- Press `F5`
- Or in terminal:
  ```bash
  dotnet run
  ```
- Output: `Hello, World!`
- **You're coding!** 🎉

### **VS Code Keyboard Shortcuts**

| Action | Shortcut |
|--------|----------|
| Run program | `F5` (after setting up launch.json) |
| Open terminal | `` Ctrl + ` `` |
| Command palette | `Ctrl + Shift + P` |
| Save | `Ctrl + S` |
| Format document | `Shift + Alt + F` |
| Comment/Uncomment | `Ctrl + /` |
| IntelliSense | `Ctrl + Space` |

---

## Option 3: Online (No Installation)

### **Best For:**
- Quick testing
- Can't install software
- Trying C# before committing
- Shared computers

### **Recommended Online IDEs:**

**1. .NET Fiddle** (Best)
- URL: https://dotnetfiddle.net/
- No sign-up required
- Supports C# console apps
- Can save and share code

**2. Replit**
- URL: https://replit.com/
- Sign up required (free)
- Full project support
- Collaboration features

**3. OneCompiler**
- URL: https://onecompiler.com/csharp
- Simple interface
- Quick testing
- No account needed

### **Limitations:**
- No debugging tools
- Limited project complexity
- Internet required
- Not suitable for capstone projects

**Recommendation:** Use online IDEs for early chapters, then install proper IDE.

---

## Understanding Your Development Environment

### **What Just Happened?**

When you created your first program, several things were set up:

**1. Project File (`.csproj`)**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
</Project>
```
- Defines project type (Console Application)
- Specifies .NET version (8.0)
- Build configuration

**2. Program.cs**
```csharp
Console.WriteLine("Hello, World!");
```
- Your actual code
- Entry point of application
- Top-level statements (C# 10+ feature)

**3. bin/ folder** (appears after building)
- Contains compiled executable
- Debug vs Release builds

**4. obj/ folder**
- Temporary build files
- Can be safely deleted

---

## Essential Concepts

### **What is .NET?**

**.NET** is a free, open-source development platform:
- Build console apps, web apps, mobile apps, games
- Write once, run anywhere (Windows, Mac, Linux)
- Includes libraries for common tasks
- Managed by Microsoft

**Versions:**
- .NET Framework (old, Windows-only) ❌
- .NET Core (newer, cross-platform) ✅
- .NET 5, 6, 7, 8+ (modern, unified) ✅

**We use .NET 8** (latest LTS - Long Term Support)

### **What is C#?**

**C# (pronounced "C Sharp")** is:
- Programming language
- Type-safe and object-oriented
- Designed by Microsoft
- Used for Windows apps, web services, games (Unity), and more

**Current version: C# 12** (comes with .NET 8)

### **Compilation Process:**

```
Your Code (Program.cs)
    ↓
C# Compiler (csc.exe)
    ↓
Intermediate Language (IL)
    ↓
Common Language Runtime (CLR)
    ↓
Machine Code
    ↓
Runs on Your Computer
```

**Key Point:** C# is compiled, not interpreted. This makes it fast!

---

## Your First Real Program

Let's modify Hello World to understand the basics:

```csharp
// This is a comment - ignored by compiler

// Using directive - brings in pre-built code
using System;

// Namespace - organizes code (optional with top-level statements)
// Top-level statements (C# 10+) - code runs directly

// Output to console
Console.WriteLine("Hello, World!");

// Ask for user input
Console.Write("What's your name? ");
string name = Console.ReadLine();

// Output with variable
Console.WriteLine($"Hello, {name}! Welcome to C#!");

// Wait for key press before closing
Console.WriteLine("Press any key to exit...");
Console.ReadKey();
```

**Try running this!**

### **What's Happening:**

1. `Console.WriteLine()` - Outputs text and moves to new line
2. `Console.Write()` - Outputs text without new line
3. `Console.ReadLine()` - Reads user input
4. `string name` - Creates a variable to store text
5. `$"..."` - String interpolation (embed variables)
6. `Console.ReadKey()` - Waits for key press

---

## Common Setup Issues

### **Issue: "dotnet command not found"**

**Solution:**
- Restart terminal/computer after installing .NET SDK
- Check installation: `dotnet --version`
- Re-download and install .NET SDK

### **Issue: "No framework found"**

**Solution:**
- Install .NET SDK (not just Runtime)
- Check SDK: `dotnet --list-sdks`
- Framework version mismatch - update .csproj

### **Issue: VS Code C# extension not working**

**Solution:**
- Install "C# Dev Kit" (not just "C#")
- Restart VS Code
- Open a .cs file to activate extension
- Check output panel for errors

### **Issue: Program closes immediately**

**Solution:**
Add at the end of your code:
```csharp
Console.WriteLine("Press any key to exit...");
Console.ReadKey();
```

Or run with `Ctrl+F5` (without debugging) in Visual Studio.

### **Issue: Can't find Visual Studio 2022 option**

**Solution:**
- Make sure you downloaded "Community" edition (free)
- Not "Code" (different product)
- Not "Express" (discontinued)

---

## Configuring Your Environment

### **Visual Studio Recommendations:**

**1. Enable Line Numbers**
- Tools → Options
- Text Editor → All Languages → General
- Check "Line numbers"

**2. Change Theme**
- Tools → Options
- Environment → General
- Color theme: Blue/Dark/Light

**3. Font Size**
- Tools → Options
- Environment → Fonts and Colors
- Size: 12-14 recommended

**4. Auto-Save**
- Tools → Options
- Environment → Documents
- Check "Save documents as..."

### **VS Code Recommendations:**

**1. Settings Sync**
- Sign in with Microsoft/GitHub
- Sync settings across devices

**2. Useful Extensions**
- "C# Dev Kit" (essential)
- "Error Lens" (inline errors)
- "Prettier" (formatting)
- "GitLens" (if using Git)

**3. Settings (File → Preferences → Settings):**
- Auto Save: `onFocusChange`
- Format On Save: `true`
- Font Size: `14`

---

## Testing Your Setup

### **Verification Program:**

Create a new console app and run this:

```csharp
using System;

Console.WriteLine("=== C# Setup Verification ===\n");

// Check .NET version
Console.WriteLine($"✓ .NET Version: {Environment.Version}");

// Check OS
Console.WriteLine($"✓ Operating System: {Environment.OSVersion}");

// Check if compilation works
int x = 5;
int y = 10;
int sum = x + y;
Console.WriteLine($"✓ Compilation: 5 + 10 = {sum}");

// Check user input
Console.Write("\n✓ User Input: Type your name: ");
string name = Console.ReadLine();
Console.WriteLine($"✓ Input Received: Hello, {name}!");

Console.WriteLine("\n✅ All checks passed! Setup complete!");
Console.WriteLine("Press any key to exit...");
Console.ReadKey();
```

**If this runs successfully, you're ready to start learning!**

---

## Project Organization Tips

### **Folder Structure:**

```
CSharpLearning/
├── Phase1-Fundamentals/
│   ├── Problem01-Calculator/
│   ├── Problem02-NumberGuess/
│   └── ...
├── Phase2-OOP/
│   ├── Problem11-BasicClass/
│   └── ...
├── Phase3-Collections/
├── Phase4-Advanced/
├── Phase5-Async/
├── Phase6-Integration/
├── InterviewProblems/
└── CapstonProjects/
```

**Tips:**
- One folder per phase
- One project per problem
- Use descriptive names
- Keep it organized from day one!

---

## Version Control (Git) - Optional but Recommended

### **Why Use Git?**
- Track your progress
- Experiment without fear
- Build portfolio on GitHub
- Industry standard

### **Basic Git Setup:**

**1. Install Git**
- Windows: https://git-scm.com/download/win
- Mac: `brew install git` or download
- Linux: `sudo apt install git`

**2. Configure Git**
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**3. For Each Project:**
```bash
cd YourProject
git init
git add .
git commit -m "Initial commit"
```

**4. Push to GitHub (optional):**
- Create repository on GitHub
- Follow their instructions to push

**Don't worry about Git now - you can add it later!**

---

## Productivity Tools (Optional)

### **Recommended But Not Required:**

**1. LINQPad** (Windows)
- Quick C# testing
- LINQ visualization
- Free version available
- https://www.linqpad.net/

**2. Snippet Manager**
- Save code snippets
- Built into Visual Studio
- Tools → Code Snippets Manager

**3. ReSharper** (Advanced)
- Visual Studio extension
- Code analysis and refactoring
- Paid (student licenses available)
- **NOT needed for this book**

---

## What's Next?

### **You're Ready When:**
✅ Visual Studio or VS Code installed
✅ .NET 8 SDK installed  
✅ Created and ran "Hello World"
✅ Understand basic IDE navigation
✅ Can create new console projects

### **Now You Can:**
1. **Start Phase 1, Problem 1** (Simple Calculator)
2. Create a dedicated learning folder
3. Set aside daily practice time
4. Join the learning community

---

## Quick Reference Card

### **Creating New Project:**

**Visual Studio:**
1. File → New → Project
2. Console App (C#)
3. Name it, create it, start coding!

**VS Code / Terminal:**
```bash
dotnet new console -n ProjectName
cd ProjectName
code .
```

### **Running Your Code:**

**Visual Studio:**
- Press `F5` (with debugging)
- Press `Ctrl+F5` (without debugging)

**VS Code / Terminal:**
```bash
dotnet run
```

### **Common Commands:**

```bash
dotnet new console -n MyApp    # Create new console app
dotnet build                    # Compile project
dotnet run                      # Run project
dotnet clean                    # Clean build files
dotnet --list-sdks             # List installed SDKs
```

---

## Troubleshooting Checklist

**Program won't run?**
- [ ] Check for compilation errors (red underlines)
- [ ] Make sure project builds (no red X's)
- [ ] Verify .NET SDK installed
- [ ] Restart IDE

**Can't see output?**
- [ ] Add `Console.ReadKey()` at end
- [ ] Run with `Ctrl+F5` (Visual Studio)
- [ ] Check console window didn't close

**IntelliSense not working?**
- [ ] Check C# extension installed (VS Code)
- [ ] Reopen project
- [ ] Delete obj/ and bin/, rebuild
- [ ] Restart IDE

**Still stuck?**
- Search error message on Google
- Check Stack Overflow
- Ask in community forums
- Re-read this chapter

---

## You're Ready! 🚀

**Setup Complete!** ✅

You now have:
- ✅ Development environment installed
- ✅ First program running
- ✅ Understanding of basic workflow
- ✅ Troubleshooting knowledge

**Time to start coding!**

**Next Chapter: Phase 1 - Fundamentals**

Let's build that Simple Calculator! 💻✨

