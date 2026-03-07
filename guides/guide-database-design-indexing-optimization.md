# Database Design, Indexing & Query Optimization Quick Reference

---

## What is Database Design?

**Database Design** = Process of organizing data to reduce redundancy and improve data integrity

**Key Goals:**
- ✅ **Data Integrity** - Accurate and consistent data
- ✅ **Eliminate Redundancy** - Store data only once
- ✅ **Optimize Performance** - Fast queries and efficient storage
- ✅ **Maintainability** - Easy to update and extend
- ✅ **Scalability** - Handle growing data volumes

### Design Process Overview

```
1. Requirements Analysis    → What data do we need?
2. Conceptual Design        → ER Diagram (entities and relationships)
3. Logical Design           → Tables, columns, relationships
4. Normalization            → Reduce redundancy (1NF to 3NF+)
5. Physical Design          → Indexes, partitioning, storage
6. Implementation           → Create tables, constraints, indexes
7. Testing & Optimization   → Query tuning, index optimization
```

---

## Database Normalization

### What is Normalization?

**Normalization** = Process of organizing data to minimize redundancy and dependency

**Benefits:**
- ✅ Eliminates data redundancy
- ✅ Prevents update anomalies
- ✅ Improves data integrity
- ✅ Reduces storage space

**Drawbacks:**
- ❌ More tables = more JOINs
- ❌ Can reduce read performance
- ❌ More complex queries

### Before Normalization Example

```sql
-- ❌ Unnormalized table with redundancy
CREATE TABLE Orders_Unnormalized (
    OrderId INT,
    OrderDate DATE,
    CustomerName VARCHAR(100),
    CustomerEmail VARCHAR(100),
    CustomerPhone VARCHAR(20),
    CustomerAddress VARCHAR(200),
    ProductName VARCHAR(100),
    ProductPrice DECIMAL(10,2),
    Quantity INT
);

-- Problems:
-- 1. Customer data repeated for every order
-- 2. Product data repeated for every order
-- 3. Cannot store customer without order
-- 4. Updating customer info requires updating multiple rows
-- 5. Deleting last order deletes customer data
```

---

## First Normal Form (1NF)

**Rule:** Eliminate repeating groups and ensure atomic values

**Requirements:**
1. Each column contains atomic (indivisible) values
2. Each column contains values of a single type
3. Each column has a unique name
4. Order doesn't matter

### Example: Converting to 1NF

```sql
-- ❌ Violates 1NF (multiple values in single column)
CREATE TABLE Customers_Not1NF (
    CustomerId INT PRIMARY KEY,
    Name VARCHAR(100),
    Phones VARCHAR(200)  -- "555-1234, 555-5678, 555-9012"
);

-- ❌ Violates 1NF (repeating columns)
CREATE TABLE Orders_Not1NF (
    OrderId INT PRIMARY KEY,
    CustomerName VARCHAR(100),
    Product1 VARCHAR(100),
    Product1Price DECIMAL(10,2),
    Product2 VARCHAR(100),
    Product2Price DECIMAL(10,2),
    Product3 VARCHAR(100),
    Product3Price DECIMAL(10,2)
);

-- ✅ 1NF compliant
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE CustomerPhones (
    CustomerId INT,
    PhoneNumber VARCHAR(20),
    PhoneType VARCHAR(20),  -- 'Mobile', 'Home', 'Work'
    PRIMARY KEY (CustomerId, PhoneNumber),
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
);

CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    OrderDate DATE,
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
);

CREATE TABLE OrderItems (
    OrderId INT,
    ProductId INT,
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    PRIMARY KEY (OrderId, ProductId),
    FOREIGN KEY (OrderId) REFERENCES Orders(OrderId)
);
```

---

## Second Normal Form (2NF)

**Rule:** Must be in 1NF AND eliminate partial dependencies

**Partial Dependency:** Non-key column depends on part of a composite primary key

**Requirements:**
1. Must be in 1NF
2. No partial dependencies (all non-key columns depend on the entire primary key)

### Example: Converting to 2NF

```sql
-- ❌ Violates 2NF (partial dependency)
CREATE TABLE OrderItems_Not2NF (
    OrderId INT,
    ProductId INT,
    ProductName VARCHAR(100),      -- Depends only on ProductId
    ProductPrice DECIMAL(10,2),    -- Depends only on ProductId
    Quantity INT,                  -- Depends on both OrderId and ProductId
    PRIMARY KEY (OrderId, ProductId)
);
-- Problem: ProductName and ProductPrice depend only on ProductId, not the full key

-- ✅ 2NF compliant
CREATE TABLE Products (
    ProductId INT PRIMARY KEY,
    ProductName VARCHAR(100),
    ProductPrice DECIMAL(10,2)
);

CREATE TABLE OrderItems (
    OrderId INT,
    ProductId INT,
    Quantity INT,
    PRIMARY KEY (OrderId, ProductId),
    FOREIGN KEY (OrderId) REFERENCES Orders(OrderId),
    FOREIGN KEY (ProductId) REFERENCES Products(ProductId)
);
```

---

## Third Normal Form (3NF)

**Rule:** Must be in 2NF AND eliminate transitive dependencies

**Transitive Dependency:** Non-key column depends on another non-key column

**Requirements:**
1. Must be in 2NF
2. No transitive dependencies (non-key columns depend only on the primary key)

### Example: Converting to 3NF

```sql
-- ❌ Violates 3NF (transitive dependency)
CREATE TABLE Employees_Not3NF (
    EmployeeId INT PRIMARY KEY,
    Name VARCHAR(100),
    DepartmentId INT,
    DepartmentName VARCHAR(100),    -- Depends on DepartmentId (transitive)
    DepartmentLocation VARCHAR(100) -- Depends on DepartmentId (transitive)
);
-- Problem: DepartmentName depends on DepartmentId, not EmployeeId

-- ✅ 3NF compliant
CREATE TABLE Departments (
    DepartmentId INT PRIMARY KEY,
    DepartmentName VARCHAR(100),
    Location VARCHAR(100)
);

CREATE TABLE Employees (
    EmployeeId INT PRIMARY KEY,
    Name VARCHAR(100),
    DepartmentId INT,
    FOREIGN KEY (DepartmentId) REFERENCES Departments(DepartmentId)
);
```

