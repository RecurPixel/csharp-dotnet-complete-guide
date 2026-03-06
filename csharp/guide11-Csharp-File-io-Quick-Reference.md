# C# File I/O Quick Reference Guide

---

## Part 1: File Operations Overview

### File I/O Concepts

**What is File I/O?** Reading from and writing to files on disk.

**Key Components:**

- **Static Classes** - Quick operations (File, Directory, Path)
- **Instance Classes** - Object-oriented with metadata (FileInfo, DirectoryInfo)
- **Streams** - Low-level byte/character operations
- **Readers/Writers** - High-level text/binary operations

**Namespaces:**

```csharp
using System.IO;              // File, Directory, Stream, Reader, Writer
using System.Text;            // Encoding
```

---

## Part 2: File Class (Static)

**What it is:** Static utility class for quick file operations

**When to use:**

- Simple, one-time operations
- Don't need file metadata
- Prefer concise code

**Namespace:** System.IO

### Reading Files

```csharp
// Read entire file as string
string content = File.ReadAllText("file.txt");

// Read with specific encoding
string content = File.ReadAllText("file.txt", Encoding.UTF8);

// Read as lines
string[] lines = File.ReadAllLines("file.txt");

// Read as bytes
byte[] bytes = File.ReadAllBytes("file.bin");

// Async versions (.NET 4.5+)
string content = await File.ReadAllTextAsync("file.txt");
string[] lines = await File.ReadAllLinesAsync("file.txt");
byte[] bytes = await File.ReadAllBytesAsync("file.bin");
```

### Writing Files

```csharp
// Write string to file (overwrites)
File.WriteAllText("file.txt", "Hello World");

// Write with encoding
File.WriteAllText("file.txt", "Hello", Encoding.UTF8);

// Write lines
string[] lines = {"Line 1", "Line 2", "Line 3"};
File.WriteAllLines("file.txt", lines);

// Write bytes
byte[] bytes = {0x48, 0x65, 0x6C, 0x6C, 0x6F};
File.WriteAllBytes("file.bin", bytes);

// Async versions
await File.WriteAllTextAsync("file.txt", "Hello");
await File.WriteAllLinesAsync("file.txt", lines);
await File.WriteAllBytesAsync("file.bin", bytes);
```

### Appending to Files

```csharp
// Append text to end of file
File.AppendAllText("log.txt", "New log entry\n");

// Append lines
string[] moreLines = {"Line 4", "Line 5"};
File.AppendAllLines("log.txt", moreLines);

// Async version
await File.AppendAllTextAsync("log.txt", "New entry\n");
await File.AppendAllLinesAsync("log.txt", moreLines);
```

### File Management

```csharp
// Check if file exists
bool exists = File.Exists("file.txt");

// Copy file
File.Copy("source.txt", "dest.txt");
File.Copy("source.txt", "dest.txt", overwrite: true);

// Move/rename file
File.Move("old.txt", "new.txt");
File.Move("old.txt", "new.txt", overwrite: true);  // .NET Core 3.0+

// Delete file
File.Delete("file.txt");  // Does not throw if file doesn't exist

// Create empty file
using FileStream fs = File.Create("file.txt");
```

### File Metadata

```csharp
// Get file attributes
FileAttributes attrs = File.GetAttributes("file.txt");
bool isReadOnly = (attrs & FileAttributes.ReadOnly) != 0;
bool isHidden = (attrs & FileAttributes.Hidden) != 0;

// Set file attributes
File.SetAttributes("file.txt", FileAttributes.ReadOnly);

// Get creation/modification times
DateTime created = File.GetCreationTime("file.txt");
DateTime modified = File.GetLastWriteTime("file.txt");
DateTime accessed = File.GetLastAccessTime("file.txt");

// Set times
File.SetCreationTime("file.txt", DateTime.Now);
File.SetLastWriteTime("file.txt", DateTime.Now);
File.SetLastAccessTime("file.txt", DateTime.Now");
```

### Opening Files

```csharp
// Open for reading
using FileStream fs = File.OpenRead("file.txt");

// Open for writing
using FileStream fs = File.OpenWrite("file.txt");

// Open with specific mode/access
using FileStream fs = File.Open("file.txt", FileMode.Open, FileAccess.Read);

// Create with StreamReader
using StreamReader reader = File.OpenText("file.txt");

// Create with StreamWriter
using StreamWriter writer = File.CreateText("file.txt");
using StreamWriter writer = File.AppendText("file.txt");
```

### File Methods Summary

