# SQL Server, T-SQL & Database Fundamentals Quick Reference

---

## What is SQL Server?

**SQL Server** = Microsoft's enterprise relational database management system (RDBMS)

**T-SQL** = Transact-SQL, Microsoft's extension of SQL standard

**Key Features:**
- ✅ **ACID compliant** - Reliable transactions
- ✅ **High performance** - Optimized query execution
- ✅ **Scalability** - Handles small to enterprise workloads
- ✅ **Security** - Row-level security, encryption, auditing
- ✅ **Integration** - Works seamlessly with .NET ecosystem
- ✅ **Rich tooling** - SQL Server Management Studio (SSMS), Azure Data Studio

### SQL Server Editions

**Express** - Free, limited to 10GB database size
```sql
-- Perfect for: Development, small applications, learning
```

**Developer** - Free, full features (non-production only)
```sql
-- Perfect for: Development and testing
```

**Standard** - Commercial, for small-medium businesses
```sql
-- Perfect for: Production workloads, limited enterprise features
```

**Enterprise** - Commercial, all features
```sql
-- Perfect for: Large-scale mission-critical applications
```

---

## SQL Language Categories

### DDL (Data Definition Language)
Define and modify database structure
```sql
CREATE, ALTER, DROP, TRUNCATE
```

### DML (Data Manipulation Language)
Manipulate data in tables
```sql
SELECT, INSERT, UPDATE, DELETE
```

### DCL (Data Control Language)
Control access permissions
```sql
GRANT, REVOKE, DENY
```

### TCL (Transaction Control Language)
Manage transactions
```sql
BEGIN TRANSACTION, COMMIT, ROLLBACK, SAVEPOINT
```

---

## Data Types in SQL Server

### Numeric Types

```sql
-- Exact Numeric
INT             -- -2,147,483,648 to 2,147,483,647 (4 bytes)
BIGINT          -- -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 (8 bytes)
SMALLINT        -- -32,768 to 32,767 (2 bytes)
TINYINT         -- 0 to 255 (1 byte)
DECIMAL(p,s)    -- Fixed precision: DECIMAL(18,2) = 123456.78
NUMERIC(p,s)    -- Same as DECIMAL
MONEY           -- -922,337,203,685,477.5808 to 922,337,203,685,477.5807
SMALLMONEY      -- -214,748.3648 to 214,748.3647
BIT             -- 0, 1, or NULL (boolean-like)

-- Approximate Numeric
FLOAT(n)        -- Floating point (4 or 8 bytes)
REAL            -- Floating point (4 bytes)

-- Examples
CREATE TABLE Products (
    ProductId INT,
    Price DECIMAL(10,2),      -- 99999999.99
    Quantity SMALLINT,
    InStock BIT,              -- 0 = false, 1 = true
    Weight FLOAT
);
```

### String Types

```sql
-- Fixed Length
CHAR(n)         -- Fixed-length, padded with spaces (max 8,000)
NCHAR(n)        -- Unicode fixed-length (max 4,000)

-- Variable Length
VARCHAR(n)      -- Variable-length (max 8,000)
VARCHAR(MAX)    -- Variable-length (max ~2GB)
NVARCHAR(n)     -- Unicode variable-length (max 4,000)
NVARCHAR(MAX)   -- Unicode variable-length (max ~2GB)

-- Text (Deprecated - use VARCHAR(MAX))
TEXT            -- ⚠️ Deprecated, use VARCHAR(MAX)
NTEXT           -- ⚠️ Deprecated, use NVARCHAR(MAX)

-- Examples
CREATE TABLE Users (
    Username VARCHAR(50),        -- Up to 50 characters
    Password NVARCHAR(255),      -- Unicode, up to 255 characters
    Bio NVARCHAR(MAX),           -- Large text
    CountryCode CHAR(2)          -- Fixed: 'US', 'UK', etc.
);

-- ✅ Best Practice: Use NVARCHAR for Unicode support
-- ✅ Use VARCHAR(MAX) instead of TEXT
-- ✅ Use appropriate length - don't over-allocate
```

### Date and Time Types

```sql
DATE            -- Date only: '2025-01-16'
TIME            -- Time only: '14:30:00.1234567'
DATETIME        -- Date + Time: '2025-01-16 14:30:00' (accuracy: 3.33ms)
DATETIME2       -- Date + Time: '2025-01-16 14:30:00.1234567' (accuracy: 100ns)
SMALLDATETIME   -- Date + Time (accuracy: 1 minute)
DATETIMEOFFSET  -- Date + Time + Timezone: '2025-01-16 14:30:00.1234567 +05:30'

-- Examples
CREATE TABLE Events (
    EventDate DATE,
    StartTime TIME,
    CreatedAt DATETIME2,           -- ✅ Recommended
    UpdatedAt DATETIMEOFFSET       -- ✅ For timezone-aware
);

-- ✅ Best Practice: Use DATETIME2 instead of DATETIME (more accurate)
-- ✅ Use DATE/TIME separately if you don't need both
-- ✅ Use DATETIMEOFFSET for timezone-aware applications
```

### Binary Types

```sql
BINARY(n)       -- Fixed-length binary (max 8,000)
VARBINARY(n)    -- Variable-length binary (max 8,000)
VARBINARY(MAX)  -- Variable-length binary (max ~2GB)
IMAGE           -- ⚠️ Deprecated, use VARBINARY(MAX)

-- Examples
CREATE TABLE Files (
    FileData VARBINARY(MAX),    -- Store files
    Thumbnail VARBINARY(1024)   -- Store small images
);
```

### Other Types

```sql
UNIQUEIDENTIFIER    -- GUID: '6F9619FF-8B86-D011-B42D-00C04FC964FF'
XML                 -- XML documents
JSON                -- ⚠️ No native type, use NVARCHAR(MAX) + JSON functions
SQL_VARIANT         -- Can store any data type (use sparingly)

-- Examples
CREATE TABLE Documents (
    DocumentId UNIQUEIDENTIFIER DEFAULT NEWID(),
    XmlContent XML,
    JsonContent NVARCHAR(MAX)
);
```

---

## DDL - Data Definition Language

### CREATE TABLE

```sql
-- Basic table
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY IDENTITY(1,1),  -- Auto-increment
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    Email NVARCHAR(100) UNIQUE NOT NULL,
    Phone VARCHAR(20),
    DateOfBirth DATE,
    IsActive BIT DEFAULT 1,
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2
);

-- Table with constraints
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY IDENTITY(1,1),
    CustomerId INT NOT NULL,
    OrderDate DATETIME2 DEFAULT GETDATE(),
    TotalAmount DECIMAL(10,2) CHECK (TotalAmount >= 0),
    Status NVARCHAR(20) DEFAULT 'Pending',
    
    -- Foreign key
    CONSTRAINT FK_Orders_Customers 
        FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
        ON DELETE CASCADE  -- Delete orders when customer is deleted
        ON UPDATE CASCADE, -- Update orders when customer ID changes
    
    -- Check constraint
    CONSTRAINT CHK_Orders_Status 
        CHECK (Status IN ('Pending', 'Processing', 'Completed', 'Cancelled'))
);

-- Table with composite primary key
CREATE TABLE OrderItems (
    OrderId INT NOT NULL,
    ProductId INT NOT NULL,
    Quantity INT NOT NULL CHECK (Quantity > 0),
    UnitPrice DECIMAL(10,2) NOT NULL,
    
    PRIMARY KEY (OrderId, ProductId),  -- Composite key
    FOREIGN KEY (OrderId) REFERENCES Orders(OrderId),
    FOREIGN KEY (ProductId) REFERENCES Products(ProductId)
);

-- Computed column
CREATE TABLE Invoices (
    InvoiceId INT PRIMARY KEY,
    Subtotal DECIMAL(10,2) NOT NULL,
    TaxRate DECIMAL(5,4) NOT NULL DEFAULT 0.10,
    Tax AS (Subtotal * TaxRate) PERSISTED,  -- Computed and stored
    Total AS (Subtotal + (Subtotal * TaxRate))  -- Computed on read
);
```

### ALTER TABLE

```sql
-- Add column
ALTER TABLE Customers
ADD MiddleName NVARCHAR(50) NULL;

-- Add multiple columns
ALTER TABLE Customers
ADD 
    Country NVARCHAR(50) DEFAULT 'USA',
    PostalCode VARCHAR(10);

-- Modify column
ALTER TABLE Customers
ALTER COLUMN Phone NVARCHAR(20);  -- Change from VARCHAR to NVARCHAR

-- Drop column
ALTER TABLE Customers
DROP COLUMN MiddleName;

-- Add constraint
ALTER TABLE Customers
ADD CONSTRAINT CHK_Customers_Email 
    CHECK (Email LIKE '%@%.%');

-- Drop constraint
ALTER TABLE Customers
DROP CONSTRAINT CHK_Customers_Email;

-- Add foreign key
ALTER TABLE Orders
ADD CONSTRAINT FK_Orders_Customers
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId);
```

### DROP TABLE

```sql
-- Drop table (cannot undo!)
DROP TABLE IF EXISTS TempTable;

-- Drop multiple tables
DROP TABLE IF EXISTS Table1, Table2, Table3;

-- ⚠️ Cannot drop if foreign keys reference it
-- Must drop dependent tables first or drop foreign keys
```