### Real-World 3NF Example

```sql
-- ❌ Not 3NF
CREATE TABLE Orders_Not3NF (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    CustomerName VARCHAR(100),      -- Transitive: depends on CustomerId
    CustomerEmail VARCHAR(100),     -- Transitive: depends on CustomerId
    OrderDate DATE,
    TotalAmount DECIMAL(10,2)
);

-- ✅ 3NF compliant
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY,
    CustomerName VARCHAR(100),
    CustomerEmail VARCHAR(100)
);

CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    OrderDate DATE,
    TotalAmount DECIMAL(10,2),
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
);
```

---

## Boyce-Codd Normal Form (BCNF)

**Rule:** Stronger version of 3NF - every determinant must be a candidate key

**Requirements:**
1. Must be in 3NF
2. For every functional dependency (X → Y), X must be a superkey

### Example: Converting to BCNF

```sql
-- ❌ Violates BCNF (but is in 3NF)
CREATE TABLE CourseInstructor_NotBCNF (
    CourseId INT,
    InstructorId INT,
    InstructorName VARCHAR(100),
    PRIMARY KEY (CourseId, InstructorId)
);
-- Problem: InstructorId → InstructorName, but InstructorId is not a superkey

-- ✅ BCNF compliant
CREATE TABLE Instructors (
    InstructorId INT PRIMARY KEY,
    InstructorName VARCHAR(100)
);

CREATE TABLE CourseInstructor (
    CourseId INT,
    InstructorId INT,
    PRIMARY KEY (CourseId, InstructorId),
    FOREIGN KEY (InstructorId) REFERENCES Instructors(InstructorId)
);
```

---

## Fourth Normal Form (4NF)

**Rule:** Must be in BCNF AND eliminate multi-valued dependencies

**Multi-Valued Dependency:** One attribute determines multiple independent sets of values

### Example: Converting to 4NF

```sql
-- ❌ Violates 4NF (multi-valued dependency)
CREATE TABLE Employee_Skills_Certifications_Not4NF (
    EmployeeId INT,
    Skill VARCHAR(50),
    Certification VARCHAR(50),
    PRIMARY KEY (EmployeeId, Skill, Certification)
);
-- Problem: Skills and Certifications are independent of each other

INSERT INTO Employee_Skills_Certifications_Not4NF VALUES
(1, 'C#', 'Azure Certification'),
(1, 'C#', 'AWS Certification'),  -- Same skill, different cert
(1, 'SQL', 'Azure Certification'), -- Different skill, same cert
(1, 'SQL', 'AWS Certification');   -- All combinations must exist

-- ✅ 4NF compliant
CREATE TABLE EmployeeSkills (
    EmployeeId INT,
    Skill VARCHAR(50),
    PRIMARY KEY (EmployeeId, Skill)
);

CREATE TABLE EmployeeCertifications (
    EmployeeId INT,
    Certification VARCHAR(50),
    PRIMARY KEY (EmployeeId, Certification)
);
```

---

## Fifth Normal Form (5NF)

**Rule:** Must be in 4NF AND eliminate join dependencies

**Join Dependency:** Data can be reconstructed by joining multiple tables

### Example: 5NF

```sql
-- Rarely needed in practice
-- Example: Agent-Company-Product relationship

CREATE TABLE AgentCompany (
    AgentId INT,
    CompanyId INT,
    PRIMARY KEY (AgentId, CompanyId)
);

CREATE TABLE CompanyProduct (
    CompanyId INT,
    ProductId INT,
    PRIMARY KEY (CompanyId, ProductId)
);

CREATE TABLE AgentProduct (
    AgentId INT,
    ProductId INT,
    PRIMARY KEY (AgentId, ProductId)
);
```

---

## Normalization Summary

| Normal Form | Rule | Example Violation |
|-------------|------|-------------------|
| **1NF** | Atomic values, no repeating groups | `Phones: "555-1234, 555-5678"` |
| **2NF** | 1NF + No partial dependencies | ProductName depends only on ProductId in composite key |
| **3NF** | 2NF + No transitive dependencies | DepartmentName depends on DepartmentId, not EmployeeId |
| **BCNF** | 3NF + Every determinant is a candidate key | InstructorId → InstructorName, but InstructorId not a key |
| **4NF** | BCNF + No multi-valued dependencies | Skills and Certifications independent |
| **5NF** | 4NF + No join dependencies | Rare in practice |

### When to Denormalize?

**Denormalization** = Intentionally adding redundancy for performance

**When to consider:**
- ✅ Read-heavy applications (reporting, analytics)
- ✅ Complex joins hurting performance
- ✅ Calculated/aggregated values accessed frequently
- ✅ Immutable historical data

**Common denormalization patterns:**
```sql
-- Adding calculated columns
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    Subtotal DECIMAL(10,2),
    Tax DECIMAL(10,2),
    TotalAmount AS (Subtotal + Tax) PERSISTED  -- Computed column
);

-- Storing aggregate values
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY,
    Name VARCHAR(100),
    TotalOrders INT DEFAULT 0,           -- Denormalized
    TotalSpent DECIMAL(10,2) DEFAULT 0   -- Denormalized
);

-- Maintaining with triggers
CREATE TRIGGER trg_Orders_UpdateCustomerStats
ON Orders
AFTER INSERT
AS
BEGIN
    UPDATE c
    SET 
        c.TotalOrders = c.TotalOrders + 1,
        c.TotalSpent = c.TotalSpent + i.TotalAmount
    FROM Customers c
    INNER JOIN INSERTED i ON c.CustomerId = i.CustomerId;
END;
```