|Method|Purpose|Async Version|
|---|---|---|
|`ReadAllText()`|Read entire file as string|`ReadAllTextAsync()`|
|`ReadAllLines()`|Read lines as string array|`ReadAllLinesAsync()`|
|`ReadAllBytes()`|Read as byte array|`ReadAllBytesAsync()`|
|`WriteAllText()`|Write string (overwrite)|`WriteAllTextAsync()`|
|`WriteAllLines()`|Write lines (overwrite)|`WriteAllLinesAsync()`|
|`WriteAllBytes()`|Write bytes (overwrite)|`WriteAllBytesAsync()`|
|`AppendAllText()`|Append string|`AppendAllTextAsync()`|
|`AppendAllLines()`|Append lines|`AppendAllLinesAsync()`|
|`Copy()`|Copy file|N/A|
|`Move()`|Move/rename file|N/A|
|`Delete()`|Delete file|N/A|
|`Exists()`|Check if exists|N/A|

---

## Part 3: FileInfo Class (Instance)

**What it is:** Object-oriented file operations with metadata

**When to use:**

- Need file properties (size, dates, etc.)
- Multiple operations on same file
- Working with file objects

**Namespace:** System.IO

### Creation

```csharp
FileInfo file = new FileInfo("file.txt");

// Properties available immediately (may be cached)
string name = file.Name;
long size = file.Length;
```

### Key Properties

```csharp
// Names and paths
file.Name                     // "file.txt"
file.FullName                 // "C:\Folder\file.txt"
file.Extension                // ".txt"
file.DirectoryName            // "C:\Folder"
file.Directory                // DirectoryInfo object

// Size and existence
file.Length                   // File size in bytes
file.Exists                   // Boolean if file exists

// Timestamps
file.CreationTime             // When created (local)
file.CreationTimeUtc          // When created (UTC)
file.LastWriteTime            // When last modified (local)
file.LastWriteTimeUtc         // When last modified (UTC)
file.LastAccessTime           // When last accessed (local)
file.LastAccessTimeUtc        // When last accessed (UTC)

// Attributes
file.Attributes               // FileAttributes flags
file.IsReadOnly               // Boolean shortcut
```

### File Operations

```csharp
// Create file
using FileStream fs = file.Create();

// Open file
using FileStream fs = file.Open(FileMode.Open, FileAccess.Read);
using FileStream fs = file.OpenRead();
using FileStream fs = file.OpenWrite();
using StreamReader reader = file.OpenText();

// Copy file
FileInfo newFile = file.CopyTo("copy.txt");
FileInfo newFile = file.CopyTo("copy.txt", overwrite: true);

// Move file
file.MoveTo("newpath.txt");

// Delete file
file.Delete();

// Refresh metadata (if file changed externally)
file.Refresh();
```

### FileInfo vs File

|Feature|File (Static)|FileInfo (Instance)|
|---|---|---|
|**Usage**|One-liner operations|Object with properties|
|**Performance**|Good for single operation|Better for multiple ops|
|**Metadata**|Separate method calls|Properties available|
|**Object creation**|None|Creates FileInfo object|
|**Use when**|Simple, quick tasks|Need metadata or multiple ops|

**Example:**

```csharp
// ❌ Inefficient with File (3 separate checks)
if (File.Exists("file.txt"))
{
    long size = new FileInfo("file.txt").Length;  // Creates FileInfo
    DateTime modified = File.GetLastWriteTime("file.txt");
}

// ✅ Efficient with FileInfo (single object)
FileInfo file = new FileInfo("file.txt");
if (file.Exists)
{
    long size = file.Length;
    DateTime modified = file.LastWriteTime;
}
```

---

## Part 4: Directory Class (Static)

**What it is:** Static utility for directory operations

**When to use:** Quick directory tasks

**Namespace:** System.IO

### Directory Management

```csharp
// Check if exists
bool exists = Directory.Exists("C:\\Folder");

// Create directory (creates parents too)
Directory.CreateDirectory("C:\\Folder\\SubFolder");

// Delete directory
Directory.Delete("C:\\Folder");                    // Empty only
Directory.Delete("C:\\Folder", recursive: true);   // With contents

// Move/rename directory
Directory.Move("C:\\OldFolder", "C:\\NewFolder");
```

### Getting Entries

```csharp
// Get all files
string[] files = Directory.GetFiles("C:\\Folder");

// Get files with pattern
string[] txtFiles = Directory.GetFiles("C:\\Folder", "*.txt");

// Get files with pattern (recursive)
string[] allTxt = Directory.GetFiles("C:\\Folder", "*.txt", SearchOption.AllDirectories);

// Get subdirectories
string[] dirs = Directory.GetDirectories("C:\\Folder");

// Get both files and directories
string[] entries = Directory.GetFileSystemEntries("C:\\Folder");
```