### TRUNCATE TABLE

```sql
-- Remove all rows, keep structure (faster than DELETE)
TRUNCATE TABLE TempData;

-- ✅ Faster than DELETE
-- ✅ Resets IDENTITY counter
-- ❌ Cannot use with foreign key references
-- ❌ Cannot use WHERE clause
-- ❌ Cannot be rolled back (in some scenarios)
```

---

## DML - Data Manipulation Language

### SELECT - Query Data

#### Basic SELECT

```sql
-- Select all columns
SELECT * FROM Customers;

-- Select specific columns
SELECT FirstName, LastName, Email FROM Customers;

-- With alias
SELECT 
    FirstName AS First,
    LastName AS Last,
    Email AS EmailAddress
FROM Customers;

-- Concatenate columns
SELECT 
    FirstName + ' ' + LastName AS FullName,
    Email
FROM Customers;

-- With expressions
SELECT 
    ProductName,
    Price,
    Quantity,
    Price * Quantity AS TotalValue
FROM Products;
```

#### WHERE Clause - Filter Rows

```sql
-- Comparison operators
SELECT * FROM Products WHERE Price > 100;
SELECT * FROM Products WHERE Price >= 100;
SELECT * FROM Products WHERE Price < 100;
SELECT * FROM Products WHERE Price <= 100;
SELECT * FROM Products WHERE Price = 100;
SELECT * FROM Products WHERE Price <> 100;  -- Not equal (also !=)

-- Logical operators
SELECT * FROM Products 
WHERE Price > 50 AND Quantity > 10;

SELECT * FROM Products 
WHERE Category = 'Electronics' OR Category = 'Books';

SELECT * FROM Products 
WHERE NOT (Price > 100);

-- BETWEEN
SELECT * FROM Products 
WHERE Price BETWEEN 50 AND 100;  -- Inclusive

-- IN
SELECT * FROM Customers 
WHERE Country IN ('USA', 'UK', 'Canada');

-- LIKE (pattern matching)
SELECT * FROM Customers 
WHERE LastName LIKE 'S%';       -- Starts with S

SELECT * FROM Customers 
WHERE Email LIKE '%@gmail.com'; -- Ends with @gmail.com

SELECT * FROM Customers 
WHERE FirstName LIKE '%oh%';    -- Contains 'oh'

SELECT * FROM Customers 
WHERE Phone LIKE '555-____';    -- _ matches single character

-- IS NULL / IS NOT NULL
SELECT * FROM Customers 
WHERE Phone IS NULL;

SELECT * FROM Customers 
WHERE Email IS NOT NULL;

-- Complex conditions
SELECT * FROM Orders 
WHERE 
    (Status = 'Completed' OR Status = 'Processing')
    AND TotalAmount > 100
    AND OrderDate >= '2024-01-01';
```

#### ORDER BY - Sort Results

```sql
-- Ascending (default)
SELECT * FROM Customers 
ORDER BY LastName;

-- Descending
SELECT * FROM Products 
ORDER BY Price DESC;

-- Multiple columns
SELECT * FROM Customers 
ORDER BY LastName ASC, FirstName ASC;

-- By column position (not recommended)
SELECT FirstName, LastName FROM Customers 
ORDER BY 2, 1;  -- Order by 2nd column, then 1st

-- With expression
SELECT ProductName, Price, Quantity, (Price * Quantity) AS Total
FROM Products
ORDER BY Total DESC;
```

#### TOP / OFFSET-FETCH - Limit Results

```sql
-- TOP - Get first N rows
SELECT TOP 10 * FROM Customers;

-- TOP with percentage
SELECT TOP 10 PERCENT * FROM Products ORDER BY Price DESC;

-- TOP with ties
SELECT TOP 5 WITH TIES * FROM Products ORDER BY Price DESC;

-- OFFSET-FETCH (SQL Server 2012+, preferred)
SELECT * FROM Products
ORDER BY ProductId
OFFSET 0 ROWS       -- Skip 0 rows
FETCH NEXT 10 ROWS ONLY;  -- Get 10 rows

-- Pagination example
SELECT * FROM Products
ORDER BY ProductId
OFFSET 20 ROWS      -- Skip first 20 (page 3)
FETCH NEXT 10 ROWS ONLY;  -- Get next 10
```

#### DISTINCT - Unique Values

```sql
-- Remove duplicates
SELECT DISTINCT Country FROM Customers;

-- Multiple columns
SELECT DISTINCT City, Country FROM Customers;

-- Count distinct
SELECT COUNT(DISTINCT Country) AS UniqueCountries 
FROM Customers;
```

### INSERT - Add Data

```sql
-- Insert single row
INSERT INTO Customers (FirstName, LastName, Email)
VALUES ('John', 'Doe', 'john@example.com');

-- Insert multiple rows
INSERT INTO Customers (FirstName, LastName, Email)
VALUES 
    ('Jane', 'Smith', 'jane@example.com'),
    ('Bob', 'Johnson', 'bob@example.com'),
    ('Alice', 'Williams', 'alice@example.com');

-- Insert all columns (not recommended)
INSERT INTO Customers
VALUES ('Mike', 'Brown', 'mike@example.com', '555-1234', '1990-01-01', 1, GETDATE(), NULL);

-- Insert with SELECT
INSERT INTO CustomersBackup (FirstName, LastName, Email)
SELECT FirstName, LastName, Email
FROM Customers
WHERE Country = 'USA';

-- Insert and get generated ID
INSERT INTO Customers (FirstName, LastName, Email)
VALUES ('Tom', 'Davis', 'tom@example.com');

SELECT SCOPE_IDENTITY() AS NewId;  -- ✅ Recommended
-- Or
SELECT @@IDENTITY AS NewId;  -- Less safe (returns last ID in session)

-- Insert with OUTPUT clause (get inserted data)
INSERT INTO Customers (FirstName, LastName, Email)
OUTPUT INSERTED.CustomerId, INSERTED.FirstName
VALUES ('Sarah', 'Miller', 'sarah@example.com');
```

### UPDATE - Modify Data

```sql
-- Update single column
UPDATE Customers
SET Email = 'newemail@example.com'
WHERE CustomerId = 1;

-- Update multiple columns
UPDATE Customers
SET 
    FirstName = 'Jonathan',
    LastName = 'Doe',
    UpdatedAt = GETDATE()
WHERE CustomerId = 1;

-- Update with calculation
UPDATE Products
SET Price = Price * 1.1  -- Increase by 10%
WHERE Category = 'Electronics';

-- Update from SELECT
UPDATE Orders
SET Status = 'Completed'
WHERE OrderId IN (
    SELECT OrderId FROM OrderItems
    WHERE Quantity > 10
);

-- Update with JOIN
UPDATE o
SET o.TotalAmount = SUM(oi.Quantity * oi.UnitPrice)
FROM Orders o
INNER JOIN OrderItems oi ON o.OrderId = oi.OrderId
GROUP BY o.OrderId;

-- Update with OUTPUT (track changes)
UPDATE Customers
SET IsActive = 0
OUTPUT 
    DELETED.CustomerId,
    DELETED.FirstName,
    INSERTED.IsActive AS NewStatus
WHERE LastLoginDate < '2023-01-01';

-- ⚠️ ALWAYS use WHERE clause unless updating all rows!
-- Test with SELECT first:
-- SELECT * FROM Customers WHERE CustomerId = 1;
```

### DELETE - Remove Data

```sql
-- Delete specific rows
DELETE FROM Customers
WHERE CustomerId = 1;

-- Delete with condition
DELETE FROM Orders
WHERE Status = 'Cancelled' AND OrderDate < '2023-01-01';

-- Delete with subquery
DELETE FROM Customers
WHERE CustomerId IN (
    SELECT CustomerId FROM Orders
    WHERE TotalAmount = 0
);

-- Delete all rows (not recommended, use TRUNCATE instead)
DELETE FROM TempTable;

-- Delete with OUTPUT (track deletions)
DELETE FROM Customers
OUTPUT DELETED.*
WHERE IsActive = 0;

-- Delete with JOIN
DELETE c
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE o.TotalAmount = 0;

-- ⚠️ ALWAYS use WHERE clause unless deleting all rows!
-- Test with SELECT first:
-- SELECT * FROM Customers WHERE IsActive = 0;
```

---

## Joins - Combining Tables

### INNER JOIN

```sql
-- Basic INNER JOIN
SELECT 
    c.FirstName,
    c.LastName,
    o.OrderId,
    o.OrderDate,
    o.TotalAmount
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId;

-- Multiple joins
SELECT 
    c.FirstName + ' ' + c.LastName AS CustomerName,
    o.OrderId,
    p.ProductName,
    oi.Quantity,
    oi.UnitPrice,
    (oi.Quantity * oi.UnitPrice) AS LineTotal
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId
INNER JOIN OrderItems oi ON o.OrderId = oi.OrderId
INNER JOIN Products p ON oi.ProductId = p.ProductId;

-- Self join
SELECT 
    e.FirstName + ' ' + e.LastName AS Employee,
    m.FirstName + ' ' + m.LastName AS Manager
FROM Employees e
INNER JOIN Employees m ON e.ManagerId = m.EmployeeId;
```