---

## Entity-Relationship (ER) Diagrams

### Relationship Types

#### One-to-One (1:1)

```sql
-- Example: User and UserProfile
CREATE TABLE Users (
    UserId INT PRIMARY KEY,
    Username VARCHAR(50) UNIQUE,
    Email VARCHAR(100)
);

CREATE TABLE UserProfiles (
    UserId INT PRIMARY KEY,  -- Same as foreign key
    Bio NVARCHAR(MAX),
    Avatar VARBINARY(MAX),
    FOREIGN KEY (UserId) REFERENCES Users(UserId)
);

-- Or combined into one table:
CREATE TABLE Users (
    UserId INT PRIMARY KEY,
    Username VARCHAR(50),
    Email VARCHAR(100),
    Bio NVARCHAR(MAX),
    Avatar VARBINARY(MAX)
);
```

#### One-to-Many (1:N)

```sql
-- Example: Customer and Orders
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,  -- Foreign key
    OrderDate DATE,
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
);

-- One customer can have many orders
-- Each order belongs to one customer
```

#### Many-to-Many (M:N)

```sql
-- Example: Students and Courses (requires junction table)
CREATE TABLE Students (
    StudentId INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Courses (
    CourseId INT PRIMARY KEY,
    CourseName VARCHAR(100)
);

-- Junction/Bridge/Link table
CREATE TABLE StudentCourses (
    StudentId INT,
    CourseId INT,
    EnrollmentDate DATE,
    Grade VARCHAR(2),
    PRIMARY KEY (StudentId, CourseId),
    FOREIGN KEY (StudentId) REFERENCES Students(StudentId),
    FOREIGN KEY (CourseId) REFERENCES Courses(CourseId)
);

-- Many students can enroll in many courses
```

### Self-Referencing Relationships

```sql
-- Employee hierarchy (employee reports to manager)
CREATE TABLE Employees (
    EmployeeId INT PRIMARY KEY,
    Name VARCHAR(100),
    ManagerId INT NULL,  -- Self-reference
    FOREIGN KEY (ManagerId) REFERENCES Employees(EmployeeId)
);

-- Category hierarchy (subcategories)
CREATE TABLE Categories (
    CategoryId INT PRIMARY KEY,
    CategoryName VARCHAR(100),
    ParentCategoryId INT NULL,  -- Self-reference
    FOREIGN KEY (ParentCategoryId) REFERENCES Categories(CategoryId)
);
```

---

## Constraints and Data Integrity

### Primary Keys

```sql
-- Single column primary key
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY IDENTITY(1,1)
);

-- Composite primary key
CREATE TABLE OrderItems (
    OrderId INT,
    ProductId INT,
    PRIMARY KEY (OrderId, ProductId)
);

-- Named primary key constraint
CREATE TABLE Products (
    ProductId INT,
    CONSTRAINT PK_Products PRIMARY KEY (ProductId)
);
```

### Foreign Keys

```sql
-- Basic foreign key
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
);

-- Named foreign key with cascading
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    CONSTRAINT FK_Orders_Customers FOREIGN KEY (CustomerId) 
        REFERENCES Customers(CustomerId)
        ON DELETE CASCADE      -- Delete orders when customer is deleted
        ON UPDATE CASCADE      -- Update orders when customer ID changes
);

-- Other referential actions:
-- ON DELETE NO ACTION (default) - Prevent deletion if referenced
-- ON DELETE SET NULL            - Set foreign key to NULL
-- ON DELETE SET DEFAULT         - Set foreign key to default value
```

### Unique Constraints

```sql
-- Single column unique
CREATE TABLE Users (
    UserId INT PRIMARY KEY,
    Email VARCHAR(100) UNIQUE
);

-- Multiple columns unique (together)
CREATE TABLE Students (
    StudentId INT PRIMARY KEY,
    StudentNumber VARCHAR(20),
    Year INT,
    CONSTRAINT UQ_Students_Number_Year UNIQUE (StudentNumber, Year)
);

-- Unique with NULL handling (multiple NULLs allowed)
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY,
    Email VARCHAR(100),
    Phone VARCHAR(20),
    CONSTRAINT UQ_Customers_Email UNIQUE (Email),
    -- Note: Multiple rows can have NULL in Email (SQL Server behavior)
);
```

### Check Constraints

```sql
-- Simple check
CREATE TABLE Products (
    ProductId INT PRIMARY KEY,
    Price DECIMAL(10,2) CHECK (Price >= 0)
);

-- Multiple conditions
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    OrderDate DATE,
    ShipDate DATE,
    Status VARCHAR(20),
    CONSTRAINT CHK_Orders_Status 
        CHECK (Status IN ('Pending', 'Processing', 'Shipped', 'Delivered', 'Cancelled')),
    CONSTRAINT CHK_Orders_Dates 
        CHECK (ShipDate IS NULL OR ShipDate >= OrderDate)
);

-- Expression-based check
CREATE TABLE Employees (
    EmployeeId INT PRIMARY KEY,
    BirthDate DATE,
    HireDate DATE,
    CONSTRAINT CHK_Employees_Age 
        CHECK (DATEDIFF(YEAR, BirthDate, HireDate) >= 18)
);
```

### Default Constraints

```sql
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    OrderDate DATETIME2 DEFAULT GETDATE(),
    Status VARCHAR(20) DEFAULT 'Pending',
    IsActive BIT DEFAULT 1,
    CreatedAt DATETIME2 DEFAULT SYSDATETIME()
);

-- Using default
INSERT INTO Orders (OrderId) VALUES (1);
-- OrderDate, Status, IsActive, CreatedAt will use defaults
```

---

## Indexing Fundamentals

### What is an Index?

**Index** = Data structure that improves query performance at the cost of write performance and storage

**Analogy:** Like a book index - helps find information quickly without reading entire book