### Enumeration (Lazy Loading)

```csharp
// Enumerate files (memory efficient for large directories)
IEnumerable<string> files = Directory.EnumerateFiles("C:\\Folder");
foreach (string file in files)
{
    // Process one at a time
}

// Enumerate with pattern
IEnumerable<string> txtFiles = Directory.EnumerateFiles("C:\\Folder", "*.txt");

// Enumerate directories
IEnumerable<string> dirs = Directory.EnumerateDirectories("C:\\Folder");

// Enumerate all entries
IEnumerable<string> entries = Directory.EnumerateFileSystemEntries("C:\\Folder");
```

### Current Directory

```csharp
// Get current working directory
string current = Directory.GetCurrentDirectory();

// Set current working directory
Directory.SetCurrentDirectory("C:\\NewFolder");
```

### Directory Information

```csharp
// Get parent directory
DirectoryInfo parent = Directory.GetParent("C:\\Folder\\SubFolder");

// Get directory root
string root = Directory.GetDirectoryRoot("C:\\Folder");  // "C:\\"

// Get logical drives
string[] drives = Directory.GetLogicalDrives();  // ["C:\\", "D:\\", ...]
```

---

## Part 5: DirectoryInfo Class (Instance)

**What it is:** Object-oriented directory operations with metadata

**When to use:** Need directory properties or multiple operations

**Namespace:** System.IO

### Creation

```csharp
DirectoryInfo dir = new DirectoryInfo("C:\\Folder");

// Properties available
string name = dir.Name;
bool exists = dir.Exists;
```

### Key Properties

```csharp
// Names and paths
dir.Name                      // "Folder"
dir.FullName                  // "C:\Folder"
dir.Parent                    // DirectoryInfo of parent
dir.Root                      // DirectoryInfo of root (e.g., "C:\")

// Existence
dir.Exists                    // Boolean

// Timestamps
dir.CreationTime
dir.LastWriteTime
dir.LastAccessTime

// Attributes
dir.Attributes
```

### Directory Operations

```csharp
// Create directory
dir.Create();

// Create subdirectory
DirectoryInfo subDir = dir.CreateSubdirectory("SubFolder");

// Delete directory
dir.Delete();                 // Empty only
dir.Delete(recursive: true);  // With contents

// Move directory
dir.MoveTo("C:\\NewPath");

// Refresh metadata
dir.Refresh();
```

### Getting Entries

```csharp
// Get files
FileInfo[] files = dir.GetFiles();
FileInfo[] txtFiles = dir.GetFiles("*.txt");
FileInfo[] allTxt = dir.GetFiles("*.txt", SearchOption.AllDirectories);

// Get subdirectories
DirectoryInfo[] subDirs = dir.GetDirectories();
DirectoryInfo[] subDirs2 = dir.GetDirectories("Sub*");

// Get both files and directories
FileSystemInfo[] entries = dir.GetFileSystemInfos();

// Enumerate (lazy)
IEnumerable<FileInfo> files = dir.EnumerateFiles();
IEnumerable<DirectoryInfo> dirs = dir.EnumerateDirectories();
IEnumerable<FileSystemInfo> entries = dir.EnumerateFileSystemInfos();
```

---

## Part 6: Path Class (Static)

**What it is:** Utility for working with file and directory path strings

**When to use:** Path manipulation, validation, parsing

**Namespace:** System.IO

### Combining Paths

```csharp
// Combine paths safely
string path = Path.Combine("C:\\Folder", "file.txt");
// "C:\Folder\file.txt"

// Combine multiple parts
string path = Path.Combine("C:\\Folder", "SubFolder", "file.txt");
// "C:\Folder\SubFolder\file.txt"

// Works cross-platform (/ on Linux, \ on Windows)
```

### Extracting Path Components

```csharp
string path = "C:\\Folder\\SubFolder\\file.txt";

// File name with extension
string fileName = Path.GetFileName(path);
// "file.txt"

// File name without extension
string fileNameOnly = Path.GetFileNameWithoutExtension(path);
// "file"

// Extension
string ext = Path.GetExtension(path);
// ".txt"

// Directory path
string dir = Path.GetDirectoryName(path);
// "C:\Folder\SubFolder"

// Root
string root = Path.GetPathRoot(path);
// "C:\"
```

### Path Manipulation

```csharp
// Change extension
string newPath = Path.ChangeExtension("file.txt", ".bak");
// "file.bak"

// Get full path from relative
string fullPath = Path.GetFullPath("..\\file.txt");
// "C:\ParentFolder\file.txt" (resolved)

// Check if path has extension
bool hasExt = Path.HasExtension("file.txt");  // true

// Check if path is rooted (absolute)
bool isRooted = Path.IsPathRooted("C:\\file.txt");  // true
bool isRooted2 = Path.IsPathRooted("file.txt");     // false
```