### LEFT JOIN (LEFT OUTER JOIN)

```sql
-- All customers, with their orders (if any)
SELECT 
    c.CustomerId,
    c.FirstName,
    c.LastName,
    o.OrderId,
    o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId;

-- Find customers with no orders
SELECT 
    c.CustomerId,
    c.FirstName,
    c.LastName
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE o.OrderId IS NULL;

-- Multiple LEFT JOINs
SELECT 
    c.FirstName,
    COUNT(o.OrderId) AS OrderCount,
    COALESCE(SUM(o.TotalAmount), 0) AS TotalSpent
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
GROUP BY c.CustomerId, c.FirstName;
```

### RIGHT JOIN (RIGHT OUTER JOIN)

```sql
-- All orders, with customer info (if available)
SELECT 
    o.OrderId,
    o.OrderDate,
    c.FirstName,
    c.LastName
FROM Orders o
RIGHT JOIN Customers c ON o.CustomerId = c.CustomerId;

-- ⚠️ Less common, usually rewritten as LEFT JOIN
-- RIGHT JOIN is the mirror of LEFT JOIN
```

### FULL OUTER JOIN

```sql
-- All customers and all orders
SELECT 
    c.CustomerId,
    c.FirstName,
    o.OrderId,
    o.TotalAmount
FROM Customers c
FULL OUTER JOIN Orders o ON c.CustomerId = o.CustomerId;

-- Find unmatched records from both sides
SELECT 
    c.CustomerId,
    c.FirstName,
    o.OrderId
FROM Customers c
FULL OUTER JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE c.CustomerId IS NULL OR o.OrderId IS NULL;
```

### CROSS JOIN

```sql
-- Cartesian product (every combination)
SELECT 
    c.FirstName,
    p.ProductName
FROM Customers c
CROSS JOIN Products p;

-- Useful for generating combinations
SELECT 
    d.Date,
    e.EmployeeName
FROM Dates d
CROSS JOIN Employees e
WHERE d.Date BETWEEN '2025-01-01' AND '2025-01-31';
```

### Join Comparison Table

```sql
-- Sample data
Customers: 1, 2, 3
Orders:    2, 3, 4

-- INNER JOIN:     2, 3        (only matching)
-- LEFT JOIN:      1, 2, 3     (all from left)
-- RIGHT JOIN:     2, 3, 4     (all from right)
-- FULL JOIN:      1, 2, 3, 4  (all from both)
-- CROSS JOIN:     1×3, 2×3, 3×3, ...  (all combinations)
```

---

## Subqueries

### Scalar Subquery (Returns single value)

```sql
-- In SELECT
SELECT 
    ProductName,
    Price,
    (SELECT AVG(Price) FROM Products) AS AvgPrice,
    Price - (SELECT AVG(Price) FROM Products) AS Difference
FROM Products;

-- In WHERE
SELECT * FROM Products
WHERE Price > (SELECT AVG(Price) FROM Products);

-- In HAVING
SELECT Category, AVG(Price) AS AvgPrice
FROM Products
GROUP BY Category
HAVING AVG(Price) > (SELECT AVG(Price) FROM Products);
```

### Multi-value Subquery (Returns multiple values)

```sql
-- IN operator
SELECT * FROM Customers
WHERE CustomerId IN (
    SELECT CustomerId FROM Orders
    WHERE TotalAmount > 1000
);

-- NOT IN
SELECT * FROM Products
WHERE ProductId NOT IN (
    SELECT ProductId FROM OrderItems
);

-- ANY / SOME
SELECT * FROM Products
WHERE Price > ANY (
    SELECT Price FROM Products WHERE Category = 'Electronics'
);

-- ALL
SELECT * FROM Products
WHERE Price > ALL (
    SELECT Price FROM Products WHERE Category = 'Books'
);
```

### Correlated Subquery (References outer query)

```sql
-- Find products with above-average price in their category
SELECT p1.ProductName, p1.Category, p1.Price
FROM Products p1
WHERE p1.Price > (
    SELECT AVG(p2.Price)
    FROM Products p2
    WHERE p2.Category = p1.Category
);

-- Find customers with above-average order totals
SELECT c.CustomerId, c.FirstName, c.LastName
FROM Customers c
WHERE (
    SELECT AVG(o.TotalAmount)
    FROM Orders o
    WHERE o.CustomerId = c.CustomerId
) > 500;

-- EXISTS
SELECT c.FirstName, c.LastName
FROM Customers c
WHERE EXISTS (
    SELECT 1 FROM Orders o
    WHERE o.CustomerId = c.CustomerId
    AND o.TotalAmount > 1000
);

-- NOT EXISTS
SELECT c.FirstName, c.LastName
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o
    WHERE o.CustomerId = c.CustomerId
);
```

### Derived Table (Subquery in FROM)

```sql
-- Subquery as table
SELECT 
    CategorySales.Category,
    CategorySales.TotalSales,
    CategorySales.AvgPrice
FROM (
    SELECT 
        Category,
        SUM(Price) AS TotalSales,
        AVG(Price) AS AvgPrice
    FROM Products
    GROUP BY Category
) AS CategorySales
WHERE CategorySales.TotalSales > 10000;

-- Multiple derived tables
SELECT 
    cs.Category,
    cs.TotalSales,
    co.OrderCount
FROM (
    SELECT Category, SUM(Price) AS TotalSales
    FROM Products
    GROUP BY Category
) AS cs
INNER JOIN (
    SELECT p.Category, COUNT(*) AS OrderCount
    FROM OrderItems oi
    INNER JOIN Products p ON oi.ProductId = p.ProductId
    GROUP BY p.Category
) AS co ON cs.Category = co.Category;
```

---

## Common Table Expressions (CTEs)

### Basic CTE

```sql
-- Simple CTE
WITH CustomerOrders AS (
    SELECT 
        c.CustomerId,
        c.FirstName,
        c.LastName,
        COUNT(o.OrderId) AS OrderCount,
        SUM(o.TotalAmount) AS TotalSpent
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
    GROUP BY c.CustomerId, c.FirstName, c.LastName
)
SELECT * FROM CustomerOrders
WHERE OrderCount > 5;

-- Multiple CTEs
WITH 
HighValueCustomers AS (
    SELECT CustomerId, FirstName, LastName
    FROM Customers
    WHERE CustomerId IN (
        SELECT CustomerId FROM Orders
        GROUP BY CustomerId
        HAVING SUM(TotalAmount) > 1000
    )
),
RecentOrders AS (
    SELECT OrderId, CustomerId, TotalAmount
    FROM Orders
    WHERE OrderDate >= DATEADD(MONTH, -6, GETDATE())
)
SELECT 
    hvc.FirstName,
    hvc.LastName,
    COUNT(ro.OrderId) AS RecentOrderCount,
    SUM(ro.TotalAmount) AS RecentTotal
FROM HighValueCustomers hvc
LEFT JOIN RecentOrders ro ON hvc.CustomerId = ro.CustomerId
GROUP BY hvc.CustomerId, hvc.FirstName, hvc.LastName;
```

### Recursive CTE

```sql
-- Employee hierarchy (organizational chart)
WITH EmployeeHierarchy AS (
    -- Anchor: Top-level employees (no manager)
    SELECT 
        EmployeeId,
        FirstName,
        LastName,
        ManagerId,
        1 AS Level,
        CAST(FirstName + ' ' + LastName AS NVARCHAR(MAX)) AS Path
    FROM Employees
    WHERE ManagerId IS NULL
    
    UNION ALL
    
    -- Recursive: Employees with managers
    SELECT 
        e.EmployeeId,
        e.FirstName,
        e.LastName,
        e.ManagerId,
        eh.Level + 1,
        CAST(eh.Path + ' -> ' + e.FirstName + ' ' + e.LastName AS NVARCHAR(MAX))
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerId = eh.EmployeeId
)
SELECT * FROM EmployeeHierarchy
ORDER BY Level, Path;

-- Generate number sequence
WITH Numbers AS (
    SELECT 1 AS N
    UNION ALL
    SELECT N + 1
    FROM Numbers
    WHERE N < 100
)
SELECT N FROM Numbers
OPTION (MAXRECURSION 100);

-- Date range generator
WITH DateRange AS (
    SELECT CAST('2025-01-01' AS DATE) AS Date
    UNION ALL
    SELECT DATEADD(DAY, 1, Date)
    FROM DateRange
    WHERE Date < '2025-12-31'
)
SELECT Date FROM DateRange
OPTION (MAXRECURSION 0);  -- 0 = unlimited recursion
```

---

## Window Functions

### ROW_NUMBER - Assign unique row numbers

```sql
-- Basic row numbering
SELECT 
    ProductName,
    Price,
    ROW_NUMBER() OVER (ORDER BY Price DESC) AS RowNum
FROM Products;

-- Partition by category
SELECT 
    Category,
    ProductName,
    Price,
    ROW_NUMBER() OVER (PARTITION BY Category ORDER BY Price DESC) AS RowNum
FROM Products;

-- Get top N per category
WITH RankedProducts AS (
    SELECT 
        Category,
        ProductName,
        Price,
        ROW_NUMBER() OVER (PARTITION BY Category ORDER BY Price DESC) AS RowNum
    FROM Products
)
SELECT * FROM RankedProducts
WHERE RowNum <= 3;

-- Pagination
SELECT 
    CustomerId,
    FirstName,
    LastName,
    ROW_NUMBER() OVER (ORDER BY CustomerId) AS RowNum
FROM Customers
ORDER BY CustomerId
OFFSET 0 ROWS
FETCH NEXT 10 ROWS ONLY;
```