**Benefits:**
- ✅ Faster SELECT queries
- ✅ Faster sorting (ORDER BY)
- ✅ Faster filtering (WHERE)
- ✅ Faster joins

**Costs:**
- ❌ Slower INSERT, UPDATE, DELETE
- ❌ Additional storage space
- ❌ Index maintenance overhead

---

## Clustered Index

**Clustered Index** = Determines the physical order of data in the table

**Key Points:**
- 📌 Only ONE clustered index per table
- 📌 Table data is sorted by clustered index key
- 📌 Leaf nodes contain actual data rows
- 📌 Primary key creates clustered index by default

```sql
-- Automatically created with PRIMARY KEY
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY,  -- Creates clustered index
    Name VARCHAR(100)
);

-- Explicit clustered index
CREATE CLUSTERED INDEX IX_Customers_CustomerId 
ON Customers(CustomerId);

-- Primary key with non-clustered index
CREATE TABLE Products (
    ProductId INT PRIMARY KEY NONCLUSTERED,  -- Non-clustered PK
    ProductName VARCHAR(100)
);

CREATE CLUSTERED INDEX IX_Products_ProductName 
ON Products(ProductName);

-- View clustered index
SELECT 
    t.name AS TableName,
    i.name AS IndexName,
    i.type_desc AS IndexType
FROM sys.tables t
INNER JOIN sys.indexes i ON t.object_id = i.object_id
WHERE i.type = 1;  -- Clustered index
```

### Choosing Clustered Index Column

**Good candidates:**
- ✅ Primary key (most common)
- ✅ Frequently used in range queries (dates, IDs)
- ✅ Unique or near-unique values
- ✅ Narrow key (int better than varchar)
- ✅ Ever-increasing values (IDENTITY, timestamps)

**Avoid:**
- ❌ Wide keys (multiple large columns)
- ❌ Frequently updated columns
- ❌ Random values (GUIDs without NEWSEQUENTIALID)

```sql
-- ✅ Good clustered index (sequential, narrow)
CREATE TABLE Orders (
    OrderId INT IDENTITY(1,1) PRIMARY KEY CLUSTERED
);

-- ❌ Poor clustered index (random GUIDs cause fragmentation)
CREATE TABLE Orders (
    OrderId UNIQUEIDENTIFIER PRIMARY KEY CLUSTERED DEFAULT NEWID()
);

-- ✅ Better for GUIDs (sequential)
CREATE TABLE Orders (
    OrderId UNIQUEIDENTIFIER PRIMARY KEY CLUSTERED DEFAULT NEWSEQUENTIALID()
);
```

---

## Non-Clustered Index

**Non-Clustered Index** = Separate structure that points to data

**Key Points:**
- 📌 Up to 999 non-clustered indexes per table (SQL Server 2016+)
- 📌 Leaf nodes contain pointer to data (or clustered index key)
- 📌 Can have multiple per table
- 📌 Can be created on views

```sql
-- Basic non-clustered index
CREATE NONCLUSTERED INDEX IX_Customers_LastName 
ON Customers(LastName);

-- Multiple columns (composite index)
CREATE NONCLUSTERED INDEX IX_Customers_LastName_FirstName 
ON Customers(LastName, FirstName);

-- With included columns (covering index)
CREATE NONCLUSTERED INDEX IX_Customers_LastName_Include 
ON Customers(LastName)
INCLUDE (FirstName, Email, Phone);

-- Unique non-clustered index
CREATE UNIQUE NONCLUSTERED INDEX IX_Customers_Email 
ON Customers(Email);

-- Filtered index (subset of rows)
CREATE NONCLUSTERED INDEX IX_Orders_ActiveOrders 
ON Orders(OrderDate)
WHERE Status IN ('Pending', 'Processing');

-- Drop index
DROP INDEX IX_Customers_LastName ON Customers;
```

### Index Column Order Matters

```sql
-- Index on (LastName, FirstName)
CREATE INDEX IX_Customers_LastName_FirstName 
ON Customers(LastName, FirstName);

-- ✅ Can use index
SELECT * FROM Customers WHERE LastName = 'Smith';
SELECT * FROM Customers WHERE LastName = 'Smith' AND FirstName = 'John';

-- ❌ Cannot use index efficiently
SELECT * FROM Customers WHERE FirstName = 'John';

-- Rule: Index can be used if query filters on leftmost columns
```

---

## Covering Index (INCLUDE clause)

**Covering Index** = Index contains all columns needed by query (no table lookup required)

```sql
-- Query we want to optimize
SELECT CustomerId, FirstName, LastName, Email
FROM Customers
WHERE LastName = 'Smith';

-- ❌ Without covering index (requires table lookup)
CREATE INDEX IX_Customers_LastName ON Customers(LastName);

-- ✅ With covering index (no table lookup needed)
CREATE INDEX IX_Customers_LastName_Covering 
ON Customers(LastName)
INCLUDE (CustomerId, FirstName, Email);

-- When to use INCLUDE:
-- ✅ Columns in SELECT but not in WHERE/JOIN
-- ✅ Columns not useful for seeks/scans
-- ✅ Large columns (reduces index size vs. adding to key)
```

---

## Filtered Index

**Filtered Index** = Index on subset of rows (SQL Server 2008+)

```sql
-- Index only active orders
CREATE INDEX IX_Orders_ActiveOrders 
ON Orders(OrderDate, CustomerId)
WHERE Status IN ('Pending', 'Processing');

-- Index only non-NULL values
CREATE INDEX IX_Customers_Phone 
ON Customers(Phone)
WHERE Phone IS NOT NULL;

-- Index recent data
CREATE INDEX IX_Orders_Recent 
ON Orders(OrderDate, TotalAmount)
WHERE OrderDate >= '2024-01-01';

-- Benefits:
-- ✅ Smaller index size
-- ✅ Faster maintenance
-- ✅ More efficient queries on filtered data
```

---

## Columnstore Index