### Temporary Files

```csharp
// Get temp directory
string tempDir = Path.GetTempPath();
// "C:\Users\Username\AppData\Local\Temp\"

// Create unique temp file name
string tempFile = Path.GetTempFileName();
// "C:\Users\Username\AppData\Local\Temp\tmp1234.tmp"

// Generate random file name
string randomName = Path.GetRandomFileName();
// "5d7j9k2a.3md" (not created, just name)
```

### Path Constants

```csharp
// Directory separator
char sep = Path.DirectorySeparatorChar;
// '\\' on Windows, '/' on Unix

// Alternative separator
char altSep = Path.AltDirectorySeparatorChar;
// '/' on Windows, '\\' on Unix

// Path separator (for environment PATH)
char pathSep = Path.PathSeparator;
// ';' on Windows, ':' on Unix

// Invalid path characters
char[] invalid = Path.GetInvalidPathChars();
char[] invalidFile = Path.GetInvalidFileNameChars();
```

### Path Methods Summary

|Method|Purpose|
|---|---|
|`Combine()`|Join paths safely|
|`GetFileName()`|Get file name with extension|
|`GetFileNameWithoutExtension()`|Get file name only|
|`GetExtension()`|Get extension|
|`GetDirectoryName()`|Get directory path|
|`GetPathRoot()`|Get root (e.g., "C:")|
|`GetFullPath()`|Resolve to absolute path|
|`ChangeExtension()`|Replace extension|
|`HasExtension()`|Check if has extension|
|`IsPathRooted()`|Check if absolute|
|`GetTempPath()`|Get temp directory|
|`GetTempFileName()`|Create temp file|
|`GetRandomFileName()`|Generate random name|

---

## Part 7: StreamReader (Text Reading)

**What it is:** Reads text files efficiently line-by-line or character-by-character

**When to use:**