### RANK and DENSE_RANK - Ranking with ties

```sql
-- RANK (gaps in ranking with ties)
SELECT 
    ProductName,
    Price,
    RANK() OVER (ORDER BY Price DESC) AS Rank
FROM Products;
-- Result: 1, 2, 2, 4, 5, ...  (skips 3)

-- DENSE_RANK (no gaps in ranking)
SELECT 
    ProductName,
    Price,
    DENSE_RANK() OVER (ORDER BY Price DESC) AS DenseRank
FROM Products;
-- Result: 1, 2, 2, 3, 4, ...  (no skip)

-- NTILE (divide into N groups)
SELECT 
    ProductName,
    Price,
    NTILE(4) OVER (ORDER BY Price DESC) AS Quartile
FROM Products;
-- Divides into 4 equal groups

-- Comparison
SELECT 
    ProductName,
    Price,
    ROW_NUMBER() OVER (ORDER BY Price DESC) AS RowNum,
    RANK() OVER (ORDER BY Price DESC) AS Rank,
    DENSE_RANK() OVER (ORDER BY Price DESC) AS DenseRank
FROM Products;
```

### Aggregate Window Functions

```sql
-- Running total
SELECT 
    OrderDate,
    TotalAmount,
    SUM(TotalAmount) OVER (ORDER BY OrderDate) AS RunningTotal
FROM Orders;

-- Running average
SELECT 
    OrderDate,
    TotalAmount,
    AVG(TotalAmount) OVER (ORDER BY OrderDate ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS RunningAvg
FROM Orders;

-- Moving average (last 7 days)
SELECT 
    OrderDate,
    TotalAmount,
    AVG(TotalAmount) OVER (
        ORDER BY OrderDate 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS MovingAvg7Day
FROM Orders;

-- Cumulative sum by category
SELECT 
    Category,
    ProductName,
    Price,
    SUM(Price) OVER (PARTITION BY Category ORDER BY ProductName) AS CumulativeSum
FROM Products;
```

### LAG and LEAD - Access previous/next rows

```sql
-- LAG (previous row)
SELECT 
    OrderDate,
    TotalAmount,
    LAG(TotalAmount, 1, 0) OVER (ORDER BY OrderDate) AS PreviousAmount,
    TotalAmount - LAG(TotalAmount, 1, 0) OVER (ORDER BY OrderDate) AS Difference
FROM Orders;

-- LEAD (next row)
SELECT 
    OrderDate,
    TotalAmount,
    LEAD(TotalAmount, 1) OVER (ORDER BY OrderDate) AS NextAmount
FROM Orders;

-- Compare with previous month
WITH MonthlySales AS (
    SELECT 
        YEAR(OrderDate) AS Year,
        MONTH(OrderDate) AS Month,
        SUM(TotalAmount) AS TotalSales
    FROM Orders
    GROUP BY YEAR(OrderDate), MONTH(OrderDate)
)
SELECT 
    Year,
    Month,
    TotalSales,
    LAG(TotalSales) OVER (ORDER BY Year, Month) AS PrevMonthSales,
    TotalSales - LAG(TotalSales) OVER (ORDER BY Year, Month) AS Change,
    CAST((TotalSales - LAG(TotalSales) OVER (ORDER BY Year, Month)) * 100.0 / LAG(TotalSales) OVER (ORDER BY Year, Month) AS DECIMAL(10,2)) AS ChangePercent
FROM MonthlySales;
```

### FIRST_VALUE and LAST_VALUE

```sql
-- First and last value in partition
SELECT 
    Category,
    ProductName,
    Price,
    FIRST_VALUE(ProductName) OVER (PARTITION BY Category ORDER BY Price DESC) AS MostExpensive,
    LAST_VALUE(ProductName) OVER (
        PARTITION BY Category 
        ORDER BY Price DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS LeastExpensive
FROM Products;
```

---

## Aggregate Functions

### Basic Aggregates

```sql
-- COUNT
SELECT COUNT(*) FROM Customers;                    -- All rows
SELECT COUNT(Phone) FROM Customers;                -- Non-NULL values
SELECT COUNT(DISTINCT Country) FROM Customers;     -- Unique values

-- SUM
SELECT SUM(TotalAmount) FROM Orders;
SELECT SUM(Price * Quantity) AS TotalValue FROM OrderItems;

-- AVG
SELECT AVG(Price) FROM Products;
SELECT AVG(CAST(Price AS DECIMAL(10,2))) FROM Products;  -- More precise

-- MIN / MAX
SELECT MIN(Price), MAX(Price) FROM Products;
SELECT MIN(OrderDate), MAX(OrderDate) FROM Orders;

-- Multiple aggregates
SELECT 
    COUNT(*) AS TotalOrders,
    SUM(TotalAmount) AS TotalRevenue,
    AVG(TotalAmount) AS AvgOrderValue,
    MIN(TotalAmount) AS MinOrder,
    MAX(TotalAmount) AS MaxOrder
FROM Orders;
```

### GROUP BY

```sql
-- Basic grouping
SELECT 
    Category,
    COUNT(*) AS ProductCount,
    AVG(Price) AS AvgPrice
FROM Products
GROUP BY Category;

-- Multiple columns
SELECT 
    Country,
    City,
    COUNT(*) AS CustomerCount
FROM Customers
GROUP BY Country, City
ORDER BY Country, City;

-- With calculations
SELECT 
    YEAR(OrderDate) AS Year,
    MONTH(OrderDate) AS Month,
    COUNT(*) AS OrderCount,
    SUM(TotalAmount) AS TotalSales,
    AVG(TotalAmount) AS AvgOrderValue
FROM Orders
GROUP BY YEAR(OrderDate), MONTH(OrderDate)
ORDER BY Year, Month;

-- GROUP BY with JOIN
SELECT 
    c.Country,
    COUNT(o.OrderId) AS OrderCount,
    SUM(o.TotalAmount) AS TotalSpent
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
GROUP BY c.Country;
```

### HAVING - Filter Groups

```sql
-- Filter aggregated results
SELECT 
    Category,
    COUNT(*) AS ProductCount,
    AVG(Price) AS AvgPrice
FROM Products
GROUP BY Category
HAVING COUNT(*) > 5;

-- Multiple HAVING conditions
SELECT 
    Country,
    COUNT(*) AS CustomerCount,
    AVG(OrderCount) AS AvgOrders
FROM (
    SELECT 
        c.Country,
        c.CustomerId,
        COUNT(o.OrderId) AS OrderCount
    FROM Customers c
    LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
    GROUP BY c.Country, c.CustomerId
) AS CustomerOrders
GROUP BY Country
HAVING COUNT(*) > 10 AND AVG(OrderCount) > 2;

-- HAVING vs WHERE
SELECT Category, AVG(Price) AS AvgPrice
FROM Products
WHERE Price > 10          -- Filter BEFORE grouping
GROUP BY Category
HAVING AVG(Price) > 50;   -- Filter AFTER grouping
```

### String Aggregation (SQL Server 2017+)

```sql
-- STRING_AGG
SELECT 
    Category,
    STRING_AGG(ProductName, ', ') AS Products
FROM Products
GROUP BY Category;

-- With ordering
SELECT 
    Category,
    STRING_AGG(ProductName, ', ') WITHIN GROUP (ORDER BY ProductName) AS Products
FROM Products
GROUP BY Category;

-- Older SQL Server (use FOR XML PATH)
SELECT 
    Category,
    STUFF((
        SELECT ', ' + ProductName
        FROM Products p2
        WHERE p2.Category = p1.Category
        ORDER BY ProductName
        FOR XML PATH(''), TYPE
    ).value('.', 'NVARCHAR(MAX)'), 1, 2, '') AS Products
FROM Products p1
GROUP BY Category;
```

---

## Stored Procedures

### Basic Stored Procedure

```sql
-- Create procedure
CREATE PROCEDURE GetCustomerById
    @CustomerId INT
AS
BEGIN
    SELECT * FROM Customers
    WHERE CustomerId = @CustomerId;
END;
GO

-- Execute procedure
EXEC GetCustomerById @CustomerId = 1;
-- Or
EXEC GetCustomerById 1;

-- Drop procedure
DROP PROCEDURE IF EXISTS GetCustomerById;
```

### Procedure with Multiple Parameters

```sql
CREATE PROCEDURE GetCustomersByCountryAndCity
    @Country NVARCHAR(50),
    @City NVARCHAR(50) = NULL  -- Optional parameter with default
AS
BEGIN
    SET NOCOUNT ON;  -- Prevent "rows affected" messages
    
    SELECT 
        CustomerId,
        FirstName,
        LastName,
        Email
    FROM Customers
    WHERE 
        Country = @Country
        AND (@City IS NULL OR City = @City);
END;
GO

-- Usage
EXEC GetCustomersByCountryAndCity @Country = 'USA';
EXEC GetCustomersByCountryAndCity @Country = 'USA', @City = 'New York';
```