**Columnstore Index** = Stores data by column (not row) for analytics

```sql
-- Non-clustered columnstore
CREATE NONCLUSTERED COLUMNSTORE INDEX IX_Sales_Columnstore 
ON Sales(ProductId, SaleDate, Amount, Quantity);

-- Clustered columnstore (SQL Server 2014+)
CREATE CLUSTERED COLUMNSTORE INDEX IX_SalesArchive_Columnstore 
ON SalesArchive;

-- When to use:
-- ✅ Large fact tables (millions of rows)
-- ✅ Analytical queries (aggregations, scans)
-- ✅ Data warehouses
-- ❌ OLTP (frequent updates)
-- ❌ Small tables
```

---

## Index Best Practices

### 1. Index Selectivity

```sql
-- ✅ High selectivity (good for index)
-- Example: CustomerId - unique or near-unique
CREATE INDEX IX_Customers_CustomerId ON Customers(CustomerId);

-- ❌ Low selectivity (poor for index)
-- Example: Gender - only 2-3 values
-- Don't create: CREATE INDEX IX_Customers_Gender ON Customers(Gender);

-- Rule: Index columns with high selectivity (many distinct values)
-- Formula: Selectivity = (Distinct Values / Total Rows)
```

### 2. Index Maintenance

```sql
-- Check index fragmentation
SELECT 
    OBJECT_NAME(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(
    DB_ID(), NULL, NULL, NULL, 'LIMITED'
) ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id 
    AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
    AND ips.page_count > 1000
ORDER BY ips.avg_fragmentation_in_percent DESC;

-- Rebuild index (< 30% fragmentation)
ALTER INDEX IX_Customers_LastName ON Customers REORGANIZE;

-- Rebuild index (> 30% fragmentation)
ALTER INDEX IX_Customers_LastName ON Customers REBUILD;

-- Rebuild all indexes on table
ALTER INDEX ALL ON Customers REBUILD;

-- Update statistics
UPDATE STATISTICS Customers;
```

### 3. Index Naming Conventions

```sql
-- Good naming pattern: IX_{TableName}_{Columns}[_{Purpose}]

-- Standard index
CREATE INDEX IX_Customers_LastName ON Customers(LastName);

-- Composite index
CREATE INDEX IX_Orders_CustomerId_OrderDate ON Orders(CustomerId, OrderDate);

-- Covering index
CREATE INDEX IX_Customers_LastName_Covering 
ON Customers(LastName) INCLUDE (FirstName, Email);

-- Filtered index
CREATE INDEX IX_Orders_ActiveOrders 
ON Orders(OrderDate) WHERE Status = 'Active';

-- Unique index
CREATE UNIQUE INDEX UX_Customers_Email ON Customers(Email);
```

### 4. Over-Indexing vs Under-Indexing

```sql
-- ❌ Over-indexed (too many indexes)
CREATE TABLE Products (
    ProductId INT PRIMARY KEY,
    ProductName VARCHAR(100),
    Category VARCHAR(50),
    Price DECIMAL(10,2),
    Quantity INT
);

-- Don't do this:
CREATE INDEX IX_Products_ProductName ON Products(ProductName);
CREATE INDEX IX_Products_Category ON Products(Category);
CREATE INDEX IX_Products_Price ON Products(Price);
CREATE INDEX IX_Products_Quantity ON Products(Quantity);
CREATE INDEX IX_Products_ProductName_Category ON Products(ProductName, Category);
CREATE INDEX IX_Products_Category_Price ON Products(Category, Price);
-- Result: Slow INSERTs, UPDATEs, DELETEs

-- ✅ Properly indexed (based on actual queries)
CREATE INDEX IX_Products_Category ON Products(Category);
CREATE INDEX IX_Products_Price ON Products(Price);
-- Only create indexes you actually need!

-- Guideline: Start with 2-4 indexes per table, add based on query patterns
```

### 5. Finding Missing Indexes

```sql
-- Query to find missing indexes
SELECT 
    OBJECT_NAME(mid.object_id) AS TableName,
    mid.equality_columns AS EqualityColumns,
    mid.inequality_columns AS InequalityColumns,
    mid.included_columns AS IncludedColumns,
    migs.avg_user_impact AS AvgUserImpact,
    migs.user_seeks + migs.user_scans AS TotalSeeksAndScans,
    'CREATE INDEX IX_' + OBJECT_NAME(mid.object_id) + '_Missing' +
    ' ON ' + mid.statement + 
    ' (' + ISNULL(mid.equality_columns, '') + 
    CASE WHEN mid.inequality_columns IS NOT NULL 
         THEN ',' + mid.inequality_columns 
         ELSE '' 
    END + ')' +
    CASE WHEN mid.included_columns IS NOT NULL 
         THEN ' INCLUDE (' + mid.included_columns + ')' 
         ELSE '' 
    END AS CreateIndexStatement
FROM sys.dm_db_missing_index_details mid
INNER JOIN sys.dm_db_missing_index_groups mig ON mid.index_handle = mig.index_handle
INNER JOIN sys.dm_db_missing_index_group_stats migs ON mig.index_group_handle = migs.group_handle
WHERE migs.avg_user_impact > 50  -- High impact
ORDER BY migs.avg_user_impact DESC;
```

### 6. Finding Unused Indexes

```sql
-- Query to find unused indexes
SELECT 
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    i.type_desc AS IndexType,
    ius.user_seeks,
    ius.user_scans,
    ius.user_lookups,
    ius.user_updates,
    'DROP INDEX ' + i.name + ' ON ' + OBJECT_NAME(i.object_id) AS DropStatement
FROM sys.indexes i
LEFT JOIN sys.dm_db_index_usage_stats ius 
    ON i.object_id = ius.object_id AND i.index_id = ius.index_id
WHERE 
    i.type_desc = 'NONCLUSTERED'
    AND i.is_primary_key = 0
    AND i.is_unique_constraint = 0
    AND (ius.user_seeks + ius.user_scans + ius.user_lookups) < 100
    AND ius.user_updates > 1000  -- More writes than reads
ORDER BY ius.user_updates DESC;
```