- Large text files (don't load entire file)
- Line-by-line processing
- Custom parsing

**Namespace:** System.IO

### Creation

```csharp
// From file path
using StreamReader reader = new StreamReader("file.txt");

// With encoding
using StreamReader reader = new StreamReader("file.txt", Encoding.UTF8);

// From FileStream
using FileStream fs = File.OpenRead("file.txt");
using StreamReader reader = new StreamReader(fs);

// From File class helper
using StreamReader reader = File.OpenText("file.txt");
```

### Reading

```csharp
using (StreamReader reader = new StreamReader("file.txt"))
{
    // Read entire file
    string all = reader.ReadToEnd();
    
    // Read line by line
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
    
    // Or use EndOfStream
    while (!reader.EndOfStream)
    {
        string line = reader.ReadLine();
    }
    
    // Read single character
    int ch = reader.Read();  // Returns -1 at end
    
    // Read into buffer
    char[] buffer = new char[1024];
    int count = reader.ReadBlock(buffer, 0, buffer.Length);
}
```

### Async Reading

```csharp
using (StreamReader reader = new StreamReader("file.txt"))
{
    // Async read all
    string all = await reader.ReadToEndAsync();
    
    // Async read line
    string line;
    while ((line = await reader.ReadLineAsync()) != null)
    {
        Console.WriteLine(line);
    }
    
    // Async read char
    int ch = await reader.ReadAsync();
    
    // Async read block
    char[] buffer = new char[1024];
    int count = await reader.ReadBlockAsync(buffer, 0, buffer.Length);
}
```

### Key Properties & Methods

```csharp
// Properties
reader.EndOfStream            // true if at end of file
reader.CurrentEncoding        // Encoding being used
reader.BaseStream             // Underlying stream

// Methods
reader.Read();                // Read one char (as int)
reader.ReadLine();            // Read one line (or null)
reader.ReadToEnd();           // Read rest of file
reader.ReadBlock(buffer, index, count); // Read into buffer
reader.Peek();                // Peek at next char without reading
reader.Close();               // Close stream
reader.Dispose();             // Release resources
```

---

## Part 8: StreamWriter (Text Writing)

**What it is:** Writes text to files efficiently

**When to use:**

- Large text generation
- Line-by-line writing
- Log files

**Namespace:** System.IO

### Creation

```csharp
// Create new file (overwrites)
using StreamWriter writer = new StreamWriter("file.txt");

// With encoding
using StreamWriter writer = new StreamWriter("file.txt", append: false, Encoding.UTF8);

// Append to existing
using StreamWriter writer = new StreamWriter("file.txt", append: true);

// From FileStream
using FileStream fs = File.OpenWrite("file.txt");
using StreamWriter writer = new StreamWriter(fs);

// From File class helpers
using StreamWriter writer = File.CreateText("file.txt");
using StreamWriter writer = File.AppendText("file.txt");
```

### Writing

```csharp
using (StreamWriter writer = new StreamWriter("file.txt"))
{
    // Write string (no newline)
    writer.Write("Hello");
    
    // Write line (with newline)
    writer.WriteLine("Hello World");
    
    // Write formatted
    writer.WriteLine("Value: {0}", 42);
    
    // Write multiple values
    writer.Write("One");
    writer.Write(" ");
    writer.WriteLine("Two");
    
    // Flush buffer to disk
    writer.Flush();
}
```

### Async Writing

```csharp
using (StreamWriter writer = new StreamWriter("file.txt"))
{
    await writer.WriteAsync("Hello");
    await writer.WriteLineAsync("Hello World");
    await writer.WriteLineAsync($"Value: {42}");
    await writer.FlushAsync();
}
```

### Key Properties & Methods

```csharp
// Properties
writer.AutoFlush              // true = write immediately (default: false)
writer.Encoding               // Encoding being used
writer.NewLine                // Line terminator ("\r\n" on Windows)
writer.BaseStream             // Underlying stream

// Methods
writer.Write(value);          // Write without newline
writer.WriteLine(value);      // Write with newline
writer.Flush();               // Force write buffered data
writer.Close();               // Close stream
writer.Dispose();             // Release resources
```

---

## Part 9: BinaryReader & BinaryWriter

### BinaryReader

**What it is:** Reads primitive data types in binary format

**When to use:** Binary files, serialized data, custom file formats

**Namespace:** System.IO

#### Creation

```csharp
using FileStream fs = File.OpenRead("file.bin");
using BinaryReader reader = new BinaryReader(fs);

// With encoding (for strings)
using BinaryReader reader = new BinaryReader(fs, Encoding.UTF8);
```

#### Reading

```csharp
using (FileStream fs = File.OpenRead("data.bin"))
using (BinaryReader reader = new BinaryReader(fs))
{
    // Read primitive types
    bool b = reader.ReadBoolean();
    byte by = reader.ReadByte();
    sbyte sb = reader.ReadSByte();
    
    char c = reader.ReadChar();
    short s = reader.ReadInt16();
    ushort us = reader.ReadUInt16();
    
    int i = reader.ReadInt32();
    uint ui = reader.ReadUInt32();
    
    long l = reader.ReadInt64();
    ulong ul = reader.ReadUInt64();
    
    float f = reader.ReadSingle();
    double d = reader.ReadDouble();
    decimal dec = reader.ReadDecimal();
    
    // Read string (length-prefixed)
    string str = reader.ReadString();
    
    // Read byte array
    byte[] bytes = reader.ReadBytes(100);
    
    // Read char array
    char[] chars = reader.ReadChars(50);
}
```

#### Key Methods

```csharp
reader.ReadBoolean();         // 1 byte
reader.ReadByte();            // 1 byte (unsigned)
reader.ReadSByte();           // 1 byte (signed)
reader.ReadChar();            // 2 bytes (Unicode)
reader.ReadInt16();           // 2 bytes (short)
reader.ReadInt32();           // 4 bytes (int)
reader.ReadInt64();           // 8 bytes (long)
reader.ReadSingle();          // 4 bytes (float)
reader.ReadDouble();          // 8 bytes (double)
reader.ReadDecimal();         // 16 bytes (decimal)
reader.ReadString();          // Variable (length-prefixed)
reader.ReadBytes(count);      // Read byte array
reader.ReadChars(count);      // Read char array
reader.PeekChar();            // Peek without reading
reader.BaseStream;            // Underlying stream
```

---

### BinaryWriter

**What it is:** Writes primitive data types in binary format

**When to use:** Binary files, serialization, custom formats

**Namespace:** System.IO

#### Creation

```csharp
using FileStream fs = File.Create("file.bin");
using BinaryWriter writer = new BinaryWriter(fs);

// With encoding (for strings)
using BinaryWriter writer = new BinaryWriter(fs, Encoding.UTF8);
```

#### Writing

```csharp
using (FileStream fs = File.Create("data.bin"))
using (BinaryWriter writer = new BinaryWriter(fs))
{
    // Write primitive types
    writer.Write(true);              // bool
    writer.Write((byte)255);         // byte
    writer.Write('A');               // char
    writer.Write((short)1000);       // short
    writer.Write(42);                // int
    writer.Write(123456789L);        // long
    writer.Write(3.14f);             // float
    writer.Write(3.14159);           // double
    writer.Write(123.45m);           // decimal
    
    // Write string (length-prefixed)
    writer.Write("Hello World");
    
    // Write byte array
    byte[] bytes = {1, 2, 3, 4, 5};
    writer.Write(bytes);
    
    // Write char array
    char[] chars = {'A', 'B', 'C'};
    writer.Write(chars);
    
    // Flush to disk
    writer.Flush();
}
```

#### Key Methods

```csharp
writer.Write(bool);
writer.Write(byte);
writer.Write(sbyte);
writer.Write(char);
writer.Write(short);
writer.Write(ushort);
writer.Write(int);
writer.Write(uint);
writer.Write(long);
writer.Write(ulong);
writer.Write(float);
writer.Write(double);
writer.Write(decimal);
writer.Write(string);         // Length-prefixed
writer.Write(byte[]);
writer.Write(char[]);
writer.Flush();
writer.BaseStream;            // Underlying stream
```

---

## Part 10: FileStream

**What it is:** Low-level stream for reading/writing bytes to files

**When to use:**

- Maximum control over file operations
- Binary data
- Base stream for readers/writers
- Custom file protocols

**Namespace:** System.IO

### Creation

```csharp
// Full control
FileStream fs = new FileStream(
    "file.bin",
    FileMode.Open,
    FileAccess.Read,
    FileShare.None);

// Helpers
FileStream fs = File.OpenRead("file.bin");
FileStream fs = File.OpenWrite("file.bin");
FileStream fs = File.Create("file.bin");
```

### FileMode Enum

```csharp
FileMode.CreateNew            // Create new (error if exists)
FileMode.Create               // Create new (overwrite if exists)
FileMode.Open                 // Open existing (error if doesn't exist)
FileMode.OpenOrCreate         // Open or create
FileMode.Truncate             // Open and clear (error if doesn't exist)
FileMode.Append               // Open and seek to end
```

### FileAccess Enum

```csharp
FileAccess.Read               // Read-only
FileAccess.Write              // Write-only
FileAccess.ReadWrite          // Both
```

### FileShare Enum

```csharp
FileShare.None                // Exclusive access
FileShare.Read                // Others can read
FileShare.Write               // Others can write
FileShare.ReadWrite           // Others can read and write
FileShare.Delete              // Others can delete
FileShare.Inheritable         // Can be inherited by child processes
```

### Reading & Writing

```csharp
using (FileStream fs = File.OpenRead("file.bin"))
{
    // Read single byte
    int b = fs.ReadByte();  // -1 at end
    
    // Read into buffer
    byte[] buffer = new byte[1024];
    int bytesRead = fs.Read(buffer, 0, buffer.Length);
    
    // Async read
    int bytesRead = await fs.ReadAsync(buffer, 0, buffer.Length);
}

using (FileStream fs = File.Create("file.bin"))
{
    // Write single byte
    fs.WriteByte(255);
    
    // Write buffer
    byte[] buffer = {1, 2, 3, 4, 5};
    fs.Write(buffer, 0, buffer.Length);
    
    // Async write
    await fs.WriteAsync(buffer, 0, buffer.Length);
    
    // Flush to disk
    fs.Flush();
}
```

### Seeking

```csharp
using (FileStream fs = File.Open("file.bin", FileMode.Open))
{
    // Get current position
    long pos = fs.Position;
    
    // Set position
    fs.Position = 0;          // Go to start
    
    // Seek
    fs.Seek(0, SeekOrigin.Begin);     // Start
    fs.Seek(0, SeekOrigin.End);       // End
    fs.Seek(10, SeekOrigin.Current);  // Relative
    
    // Get length
    long size = fs.Length;
    
    // Set length (truncate or extend)
    fs.SetLength(1000);
}
```

### Key Properties & Methods

```csharp
// Properties
fs.Length                     // File size in bytes
fs.Position                   // Current position
fs.CanRead                    // Boolean
fs.CanWrite                   // Boolean
fs.CanSeek                    // Boolean
fs.Name                       // File path

// Methods
fs.Read(buffer, offset, count);
fs.ReadByte();
fs.Write(buffer, offset, count);
fs.WriteByte(byte);
fs.Seek(offset, origin);
fs.Flush();
fs.SetLength(length);
fs.Close();
fs.Dispose();

// Async methods
fs.ReadAsync(buffer, offset, count);
fs.WriteAsync(buffer, offset, count);
fs.FlushAsync();
```

---

## Part 11: Specialized Classes

### BufferedStream

**What it is:** Adds buffering to another stream for better performance

**When to use:** Many small reads/writes

**Namespace:** System.IO

```csharp
using FileStream fs = File.OpenRead("large.bin");
using BufferedStream bs = new BufferedStream(fs, bufferSize: 4096);

// Faster reads due to buffering
byte[] buffer = new byte[100];
int bytesRead = bs.Read(buffer, 0, buffer.Length);
```

---

### MemoryStream

**What it is:** Stream that uses memory instead of a file

**When to use:**

- In-memory buffer
- Convert between streams and byte arrays
- Testing

**Namespace:** System.IO

```csharp
// Create from byte array
byte[] data = {1, 2, 3, 4, 5};
using MemoryStream ms = new MemoryStream(data);

// Create empty
using MemoryStream ms = new MemoryStream();
ms.Write(data, 0, data.Length);

// Get as byte array
byte[] result = ms.ToArray();

// Get underlying buffer (faster but unsafe)
byte[] buffer = ms.GetBuffer();

// Write to another stream
using FileStream fs = File.Create("output.bin");
ms.WriteTo(fs);
```

---

### DriveInfo

**What it is:** Information about disk drives

**When to use:** Check disk space, drive type

**Namespace:** System.IO

```csharp
// Get all drives
DriveInfo[] drives = DriveInfo.GetDrives();

foreach (DriveInfo drive in drives)
{
    if (drive.IsReady)
    {
        Console.WriteLine($"Drive: {drive.Name}");
        Console.WriteLine($"Type: {drive.DriveType}");
        Console.WriteLine($"Format: {drive.DriveFormat}");
        Console.WriteLine($"Total: {drive.TotalSize} bytes");
        Console.WriteLine($"Free: {drive.TotalFreeSpace} bytes");
        Console.WriteLine($"Available: {drive.AvailableFreeSpace} bytes");
        Console.WriteLine($"Label: {drive.VolumeLabel}");
    }
}

// Specific drive
DriveInfo cDrive = new DriveInfo("C");
if (cDrive.IsReady)
{
    double percentFree = (double)cDrive.AvailableFreeSpace / cDrive.TotalSize * 100;
    Console.WriteLine($"C: drive is {percentFree:F2}% free");
}
```

#### DriveType Enum

```csharp
DriveType.Fixed               // Hard disk
DriveType.Removable           // USB, floppy
DriveType.CDRom               // CD/DVD
DriveType.Network             // Network drive
DriveType.Ram                 // RAM disk
DriveType.Unknown             // Unknown type
DriveType.NoRootDirectory     // No root directory
```

---

### StringReader & StringWriter

**What it is:** Read from/write to strings as streams

**When to use:** Parse strings line-by-line, build strings with stream API

**Namespace:** System.IO

#### StringReader

```csharp
string data = "Line 1\nLine 2\nLine 3";

using StringReader reader = new StringReader(data);

// Read line by line
string line;
while ((line = reader.ReadLine()) != null)
{
    Console.WriteLine(line);
}

// Read all
string all = reader.ReadToEnd();

// Read character
int ch = reader.Read();
```

#### StringWriter

```csharp
using StringWriter writer = new StringWriter();

// Write
writer.WriteLine("Line 1");
writer.WriteLine("Line 2");
writer.Write("Partial ");
writer.WriteLine("line");

// Get result
string result = writer.ToString();
// "Line 1\r\nLine 2\r\nPartial line\r\n"

// Use specific string builder
StringBuilder sb = new StringBuilder();
using StringWriter writer2 = new StringWriter(sb);
writer2.WriteLine("Hello");
```

---

### FileSystemWatcher

**What it is:** Monitors file system for changes

**When to use:** Detect file/directory changes in real-time

**Namespace:** System.IO

```csharp
FileSystemWatcher watcher = new FileSystemWatcher();

// Configure
watcher.Path = @"C:\Folder";
watcher.Filter = "*.txt";                   // Watch only .txt files
watcher.NotifyFilter = NotifyFilters.FileName 
                     | NotifyFilters.Size 
                     | NotifyFilters.LastWrite;
watcher.IncludeSubdirectories = true;

// Subscribe to events
watcher.Created += (sender, e) => 
    Console.WriteLine($"Created: {e.FullPath}");
    
watcher.Changed += (sender, e) => 
    Console.WriteLine($"Changed: {e.FullPath}");
    
watcher.Deleted += (sender, e) => 
    Console.WriteLine($"Deleted: {e.FullPath}");
    
watcher.Renamed += (sender, e) => 
    Console.WriteLine($"Renamed: {e.OldFullPath} -> {e.FullPath}");
    
watcher.Error += (sender, e) => 
    Console.WriteLine($"Error: {e.GetException()}");

// Start watching
watcher.EnableRaisingEvents = true;

// Keep alive
Console.WriteLine("Press Enter to stop...");
Console.ReadLine();

// Stop watching
watcher.EnableRaisingEvents = false;
watcher.Dispose();
```

#### NotifyFilters Enum

```csharp
NotifyFilters.FileName        // File name changes
NotifyFilters.DirectoryName   // Directory name changes
NotifyFilters.Attributes      // Attribute changes
NotifyFilters.Size            // File size changes
NotifyFilters.LastWrite       // Last write time changes
NotifyFilters.LastAccess      // Last access time changes
NotifyFilters.CreationTime    // Creation time changes
NotifyFilters.Security        // Security changes
```

---

## Part 12: Best Practices

### Error Handling

```csharp
try
{
    using StreamReader reader = new StreamReader("file.txt");
    string content = reader.ReadToEnd();
}
catch (FileNotFoundException ex)
{
    Console.WriteLine($"File not found: {ex.Message}");
}
catch (DirectoryNotFoundException ex)
{
    Console.WriteLine($"Directory not found: {ex.Message}");
}
catch (UnauthorizedAccessException ex)
{
    Console.WriteLine($"Access denied: {ex.Message}");
}
catch (IOException ex)
{
    Console.WriteLine($"I/O error: {ex.Message}");
}
```

### Always Use `using` for Streams

```csharp
// ✅ Good (automatic disposal)
using (StreamReader reader = new StreamReader("file.txt"))
{
    // Use reader
}  // Automatically disposed

// ✅ Better (using declaration, C# 8.0+)
using StreamReader reader = new StreamReader("file.txt");
// Use reader
// Disposed at end of scope

// ❌ Bad (manual disposal required)
StreamReader reader = new StreamReader("file.txt");
try
{
    // Use reader
}
finally
{
    reader?.Dispose();  // Must remember!
}
```

### Use Path.Combine for Paths

```csharp
// ❌ Bad (hardcoded separators, breaks on Linux)
string path = "C:\\Folder\\" + "file.txt";

// ✅ Good (works on all platforms)
string path = Path.Combine("C:\\Folder", "file.txt");
```

### Check Existence Before Operations

```csharp
// ✅ Good
if (File.Exists("file.txt"))
{
    string content = File.ReadAllText("file.txt");
}

if (Directory.Exists("C:\\Folder"))
{
    string[] files = Directory.GetFiles("C:\\Folder");
}
```

### Use Async Methods for I/O

```csharp
// ✅ Good (non-blocking)
public async Task ProcessFileAsync()
{
    string content = await File.ReadAllTextAsync("file.txt");
    await File.WriteAllTextAsync("output.txt", content.ToUpper());
}

// ❌ Bad (blocks thread)
public void ProcessFile()
{
    string content = File.ReadAllText("file.txt");
    File.WriteAllText("output.txt", content.ToUpper());
}
```

### Specify Encoding for Text

```csharp
// ✅ Good (explicit encoding)
string content = File.ReadAllText("file.txt", Encoding.UTF8);
File.WriteAllText("output.txt", content, Encoding.UTF8);

// ⚠️ Default encoding varies by platform
```

---

## Quick Reference Summary

### When to Use What

|Task|Use This|
|---|---|
|**Read entire text file**|`File.ReadAllText()`|
|**Write entire text file**|`File.WriteAllText()`|
|**Read line-by-line**|`StreamReader`|
|**Write line-by-line**|`StreamWriter`|
|**Binary data**|`BinaryReader` / `BinaryWriter`|
|**Low-level control**|`FileStream`|
|**In-memory data**|`MemoryStream`|
|**Parse string**|`StringReader`|
|**Build string**|`StringWriter`|
|**Path operations**|`Path` class|
|**File metadata**|`FileInfo`|
|**Directory operations**|`Directory` or `DirectoryInfo`|
|**Watch for changes**|`FileSystemWatcher`|
|**Check disk space**|`DriveInfo`|

### File vs FileInfo

- **File** - Static, one-liner operations
- **FileInfo** - Object with properties, better for multiple operations

### Directory vs DirectoryInfo

- **Directory** - Static, quick operations
- **DirectoryInfo** - Object with properties, better for multiple operations

### Common Exceptions

```csharp
FileNotFoundException          // File doesn't exist
DirectoryNotFoundException     // Directory doesn't exist
UnauthorizedAccessException    // Permission denied
IOException                    // General I/O error
PathTooLongException          // Path exceeds max length
NotSupportedException         // Invalid path format
```

---

**Guide Complete!** You now have a comprehensive File I/O reference! 📁