### Procedure with OUTPUT Parameters

```sql
CREATE PROCEDURE CreateCustomer
    @FirstName NVARCHAR(50),
    @LastName NVARCHAR(50),
    @Email NVARCHAR(100),
    @CustomerId INT OUTPUT
AS
BEGIN
    INSERT INTO Customers (FirstName, LastName, Email)
    VALUES (@FirstName, @LastName, @Email);
    
    SET @CustomerId = SCOPE_IDENTITY();
END;
GO

-- Usage
DECLARE @NewId INT;
EXEC CreateCustomer 
    @FirstName = 'John',
    @LastName = 'Doe',
    @Email = 'john@example.com',
    @CustomerId = @NewId OUTPUT;
    
SELECT @NewId AS NewCustomerId;
```

### Procedure with Error Handling

```sql
CREATE PROCEDURE UpdateCustomerEmail
    @CustomerId INT,
    @NewEmail NVARCHAR(100)
AS
BEGIN
    SET NOCOUNT ON;
    
    BEGIN TRY
        BEGIN TRANSACTION;
        
        -- Check if customer exists
        IF NOT EXISTS (SELECT 1 FROM Customers WHERE CustomerId = @CustomerId)
        BEGIN
            THROW 50001, 'Customer not found', 1;
        END
        
        -- Check if email already exists
        IF EXISTS (SELECT 1 FROM Customers WHERE Email = @NewEmail AND CustomerId <> @CustomerId)
        BEGIN
            THROW 50002, 'Email already exists', 1;
        END
        
        -- Update email
        UPDATE Customers
        SET Email = @NewEmail, UpdatedAt = GETDATE()
        WHERE CustomerId = @CustomerId;
        
        COMMIT TRANSACTION;
        
        SELECT 'Success' AS Result;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;
        
        -- Return error details
        SELECT 
            ERROR_NUMBER() AS ErrorNumber,
            ERROR_MESSAGE() AS ErrorMessage,
            ERROR_SEVERITY() AS ErrorSeverity,
            ERROR_STATE() AS ErrorState;
    END CATCH
END;
GO
```

### Procedure with Dynamic SQL

```sql
CREATE PROCEDURE SearchProducts
    @SearchTerm NVARCHAR(100),
    @OrderByColumn NVARCHAR(50) = 'ProductName',
    @OrderDirection NVARCHAR(4) = 'ASC'
AS
BEGIN
    SET NOCOUNT ON;
    
    DECLARE @SQL NVARCHAR(MAX);
    
    -- Validate order direction
    IF @OrderDirection NOT IN ('ASC', 'DESC')
        SET @OrderDirection = 'ASC';
    
    -- Build dynamic SQL
    SET @SQL = N'
        SELECT ProductId, ProductName, Category, Price
        FROM Products
        WHERE ProductName LIKE @SearchTerm
        ORDER BY ' + QUOTENAME(@OrderByColumn) + ' ' + @OrderDirection;
    
    -- Execute with parameters
    EXEC sp_executesql 
        @SQL,
        N'@SearchTerm NVARCHAR(100)',
        @SearchTerm = @SearchTerm;
END;
GO

-- Usage
EXEC SearchProducts @SearchTerm = '%Laptop%', @OrderByColumn = 'Price', @OrderDirection = 'DESC';
```

---

## Functions

### Scalar Functions (Return single value)

```sql
-- Create scalar function
CREATE FUNCTION dbo.CalculateTax
(
    @Amount DECIMAL(10,2),
    @TaxRate DECIMAL(5,4)
)
RETURNS DECIMAL(10,2)
AS
BEGIN
    RETURN @Amount * @TaxRate;
END;
GO

-- Usage
SELECT 
    ProductName,
    Price,
    dbo.CalculateTax(Price, 0.10) AS Tax,
    Price + dbo.CalculateTax(Price, 0.10) AS TotalPrice
FROM Products;

-- Another example
CREATE FUNCTION dbo.GetFullName
(
    @FirstName NVARCHAR(50),
    @LastName NVARCHAR(50)
)
RETURNS NVARCHAR(101)
AS
BEGIN
    RETURN @FirstName + ' ' + @LastName;
END;
GO

SELECT dbo.GetFullName(FirstName, LastName) AS FullName
FROM Customers;
```

### Table-Valued Functions (Return table)

```sql
-- Inline table-valued function (preferred)
CREATE FUNCTION dbo.GetCustomerOrders
(
    @CustomerId INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        o.OrderId,
        o.OrderDate,
        o.TotalAmount,
        o.Status
    FROM Orders o
    WHERE o.CustomerId = @CustomerId
);
GO

-- Usage
SELECT * FROM dbo.GetCustomerOrders(1);

-- With JOIN
SELECT 
    c.FirstName,
    c.LastName,
    co.OrderId,
    co.TotalAmount
FROM Customers c
CROSS APPLY dbo.GetCustomerOrders(c.CustomerId) co;

-- Multi-statement table-valued function
CREATE FUNCTION dbo.GetTopCustomers
(
    @TopN INT
)
RETURNS @TopCustomers TABLE
(
    CustomerId INT,
    FullName NVARCHAR(101),
    TotalSpent DECIMAL(10,2),
    OrderCount INT
)
AS
BEGIN
    INSERT INTO @TopCustomers
    SELECT TOP (@TopN)
        c.CustomerId,
        c.FirstName + ' ' + c.LastName AS FullName,
        SUM(o.TotalAmount) AS TotalSpent,
        COUNT(o.OrderId) AS OrderCount
    FROM Customers c
    INNER JOIN Orders o ON c.CustomerId = o.CustomerId
    GROUP BY c.CustomerId, c.FirstName, c.LastName
    ORDER BY SUM(o.TotalAmount) DESC;
    
    RETURN;
END;
GO

SELECT * FROM dbo.GetTopCustomers(10);
```

---

## Triggers

### AFTER Triggers (Fire after operation)

```sql
-- Audit trigger (track changes)
CREATE TABLE CustomerAudit (
    AuditId INT IDENTITY(1,1) PRIMARY KEY,
    CustomerId INT,
    Action NVARCHAR(10),
    OldValue NVARCHAR(MAX),
    NewValue NVARCHAR(MAX),
    ChangedBy NVARCHAR(100) DEFAULT SYSTEM_USER,
    ChangedAt DATETIME2 DEFAULT GETDATE()
);
GO

CREATE TRIGGER trg_Customer_Audit
ON Customers
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    SET NOCOUNT ON;
    
    -- INSERT
    IF EXISTS (SELECT * FROM INSERTED) AND NOT EXISTS (SELECT * FROM DELETED)
    BEGIN
        INSERT INTO CustomerAudit (CustomerId, Action, NewValue)
        SELECT 
            CustomerId,
            'INSERT',
            FirstName + ' ' + LastName + ' (' + Email + ')'
        FROM INSERTED;
    END
    
    -- UPDATE
    IF EXISTS (SELECT * FROM INSERTED) AND EXISTS (SELECT * FROM DELETED)
    BEGIN
        INSERT INTO CustomerAudit (CustomerId, Action, OldValue, NewValue)
        SELECT 
            i.CustomerId,
            'UPDATE',
            d.FirstName + ' ' + d.LastName + ' (' + d.Email + ')',
            i.FirstName + ' ' + i.LastName + ' (' + i.Email + ')'
        FROM INSERTED i
        INNER JOIN DELETED d ON i.CustomerId = d.CustomerId;
    END
    
    -- DELETE
    IF NOT EXISTS (SELECT * FROM INSERTED) AND EXISTS (SELECT * FROM DELETED)
    BEGIN
        INSERT INTO CustomerAudit (CustomerId, Action, OldValue)
        SELECT 
            CustomerId,
            'DELETE',
            FirstName + ' ' + LastName + ' (' + Email + ')'
        FROM DELETED;
    END
END;
GO
```

### INSTEAD OF Triggers (Override operation)

```sql
-- Prevent deletion of certain records
CREATE TRIGGER trg_Customer_PreventDelete
ON Customers
INSTEAD OF DELETE
AS
BEGIN
    SET NOCOUNT ON;
    
    -- Check if any customers have orders
    IF EXISTS (
        SELECT 1 FROM DELETED d
        INNER JOIN Orders o ON d.CustomerId = o.CustomerId
    )
    BEGIN
        THROW 50003, 'Cannot delete customers with existing orders', 1;
        ROLLBACK;
    END
    ELSE
    BEGIN
        -- Perform actual deletion
        DELETE FROM Customers
        WHERE CustomerId IN (SELECT CustomerId FROM DELETED);
    END
END;
GO
```

### Trigger Management

```sql
-- Disable trigger
DISABLE TRIGGER trg_Customer_Audit ON Customers;

-- Enable trigger
ENABLE TRIGGER trg_Customer_Audit ON Customers;

-- Drop trigger
DROP TRIGGER IF EXISTS trg_Customer_Audit;

-- View all triggers
SELECT 
    t.name AS TriggerName,
    OBJECT_NAME(t.parent_id) AS TableName,
    t.is_disabled
FROM sys.triggers t
WHERE t.parent_class = 1;  -- Object triggers (not database triggers)
```