---

## Query Optimization Fundamentals

### Execution Plans

**Execution Plan** = SQL Server's roadmap for executing a query

**Types:**
1. **Estimated Plan** - What SQL Server plans to do (CTRL+L)
2. **Actual Plan** - What SQL Server actually did (CTRL+M)

```sql
-- Enable actual execution plan
SET STATISTICS TIME ON;
SET STATISTICS IO ON;

-- Run query
SELECT * FROM Customers WHERE LastName = 'Smith';

-- View plan in SSMS (CTRL+M then execute query)
-- Or get XML plan:
SELECT * FROM Customers WHERE LastName = 'Smith'
OPTION (RECOMPILE, QUERYTRACEON 8666);
```

### Reading Execution Plans

**Read from right to left, top to bottom**

**Common operators:**
- **Table Scan** 🔴 - Reads entire table (slow for large tables)
- **Clustered Index Scan** 🟡 - Reads all rows in clustered index
- **Index Seek** 🟢 - Efficiently finds specific rows (fast)
- **Index Scan** 🟡 - Reads all rows in index
- **Key Lookup** 🟡 - Extra table lookup after index seek
- **Nested Loops** 🟢 - Good for small datasets
- **Hash Match** 🟡 - Good for large datasets
- **Merge Join** 🟢 - Best when inputs are sorted

**Cost percentages:**
- Higher percentage = more expensive operation
- Focus optimization on highest cost operators

```sql
-- Example query with execution plan analysis
SELECT 
    c.CustomerId,
    c.FirstName,
    c.LastName,
    COUNT(o.OrderId) AS OrderCount,
    SUM(o.TotalAmount) AS TotalSpent
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE c.Country = 'USA'
GROUP BY c.CustomerId, c.FirstName, c.LastName
ORDER BY TotalSpent DESC;

-- Look for:
-- 🔴 Table/Index Scans on large tables
-- 🟡 High-cost operators (> 20%)
-- 🔴 Key Lookups (consider covering index)
-- 🔴 Implicit conversions (yellow warning icons)
-- 🟡 Missing index recommendations
```

---

## Query Optimization Techniques

### 1. SELECT Only Required Columns

```sql
-- ❌ Bad (transfers unnecessary data)
SELECT * FROM Customers WHERE CustomerId = 1;

-- ✅ Good (specific columns only)
SELECT CustomerId, FirstName, LastName, Email 
FROM Customers 
WHERE CustomerId = 1;

-- Benefits:
-- - Less data transferred
-- - Can use covering indexes
-- - Better memory utilization
```

### 2. Use WHERE Instead of HAVING

```sql
-- ❌ Bad (filters after grouping)
SELECT Category, AVG(Price) AS AvgPrice
FROM Products
GROUP BY Category
HAVING Category = 'Electronics';

-- ✅ Good (filters before grouping)
SELECT Category, AVG(Price) AS AvgPrice
FROM Products
WHERE Category = 'Electronics'
GROUP BY Category;

-- Rule: Use WHERE for row filtering, HAVING for aggregate filtering
```

### 3. Avoid Functions on Indexed Columns

```sql
-- ❌ Bad (can't use index - non-sargable)
SELECT * FROM Orders 
WHERE YEAR(OrderDate) = 2024;

SELECT * FROM Customers 
WHERE UPPER(LastName) = 'SMITH';

-- ✅ Good (sargable - can use index)
SELECT * FROM Orders 
WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';

SELECT * FROM Customers 
WHERE LastName = 'Smith';

-- Sargable = Search ARGument ABLE (can use index)
```

### 4. Use EXISTS Instead of IN with Subqueries

```sql
-- ❌ Slower for large subqueries
SELECT * FROM Customers
WHERE CustomerId IN (
    SELECT CustomerId FROM Orders WHERE TotalAmount > 1000
);

-- ✅ Faster (stops at first match)
SELECT * FROM Customers c
WHERE EXISTS (
    SELECT 1 FROM Orders o 
    WHERE o.CustomerId = c.CustomerId 
    AND o.TotalAmount > 1000
);

-- EXISTS is more efficient when:
-- - Subquery returns many rows
-- - You only care about existence, not values
```

### 5. Use JOIN Instead of Subqueries (Usually)

```sql
-- ❌ Subquery (can be slower)
SELECT 
    c.CustomerId,
    c.FirstName,
    (SELECT COUNT(*) FROM Orders o WHERE o.CustomerId = c.CustomerId) AS OrderCount
FROM Customers c;

-- ✅ JOIN (usually faster)
SELECT 
    c.CustomerId,
    c.FirstName,
    COUNT(o.OrderId) AS OrderCount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
GROUP BY c.CustomerId, c.FirstName;

-- Exception: Subqueries can be better for scalar values or when optimized well
```

### 6. Avoid SELECT DISTINCT (Use GROUP BY if Possible)

```sql
-- ❌ DISTINCT (requires sort/hash to remove duplicates)
SELECT DISTINCT CustomerId FROM Orders;

-- ✅ GROUP BY (might be optimized better)
SELECT CustomerId FROM Orders GROUP BY CustomerId;

-- Better: Fix root cause of duplicates with proper joins
```

### 7. Use UNION ALL Instead of UNION

```sql
-- ❌ UNION (removes duplicates - expensive sort operation)
SELECT FirstName, LastName FROM Customers
UNION
SELECT FirstName, LastName FROM Employees;

-- ✅ UNION ALL (keeps duplicates - faster)
SELECT FirstName, LastName FROM Customers
UNION ALL
SELECT FirstName, LastName FROM Employees;

-- Use UNION ALL unless you specifically need to remove duplicates
```

### 8. Avoid Wildcard at Start of LIKE