---

## Transactions and Locking

### Basic Transactions

```sql
-- Simple transaction
BEGIN TRANSACTION;

UPDATE Customers SET Email = 'newemail@example.com' WHERE CustomerId = 1;
UPDATE Orders SET Status = 'Completed' WHERE OrderId = 100;

COMMIT TRANSACTION;  -- Save changes
-- Or
ROLLBACK TRANSACTION;  -- Undo changes
```

### Transaction with Error Handling

```sql
BEGIN TRY
    BEGIN TRANSACTION;
    
    -- Transfer money between accounts
    UPDATE Accounts SET Balance = Balance - 100 WHERE AccountId = 1;
    UPDATE Accounts SET Balance = Balance + 100 WHERE AccountId = 2;
    
    -- Simulate error
    -- THROW 50000, 'Simulated error', 1;
    
    COMMIT TRANSACTION;
    SELECT 'Transaction completed successfully' AS Result;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;
    
    SELECT 
        ERROR_MESSAGE() AS ErrorMessage,
        ERROR_NUMBER() AS ErrorNumber;
END CATCH;
```

### Savepoints

```sql
BEGIN TRANSACTION;

INSERT INTO Customers (FirstName, LastName, Email)
VALUES ('John', 'Doe', 'john@example.com');

SAVE TRANSACTION SavePoint1;

INSERT INTO Orders (CustomerId, TotalAmount)
VALUES (1, 100.00);

-- Rollback to savepoint (keeps first INSERT)
ROLLBACK TRANSACTION SavePoint1;

COMMIT TRANSACTION;
```

### Isolation Levels

```sql
-- READ UNCOMMITTED (Dirty reads possible)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
BEGIN TRANSACTION;
SELECT * FROM Products;
COMMIT;

-- READ COMMITTED (Default, no dirty reads)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- REPEATABLE READ (No dirty or non-repeatable reads)
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- SERIALIZABLE (Highest isolation, no phantom reads)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- SNAPSHOT (Uses row versioning)
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;

-- Query-level hints
SELECT * FROM Products WITH (NOLOCK);        -- READ UNCOMMITTED
SELECT * FROM Products WITH (READCOMMITTED); -- READ COMMITTED
SELECT * FROM Products WITH (REPEATABLEREAD);-- REPEATABLE READ
SELECT * FROM Products WITH (SERIALIZABLE);  -- SERIALIZABLE
```

### Lock Types

```sql
-- Exclusive lock (for updates)
SELECT * FROM Products WITH (XLOCK);

-- Shared lock (for reads)
SELECT * FROM Products WITH (HOLDLOCK);

-- Update lock (prevents deadlocks)
SELECT * FROM Products WITH (UPDLOCK);

-- Row lock (lock specific rows)
SELECT * FROM Products WITH (ROWLOCK) WHERE ProductId = 1;

-- Page lock
SELECT * FROM Products WITH (PAGLOCK);

-- Table lock
SELECT * FROM Products WITH (TABLOCK);
```

---

## Indexes (Brief Overview)

### Index Types

```sql
-- Clustered index (one per table, defines physical order)
CREATE CLUSTERED INDEX IX_Customers_CustomerId 
ON Customers(CustomerId);

-- Non-clustered index (multiple allowed, separate structure)
CREATE NONCLUSTERED INDEX IX_Customers_Email 
ON Customers(Email);

-- Unique index (enforce uniqueness)
CREATE UNIQUE INDEX IX_Customers_Email_Unique 
ON Customers(Email);

-- Composite index (multiple columns)
CREATE INDEX IX_Orders_CustomerId_OrderDate 
ON Orders(CustomerId, OrderDate);

-- Covering index (include non-key columns)
CREATE INDEX IX_Orders_CustomerId_Include 
ON Orders(CustomerId)
INCLUDE (OrderDate, TotalAmount);

-- Filtered index (subset of rows)
CREATE INDEX IX_Orders_ActiveOrders 
ON Orders(OrderDate)
WHERE Status = 'Active';

-- Drop index
DROP INDEX IX_Customers_Email ON Customers;
```

---

## Built-in Functions

### String Functions

```sql
-- LEN, DATALENGTH
SELECT LEN('Hello');           -- 5
SELECT DATALENGTH('Hello');    -- 5 (or 10 for NVARCHAR)

-- UPPER, LOWER
SELECT UPPER('hello');         -- HELLO
SELECT LOWER('HELLO');         -- hello

-- LTRIM, RTRIM, TRIM
SELECT LTRIM('  Hello  ');     -- 'Hello  '
SELECT RTRIM('  Hello  ');     -- '  Hello'
SELECT TRIM('  Hello  ');      -- 'Hello' (SQL Server 2017+)

-- SUBSTRING
SELECT SUBSTRING('Hello World', 1, 5);  -- 'Hello'
SELECT SUBSTRING('Hello World', 7, 5);  -- 'World'

-- LEFT, RIGHT
SELECT LEFT('Hello World', 5);   -- 'Hello'
SELECT RIGHT('Hello World', 5);  -- 'World'

-- CHARINDEX, PATINDEX
SELECT CHARINDEX('World', 'Hello World');  -- 7
SELECT PATINDEX('%World%', 'Hello World'); -- 7

-- REPLACE
SELECT REPLACE('Hello World', 'World', 'SQL');  -- 'Hello SQL'

-- REPLICATE
SELECT REPLICATE('*', 5);  -- '*****'

-- REVERSE
SELECT REVERSE('Hello');   -- 'olleH'

-- CONCAT, CONCAT_WS
SELECT CONCAT('Hello', ' ', 'World');           -- 'Hello World'
SELECT CONCAT_WS(', ', 'John', 'Doe', 'USA');  -- 'John, Doe, USA'

-- STRING_SPLIT (SQL Server 2016+)
SELECT value FROM STRING_SPLIT('Apple,Orange,Banana', ',');
-- Result:
-- Apple
-- Orange
-- Banana

-- FORMAT
SELECT FORMAT(1234567.89, 'N2');  -- 1,234,567.89
SELECT FORMAT(GETDATE(), 'yyyy-MM-dd');  -- 2025-01-16
```

### Date Functions

```sql
-- GETDATE, GETUTCDATE
SELECT GETDATE();       -- Current local datetime
SELECT GETUTCDATE();    -- Current UTC datetime
SELECT SYSDATETIME();   -- Current datetime (more precise)

-- DATEPART, YEAR, MONTH, DAY
SELECT DATEPART(YEAR, GETDATE());   -- 2025
SELECT YEAR(GETDATE());             -- 2025
SELECT MONTH(GETDATE());            -- 1
SELECT DAY(GETDATE());              -- 16

-- DATEADD
SELECT DATEADD(DAY, 7, GETDATE());      -- Add 7 days
SELECT DATEADD(MONTH, 1, GETDATE());    -- Add 1 month
SELECT DATEADD(YEAR, -1, GETDATE());    -- Subtract 1 year

-- DATEDIFF
SELECT DATEDIFF(DAY, '2025-01-01', '2025-01-16');     -- 15
SELECT DATEDIFF(MONTH, '2024-01-01', '2025-01-01');   -- 12
SELECT DATEDIFF(YEAR, '2020-01-01', '2025-01-01');    -- 5

-- EOMONTH (end of month)
SELECT EOMONTH(GETDATE());          -- 2025-01-31
SELECT EOMONTH(GETDATE(), 1);       -- Next month end

-- DATEFROMPARTS
SELECT DATEFROMPARTS(2025, 1, 16);  -- 2025-01-16

-- FORMAT
SELECT FORMAT(GETDATE(), 'yyyy-MM-dd');          -- 2025-01-16
SELECT FORMAT(GETDATE(), 'dd/MM/yyyy');          -- 16/01/2025
SELECT FORMAT(GETDATE(), 'MMMM dd, yyyy');       -- January 16, 2025
```

### Conversion Functions

```sql
-- CAST
SELECT CAST('123' AS INT);               -- 123
SELECT CAST(123.456 AS DECIMAL(10,2));   -- 123.46
SELECT CAST(GETDATE() AS DATE);          -- 2025-01-16

-- CONVERT (with style)
SELECT CONVERT(INT, '123');                     -- 123
SELECT CONVERT(VARCHAR(10), GETDATE(), 101);    -- 01/16/2025
SELECT CONVERT(VARCHAR(10), GETDATE(), 103);    -- 16/01/2025
SELECT CONVERT(VARCHAR(20), GETDATE(), 120);    -- 2025-01-16 14:30:00

-- TRY_CAST, TRY_CONVERT (return NULL on error)
SELECT TRY_CAST('abc' AS INT);          -- NULL (instead of error)
SELECT TRY_CONVERT(INT, 'abc');         -- NULL

-- STR (number to string)
SELECT STR(123.456, 10, 2);             -- '    123.46'
```

### Math Functions

```sql
-- Basic
SELECT ABS(-5);          -- 5
SELECT CEILING(4.3);     -- 5
SELECT FLOOR(4.7);       -- 4
SELECT ROUND(4.567, 2);  -- 4.570
SELECT POWER(2, 3);      -- 8
SELECT SQRT(16);         -- 4
SELECT SQUARE(4);        -- 16

-- Aggregate
SELECT SUM(Price) FROM Products;
SELECT AVG(Price) FROM Products;
SELECT MIN(Price) FROM Products;
SELECT MAX(Price) FROM Products;
SELECT COUNT(*) FROM Products;
```

### NULL Handling

```sql
-- ISNULL (SQL Server specific)
SELECT ISNULL(Phone, 'No phone') FROM Customers;

-- COALESCE (ANSI standard, accepts multiple values)
SELECT COALESCE(Phone, Mobile, Email, 'No contact') FROM Customers;

-- NULLIF (returns NULL if equal)
SELECT NULLIF(Price, 0) FROM Products;  -- Returns NULL if Price = 0

-- IIF (inline IF)
SELECT IIF(Price > 100, 'Expensive', 'Affordable') FROM Products;

-- CASE expression
SELECT 
    ProductName,
    CASE 
        WHEN Price < 50 THEN 'Budget'
        WHEN Price < 100 THEN 'Mid-range'
        ELSE 'Premium'
    END AS PriceCategory
FROM Products;

-- CASE with multiple conditions
SELECT 
    FirstName,
    LastName,
    CASE 
        WHEN Country = 'USA' AND City = 'New York' THEN 'NY Customer'
        WHEN Country = 'USA' THEN 'US Customer'
        WHEN Country IN ('UK', 'Canada') THEN 'International'
        ELSE 'Other'
    END AS CustomerType
FROM Customers;
```

### JSON Functions (SQL Server 2016+)

```sql
-- FOR JSON (convert to JSON)
SELECT CustomerId, FirstName, LastName
FROM Customers
WHERE Country = 'USA'
FOR JSON PATH;

-- OPENJSON (parse JSON)
DECLARE @json NVARCHAR(MAX) = N'{"name":"John","age":30,"city":"New York"}';
SELECT * FROM OPENJSON(@json);

-- JSON_VALUE (extract scalar value)
DECLARE @json NVARCHAR(MAX) = N'{"name":"John","age":30}';
SELECT JSON_VALUE(@json, '$.name');  -- John
SELECT JSON_VALUE(@json, '$.age');   -- 30

-- JSON_QUERY (extract object or array)
DECLARE @json NVARCHAR(MAX) = N'{"person":{"name":"John","age":30}}';
SELECT JSON_QUERY(@json, '$.person');  -- {"name":"John","age":30}

-- ISJSON (validate JSON)
SELECT ISJSON('{"name":"John"}');  -- 1 (valid)
SELECT ISJSON('{invalid}');        -- 0 (invalid)

-- JSON_MODIFY (modify JSON)
DECLARE @json NVARCHAR(MAX) = N'{"name":"John","age":30}';
SET @json = JSON_MODIFY(@json, '$.age', 31);
SELECT @json;  -- {"name":"John","age":31}
```

---

## Common Table Patterns

### Pagination

```sql
-- OFFSET-FETCH (recommended)
DECLARE @PageNumber INT = 2;
DECLARE @PageSize INT = 10;

SELECT *
FROM Products
ORDER BY ProductId
OFFSET (@PageNumber - 1) * @PageSize ROWS
FETCH NEXT @PageSize ROWS ONLY;

-- ROW_NUMBER (older approach)
WITH PagedResults AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (ORDER BY ProductId) AS RowNum
    FROM Products
)
SELECT *
FROM PagedResults
WHERE RowNum BETWEEN 11 AND 20;
```

### Deduplication

```sql
-- Remove duplicates (keep first)
WITH DuplicateCustomers AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (PARTITION BY Email ORDER BY CustomerId) AS RowNum
    FROM Customers
)
DELETE FROM DuplicateCustomers
WHERE RowNum > 1;

-- Find duplicates
SELECT Email, COUNT(*) AS DuplicateCount
FROM Customers
GROUP BY Email
HAVING COUNT(*) > 1;
```

### Running Totals

```sql
-- Running total
SELECT 
    OrderDate,
    TotalAmount,
    SUM(TotalAmount) OVER (ORDER BY OrderDate) AS RunningTotal
FROM Orders;

-- Running total by category
SELECT 
    Category,
    ProductName,
    Price,
    SUM(Price) OVER (PARTITION BY Category ORDER BY ProductName) AS RunningTotalInCategory
FROM Products;
```

### Pivot and Unpivot

```sql
-- Sample data
CREATE TABLE Sales (
    Year INT,
    Quarter INT,
    Amount DECIMAL(10,2)
);

INSERT INTO Sales VALUES 
(2024, 1, 100), (2024, 2, 150), (2024, 3, 200), (2024, 4, 250),
(2025, 1, 120), (2025, 2, 180), (2025, 3, 220), (2025, 4, 280);

-- PIVOT (rows to columns)
SELECT *
FROM (
    SELECT Year, Quarter, Amount
    FROM Sales
) AS SourceTable
PIVOT (
    SUM(Amount)
    FOR Quarter IN ([1], [2], [3], [4])
) AS PivotTable;
-- Result:
-- Year  1     2     3     4
-- 2024  100   150   200   250
-- 2025  120   180   220   280

-- UNPIVOT (columns to rows)
SELECT Year, Quarter, Amount
FROM (
    SELECT Year, [1], [2], [3], [4]
    FROM (
        SELECT Year, Quarter, Amount
        FROM Sales
    ) AS SourceTable
    PIVOT (
        SUM(Amount)
        FOR Quarter IN ([1], [2], [3], [4])
    ) AS PivotTable
) AS PivotedData
UNPIVOT (
    Amount FOR Quarter IN ([1], [2], [3], [4])
) AS UnpivotTable;
```

### Merge (Upsert)

```sql
-- MERGE statement
MERGE INTO Customers AS Target
USING (
    SELECT CustomerId, FirstName, LastName, Email
    FROM CustomersStaging
) AS Source
ON Target.CustomerId = Source.CustomerId
WHEN MATCHED THEN
    UPDATE SET 
        Target.FirstName = Source.FirstName,
        Target.LastName = Source.LastName,
        Target.Email = Source.Email,
        Target.UpdatedAt = GETDATE()
WHEN NOT MATCHED BY TARGET THEN
    INSERT (CustomerId, FirstName, LastName, Email, CreatedAt)
    VALUES (Source.CustomerId, Source.FirstName, Source.LastName, Source.Email, GETDATE())
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;

-- Alternative: Separate INSERT/UPDATE
IF EXISTS (SELECT 1 FROM Customers WHERE CustomerId = @CustomerId)
    UPDATE Customers SET FirstName = @FirstName WHERE CustomerId = @CustomerId;
ELSE
    INSERT INTO Customers (FirstName, LastName) VALUES (@FirstName, @LastName);
```

---

## Best Practices

### 1. Always Use WHERE in UPDATE/DELETE

```sql
-- ❌ Dangerous (updates ALL rows)
UPDATE Customers SET IsActive = 0;

-- ✅ Safe (updates specific rows)
UPDATE Customers SET IsActive = 0 WHERE CustomerId = 1;

-- ✅ Test with SELECT first
SELECT * FROM Customers WHERE CustomerId = 1;
-- If correct, convert to UPDATE
UPDATE Customers SET IsActive = 0 WHERE CustomerId = 1;
```

### 2. Use Parameterized Queries (Prevent SQL Injection)

```csharp
// ❌ Vulnerable to SQL injection
string sql = $"SELECT * FROM Users WHERE Email = '{email}'";

// ✅ Parameterized (safe)
string sql = "SELECT * FROM Users WHERE Email = @Email";
command.Parameters.AddWithValue("@Email", email);
```

### 3. Use Appropriate Data Types

```sql
-- ❌ Wastes space
CREATE TABLE Products (
    ProductId VARCHAR(255),
    Price VARCHAR(50)
);

-- ✅ Appropriate types
CREATE TABLE Products (
    ProductId INT,
    Price DECIMAL(10,2)
);
```

### 4. Use NVARCHAR for Unicode

```sql
-- ❌ Cannot store Unicode characters
CREATE TABLE Customers (
    Name VARCHAR(50)
);

-- ✅ Supports all languages
CREATE TABLE Customers (
    Name NVARCHAR(50)
);
```

### 5. Add Proper Constraints

```sql
-- ✅ Good table design
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY IDENTITY(1,1),
    CustomerId INT NOT NULL,
    OrderDate DATETIME2 DEFAULT GETDATE(),
    TotalAmount DECIMAL(10,2) CHECK (TotalAmount >= 0),
    Status NVARCHAR(20) DEFAULT 'Pending' CHECK (Status IN ('Pending', 'Completed', 'Cancelled')),
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
);
```

### 6. Use Transactions for Multiple Operations

```sql
-- ✅ Atomic operation
BEGIN TRY
    BEGIN TRANSACTION;
    
    UPDATE Accounts SET Balance = Balance - 100 WHERE AccountId = 1;
    UPDATE Accounts SET Balance = Balance + 100 WHERE AccountId = 2;
    
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    THROW;
END CATCH;
```

### 7. Use EXISTS Instead of COUNT(*) > 0

```sql
-- ❌ Slower (counts all)
IF (SELECT COUNT(*) FROM Orders WHERE CustomerId = 1) > 0
    PRINT 'Has orders';

-- ✅ Faster (stops at first match)
IF EXISTS (SELECT 1 FROM Orders WHERE CustomerId = 1)
    PRINT 'Has orders';
```