```sql
-- ❌ Can't use index
SELECT * FROM Customers WHERE LastName LIKE '%son';

-- ✅ Can use index
SELECT * FROM Customers WHERE LastName LIKE 'John%';

-- ✅ Better: Full-text search for complex patterns
CREATE FULLTEXT INDEX ON Customers(LastName);
SELECT * FROM Customers WHERE CONTAINS(LastName, 'son');
```

### 9. Use Appropriate JOIN Types

```sql
-- ❌ Bad (unnecessary LEFT JOIN)
SELECT c.*, o.OrderId
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE o.OrderId IS NOT NULL;  -- Negates LEFT JOIN

-- ✅ Good (INNER JOIN)
SELECT c.*, o.OrderId
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId;
```

### 10. Optimize OR Conditions

```sql
-- ❌ OR conditions can't use indexes efficiently
SELECT * FROM Products 
WHERE Category = 'Electronics' OR Price > 1000;

-- ✅ Use UNION ALL if possible
SELECT * FROM Products WHERE Category = 'Electronics'
UNION ALL
SELECT * FROM Products WHERE Price > 1000 AND Category <> 'Electronics';

-- Or use IN for single column
SELECT * FROM Products 
WHERE Category IN ('Electronics', 'Books');
```

---

## Common Query Anti-Patterns

### 1. N+1 Query Problem

```sql
-- ❌ Anti-pattern (multiple round trips)
-- In application code:
foreach (var customer in customers)
{
    var orders = ExecuteQuery("SELECT * FROM Orders WHERE CustomerId = " + customer.Id);
    // Process orders
}

-- ✅ Solution (single query)
SELECT 
    c.CustomerId,
    c.FirstName,
    o.OrderId,
    o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE c.Country = 'USA';
```

### 2. Using Scalar Functions in SELECT

```sql
-- ❌ Bad (function called for every row)
CREATE FUNCTION dbo.GetCustomerOrderCount(@CustomerId INT)
RETURNS INT
AS
BEGIN
    RETURN (SELECT COUNT(*) FROM Orders WHERE CustomerId = @CustomerId);
END;

SELECT 
    CustomerId,
    FirstName,
    dbo.GetCustomerOrderCount(CustomerId) AS OrderCount  -- Called per row!
FROM Customers;

-- ✅ Good (single join)
SELECT 
    c.CustomerId,
    c.FirstName,
    COUNT(o.OrderId) AS OrderCount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
GROUP BY c.CustomerId, c.FirstName;
```

### 3. Using Cursors Instead of Set-Based Operations

```sql
-- ❌ Anti-pattern (cursor - row-by-row processing)
DECLARE @CustomerId INT, @Email VARCHAR(100);
DECLARE customer_cursor CURSOR FOR 
    SELECT CustomerId, Email FROM Customers;

OPEN customer_cursor;
FETCH NEXT FROM customer_cursor INTO @CustomerId, @Email;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Process each customer
    UPDATE Customers SET LastProcessed = GETDATE() 
    WHERE CustomerId = @CustomerId;
    
    FETCH NEXT FROM customer_cursor INTO @CustomerId, @Email;
END;

CLOSE customer_cursor;
DEALLOCATE customer_cursor;

-- ✅ Solution (set-based)
UPDATE Customers 
SET LastProcessed = GETDATE();
-- Single statement, much faster!
```

### 4. Implicit Data Type Conversion

```sql
-- ❌ Bad (implicit conversion prevents index use)
CREATE TABLE Products (
    ProductCode VARCHAR(20)
);
CREATE INDEX IX_Products_ProductCode ON Products(ProductCode);

-- Query with implicit conversion
SELECT * FROM Products WHERE ProductCode = 123;  -- INT to VARCHAR conversion

-- ✅ Good (explicit, correct type)
SELECT * FROM Products WHERE ProductCode = '123';
```

### 5. Using NOT IN with NULLs

```sql
-- ❌ Dangerous (returns no results if subquery has NULL)
SELECT * FROM Customers
WHERE CustomerId NOT IN (
    SELECT CustomerId FROM Orders WHERE ShipDate IS NULL
);
-- If any CustomerId is NULL in subquery, entire query returns empty

-- ✅ Safe alternatives
SELECT * FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o 
    WHERE o.CustomerId = c.CustomerId AND o.ShipDate IS NULL
);

-- Or use LEFT JOIN
SELECT c.*
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId AND o.ShipDate IS NULL
WHERE o.OrderId IS NULL;
```

---

## Statistics and Query Optimizer

### Database Statistics

**Statistics** = Metadata about data distribution used by query optimizer

```sql
-- View statistics for a table
DBCC SHOW_STATISTICS('Customers', 'IX_Customers_LastName');

-- Update statistics manually
UPDATE STATISTICS Customers;

-- Update statistics for specific index
UPDATE STATISTICS Customers IX_Customers_LastName;

-- Auto-create statistics (enabled by default)
ALTER DATABASE YourDatabase SET AUTO_CREATE_STATISTICS ON;
ALTER DATABASE YourDatabase SET AUTO_UPDATE_STATISTICS ON;

-- Check when statistics were last updated
SELECT 
    OBJECT_NAME(s.object_id) AS TableName,
    s.name AS StatisticsName,
    sp.last_updated,
    sp.rows,
    sp.rows_sampled,
    sp.modification_counter
FROM sys.stats s
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
WHERE OBJECT_NAME(s.object_id) = 'Customers';
```

### Query Hints

```sql
-- Force specific index
SELECT * FROM Customers WITH (INDEX(IX_Customers_LastName))
WHERE LastName = 'Smith';

-- Force index seek
SELECT * FROM Customers WITH (FORCESEEK)
WHERE LastName = 'Smith';

-- Use NOLOCK (dirty reads)
SELECT * FROM Products WITH (NOLOCK);

-- Force recompile (don't cache plan)
SELECT * FROM Customers 
WHERE LastName = @LastName
OPTION (RECOMPILE);

-- Force specific join type
SELECT * FROM Customers c
INNER LOOP JOIN Orders o ON c.CustomerId = o.CustomerId;  -- Nested loops
INNER MERGE JOIN Orders o ON c.CustomerId = o.CustomerId; -- Merge join
INNER HASH JOIN Orders o ON c.CustomerId = o.CustomerId;  -- Hash join

-- Maxdop (max degree of parallelism)
SELECT * FROM LargeTable
OPTION (MAXDOP 4);  -- Use 4 cores
```

---

## Performance Monitoring

### Key Performance Metrics

```sql
-- Find expensive queries
SELECT TOP 10
    qs.total_elapsed_time / qs.execution_count AS avg_elapsed_time_ms,
    qs.execution_count,
    qs.total_logical_reads / qs.execution_count AS avg_logical_reads,
    SUBSTRING(qt.text, (qs.statement_start_offset/2) + 1,
        ((CASE qs.statement_end_offset
            WHEN -1 THEN DATALENGTH(qt.text)
            ELSE qs.statement_end_offset
        END - qs.statement_start_offset)/2) + 1) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY avg_elapsed_time_ms DESC;

-- Find queries with high I/O
SELECT TOP 10
    qs.total_logical_reads,
    qs.execution_count,
    qs.total_logical_reads / qs.execution_count AS avg_logical_reads,
    qt.text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY total_logical_reads DESC;

-- Wait statistics (what queries are waiting for)
SELECT 
    wait_type,
    wait_time_ms / 1000.0 / 60 AS wait_time_minutes,
    waiting_tasks_count
FROM sys.dm_os_wait_stats
WHERE wait_type NOT LIKE '%SLEEP%'
ORDER BY wait_time_ms DESC;

-- Database size
SELECT 
    name AS DatabaseName,
    size * 8 / 1024 AS SizeMB
FROM sys.master_files
WHERE database_id = DB_ID();
```

---

## Best Practices Summary

### Database Design
1. ✅ Normalize to 3NF, denormalize only when necessary
2. ✅ Use appropriate data types (don't over-allocate)
3. ✅ Add proper constraints (PK, FK, CHECK, UNIQUE)
4. ✅ Use IDENTITY for auto-incrementing keys
5. ✅ Consider soft deletes instead of hard deletes

### Indexing
1. ✅ Create indexes on foreign keys
2. ✅ Create indexes on frequently filtered columns (WHERE, JOIN)
3. ✅ Use covering indexes for frequently accessed columns
4. ✅ Don't over-index (2-5 indexes per table is usually enough)
5. ✅ Maintain indexes regularly (rebuild/reorganize)

### Query Writing
1. ✅ Select only required columns (avoid SELECT *)
2. ✅ Use WHERE before HAVING
3. ✅ Avoid functions on indexed columns
4. ✅ Use EXISTS over IN for subqueries
5. ✅ Use UNION ALL over UNION when duplicates are OK
6. ✅ Use appropriate JOIN types
7. ✅ Avoid cursors (use set-based operations)
8. ✅ Test queries with production-like data volumes

### Performance
1. ✅ Analyze execution plans
2. ✅ Monitor query performance regularly
3. ✅ Keep statistics updated
4. ✅ Use connection pooling
5. ✅ Consider caching for frequently accessed data

---

## Quick Decision Trees

### Should I Create an Index?

```
Is the column used in:
├─ WHERE clause frequently? → ✅ Yes
├─ JOIN conditions? → ✅ Yes
├─ ORDER BY often? → ✅ Yes
├─ GROUP BY often? → ✅ Yes
└─ Has low selectivity (few distinct values)? → ❌ No

Is the table:
├─ Mostly reads (SELECT)? → ✅ Yes
├─ Frequently updated (INSERT/UPDATE/DELETE)? → ⚠️ Careful
└─ Very small (< 1000 rows)? → ❌ No (table scan is fine)

Do you already have:
├─ 5+ indexes on this table? → ⚠️ Evaluate necessity
└─ Similar index? → ❌ No (avoid duplicate indexes)
```

### Clustered vs Non-Clustered?

```
Clustered Index:
├─ Primary key? → ✅ Yes (default choice)
├─ Ever-increasing value (IDENTITY)? → ✅ Yes
├─ Frequently used in range queries? → ✅ Yes
├─ Random values (GUID)? → ❌ No
└─ Frequently updated? → ❌ No

Non-Clustered Index:
├─ Need multiple indexes on table? → ✅ Yes
├─ Used for specific lookups? → ✅ Yes
├─ Need covering index? → ✅ Yes
└─ Already have clustered index? → ✅ Yes
```

### Query Optimization Checklist

```
Step 1: Analyze Execution Plan
├─ Table scans on large tables? → Add index
├─ Index scans that should be seeks? → Add WHERE or improve index
├─ Key lookups? → Add covering index
├─ Missing index recommendations? → Consider adding
└─ Implicit conversions (yellow warning)? → Fix data types

Step 2: Check Query Structure
├─ SELECT *? → Select specific columns
├─ Functions on indexed columns? → Rewrite to be sargable
├─ Subqueries? → Consider JOINs
├─ OR conditions? → Consider UNION ALL
└─ Cursors? → Rewrite as set-based

Step 3: Verify Statistics
├─ Statistics outdated? → Update statistics
├─ Wrong cardinality estimates? → Recompile or update statistics
└─ Parameter sniffing issues? → Use OPTION (RECOMPILE)

Step 4: Test Performance
├─ Run with actual data volumes
├─ Measure execution time and I/O
├─ Compare before/after metrics
└─ Monitor in production
```

---

**Guide Complete!** This comprehensive database design and optimization guide covers normalization, ER diagrams, constraints, indexing strategies, query optimization, execution plans, and performance tuning. Master these concepts to design efficient databases and write high-performance queries! 🚀