### 8. Use SET NOCOUNT ON in Procedures

```sql
CREATE PROCEDURE GetCustomers
AS
BEGIN
    SET NOCOUNT ON;  -- ✅ Prevents "rows affected" messages
    
    SELECT * FROM Customers;
END;
```

### 9. Use Schema Names

```sql
-- ❌ Ambiguous
SELECT * FROM Customers;

-- ✅ Explicit schema
SELECT * FROM dbo.Customers;
```

### 10. Comment Complex Queries

```sql
-- ✅ Good practice
-- Get top 10 customers by total spend in 2024
-- Includes only completed orders
SELECT TOP 10
    c.CustomerId,
    c.FirstName + ' ' + c.LastName AS CustomerName,
    SUM(o.TotalAmount) AS TotalSpent,
    COUNT(o.OrderId) AS OrderCount
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE 
    o.Status = 'Completed'
    AND YEAR(o.OrderDate) = 2024
GROUP BY c.CustomerId, c.FirstName, c.LastName
ORDER BY TotalSpent DESC;
```

---

## Common Pitfalls

### 1. SELECT *

```sql
-- ❌ Don't use SELECT *
SELECT * FROM Customers;

-- ✅ Select only needed columns
SELECT CustomerId, FirstName, LastName, Email FROM Customers;

-- Why? SELECT * is:
-- - Slower (transfers unnecessary data)
-- - Breaks when columns change
-- - Unclear what data is needed
```

### 2. Not Using WHERE in DELETE/UPDATE

```sql
-- ❌ Deletes ALL customers
DELETE FROM Customers;

-- ✅ Delete specific customers
DELETE FROM Customers WHERE IsActive = 0 AND LastLoginDate < '2023-01-01';
```

### 3. NOLOCK Misuse

```sql
-- ❌ Overused (can read uncommitted data)
SELECT * FROM Products WITH (NOLOCK);

-- ✅ Use only when dirty reads are acceptable
-- Consider READ COMMITTED SNAPSHOT instead
```

### 4. Implicit Conversion

```sql
-- ❌ Implicit conversion (slow)
CREATE TABLE Products (ProductCode VARCHAR(20));
SELECT * FROM Products WHERE ProductCode = 123;  -- VARCHAR to INT

-- ✅ Explicit conversion
SELECT * FROM Products WHERE ProductCode = '123';
```

### 5. Functions in WHERE (Non-Sargable)

```sql
-- ❌ Can't use index
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2024;

-- ✅ Sargable (can use index)
SELECT * FROM Orders WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';
```

### 6. Not Handling NULLs

```sql
-- ❌ NULL comparison
SELECT * FROM Customers WHERE Phone = NULL;  -- Always returns nothing

-- ✅ Correct NULL check
SELECT * FROM Customers WHERE Phone IS NULL;
```

### 7. Cursor Overuse

```sql
-- ❌ Slow cursor approach
DECLARE @CustomerId INT;
DECLARE customer_cursor CURSOR FOR 
    SELECT CustomerId FROM Customers;
OPEN customer_cursor;
FETCH NEXT FROM customer_cursor INTO @CustomerId;
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Process each customer
    FETCH NEXT FROM customer_cursor INTO @CustomerId;
END;
CLOSE customer_cursor;
DEALLOCATE customer_cursor;

-- ✅ Set-based approach (much faster)
UPDATE Customers SET IsActive = 1 WHERE LastLoginDate > '2024-01-01';
```

---

## Quick Reference Tables

### JOIN Types Comparison

| Join Type | Description | Returns |
|-----------|-------------|---------|
| INNER JOIN | Matching rows only | Rows that exist in both tables |
| LEFT JOIN | All from left + matching | All left rows + matching right (NULL if no match) |
| RIGHT JOIN | All from right + matching | All right rows + matching left (NULL if no match) |
| FULL JOIN | All from both | All rows from both (NULL where no match) |
| CROSS JOIN | Cartesian product | Every combination of rows |

### Aggregate Functions

| Function | Purpose | Example |
|----------|---------|---------|
| COUNT(*) | Count all rows | `COUNT(*) = 100` |
| COUNT(col) | Count non-NULL | `COUNT(Phone)` |
| SUM(col) | Sum values | `SUM(Price)` |
| AVG(col) | Average | `AVG(Price)` |
| MIN(col) | Minimum | `MIN(Price)` |
| MAX(col) | Maximum | `MAX(Price)` |
| STRING_AGG | Concatenate | `STRING_AGG(Name, ', ')` |

### Window Functions

| Function | Purpose | Example |
|----------|---------|---------|
| ROW_NUMBER() | Unique row number | `ROW_NUMBER() OVER (ORDER BY Price)` |
| RANK() | Rank with gaps | `RANK() OVER (ORDER BY Price DESC)` |
| DENSE_RANK() | Rank no gaps | `DENSE_RANK() OVER (ORDER BY Price DESC)` |
| NTILE(n) | Divide into n groups | `NTILE(4) OVER (ORDER BY Price)` |
| LAG() | Previous row | `LAG(Price, 1) OVER (ORDER BY Date)` |
| LEAD() | Next row | `LEAD(Price, 1) OVER (ORDER BY Date)` |
| SUM() OVER | Running total | `SUM(Amount) OVER (ORDER BY Date)` |

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|-----------|---------------------|--------------|-------------|
| READ UNCOMMITTED | ✅ Yes | ✅ Yes | ✅ Yes | ⚡ Fastest |
| READ COMMITTED (Default) | ❌ No | ✅ Yes | ✅ Yes | ⚡⚡ Fast |
| REPEATABLE READ | ❌ No | ❌ No | ✅ Yes | ⚡ Slower |
| SERIALIZABLE | ❌ No | ❌ No | ❌ No | 🐌 Slowest |
| SNAPSHOT | ❌ No | ❌ No | ❌ No | ⚡⚡ Fast (versions) |

### Constraint Types

| Constraint | Purpose | Example |
|------------|---------|---------|
| PRIMARY KEY | Unique identifier | `CustomerId INT PRIMARY KEY` |
| FOREIGN KEY | Reference another table | `FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)` |
| UNIQUE | Unique values | `Email NVARCHAR(100) UNIQUE` |
| CHECK | Validate values | `CHECK (Price >= 0)` |
| DEFAULT | Default value | `DEFAULT GETDATE()` |
| NOT NULL | Require value | `FirstName NVARCHAR(50) NOT NULL` |

---

## T-SQL Query Execution Order

```sql
-- Written order:
SELECT      -- 5. Select columns
FROM        -- 1. Get data from tables
JOIN        -- 2. Join tables
WHERE       -- 3. Filter rows
GROUP BY    -- 4. Group rows
HAVING      -- 6. Filter groups
ORDER BY    -- 7. Sort results
OFFSET-FETCH -- 8. Limit results

-- Actual execution order:
1. FROM + JOIN    -- Get and combine data
2. WHERE          -- Filter rows
3. GROUP BY       -- Group data
4. HAVING         -- Filter groups
5. SELECT         -- Select columns
6. DISTINCT       -- Remove duplicates
7. ORDER BY       -- Sort results
8. OFFSET-FETCH   -- Limit results
```

---

## Decision Tree: What Query Should I Write?

```
What do you want to do?

Get data?
  └─> SELECT

Add data?
  ├─> Single row → INSERT INTO ... VALUES
  ├─> Multiple rows → INSERT INTO ... VALUES (...), (...)
  └─> From another table → INSERT INTO ... SELECT

Update data?
  ├─> Simple → UPDATE ... SET ... WHERE
  ├─> From another table → UPDATE ... FROM ... JOIN
  └─> All rows → UPDATE ... SET (no WHERE)

Delete data?
  ├─> Specific rows → DELETE FROM ... WHERE
  ├─> All rows (keep structure) → TRUNCATE TABLE
  └─> All rows (can rollback) → DELETE FROM (no WHERE)

Create structure?
  ├─> New table → CREATE TABLE
  ├─> Modify table → ALTER TABLE
  └─> Remove table → DROP TABLE

Combine tables?
  ├─> Matching rows only → INNER JOIN
  ├─> All from left → LEFT JOIN
  ├─> All from right → RIGHT JOIN
  ├─> All from both → FULL OUTER JOIN
  └─> All combinations → CROSS JOIN

Filter results?
  ├─> Before grouping → WHERE
  └─> After grouping → HAVING

Aggregate data?
  ├─> Count → COUNT
  ├─> Sum → SUM
  ├─> Average → AVG
  └─> Group → GROUP BY

Sort results?
  └─> ORDER BY

Limit results?
  ├─> Top N → TOP N or OFFSET-FETCH
  └─> Pagination → OFFSET-FETCH

Complex query?
  ├─> Reusable → CTE (WITH)
  ├─> Recursive → Recursive CTE
  └─> One-time → Subquery
```

---

**Guide Complete!** This comprehensive SQL Server & T-SQL guide covers fundamentals, DDL, DML, joins, subqueries, CTEs, window functions, stored procedures, functions, triggers, transactions, and best practices. Master these concepts and you'll be well-prepared for any backend interview! 🎯
