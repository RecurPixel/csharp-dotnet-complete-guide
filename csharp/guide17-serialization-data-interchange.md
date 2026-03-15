# C# Serialization and Data Interchange Quick Reference

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                   FORMAT SELECTION GUIDE                                 │
│                                                                          │
│  Format   │ Use When                         │ Avoid When               │
│  ─────────┼──────────────────────────────────┼────────────────────────  │
│  JSON     │ APIs, configs, modern storage     │ Binary-critical perf     │
│  CSV      │ Tabular exports/imports           │ Nested / complex data    │
│  XML      │ Legacy systems, SOAP, config      │ New greenfield APIs      │
│  Binary   │ Internal cache, high perf         │ Cross-language exchange  │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 1. System.Text.Json — Core Usage

### Serialize and Deserialize

```csharp
using System.Text.Json;

var options = new JsonSerializerOptions
{
    PropertyNamingPolicy        = JsonNamingPolicy.CamelCase,
    WriteIndented               = false,
    DefaultIgnoreCondition      = JsonIgnoreCondition.WhenWritingNull,
    PropertyNameCaseInsensitive = true,     // tolerant deserialization
};

// Serialize
string json = JsonSerializer.Serialize(obj, options);

// Serialize to stream (preferred for large payloads)
await JsonSerializer.SerializeAsync(stream, obj, options);

// Deserialize
MyDto? dto = JsonSerializer.Deserialize<MyDto>(json, options);

// Deserialize from stream
MyDto? fromStream = await JsonSerializer.DeserializeAsync<MyDto>(stream, options);
```

### Useful JsonSerializerOptions Presets

```csharp
// Read-friendly for APIs
var apiOptions = new JsonSerializerOptions(JsonSerializerDefaults.Web)
{
    WriteIndented = false,
};

// Strict (default): property names must match exactly
var strict = new JsonSerializerOptions(JsonSerializerDefaults.General);

// ✅ Reuse options instances — they cache reflection metadata internally
// ❌ Don't create a new JsonSerializerOptions per call
```

### Attributes

```csharp
public class OrderDto
{
    [JsonPropertyName("order_id")]
    public int Id { get; init; }

    [JsonIgnore]
    public string InternalNote { get; set; } = "";

    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
    public string? OptionalField { get; set; }

    [JsonRequired]                         // .NET 7+ — throws if missing in JSON
    public string CustomerId { get; init; } = "";

    [JsonConverter(typeof(JsonStringEnumConverter))]
    public OrderStatus Status { get; set; }
}
```

---

## 2. Custom Converters

```csharp
// Custom converter for a type System.Text.Json doesn't handle natively
public class DateOnlyConverter : JsonConverter<DateOnly>
{
    private const string Format = "yyyy-MM-dd";

    public override DateOnly Read(
        ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        string? value = reader.GetString();
        return DateOnly.ParseExact(value!, Format, CultureInfo.InvariantCulture);
    }

    public override void Write(
        Utf8JsonWriter writer, DateOnly value, JsonSerializerOptions options)
        => writer.WriteStringValue(value.ToString(Format, CultureInfo.InvariantCulture));
}

// Register on options
options.Converters.Add(new DateOnlyConverter());
options.Converters.Add(new JsonStringEnumConverter(JsonNamingPolicy.CamelCase));
```

---

## 3. Naming Policies

```csharp
// camelCase  → JsonNamingPolicy.CamelCase      (default in Web APIs)
// snake_case → JsonNamingPolicy.SnakeCaseLower  (.NET 8+)
// UPPER_CASE → custom policy

public class UpperSnakeCasePolicy : JsonNamingPolicy
{
    public override string ConvertName(string name)
        => string.Concat(name.Select((c, i) =>
            i > 0 && char.IsUpper(c) ? $"_{c}" : $"{c}")).ToUpperInvariant();
}
```

---

## 4. Polymorphism and Type Discrimination (.NET 7+)

```csharp
[JsonPolymorphic(TypeDiscriminatorPropertyName = "$type")]
[JsonDerivedType(typeof(CatDto), "cat")]
[JsonDerivedType(typeof(DogDto), "dog")]
public abstract class AnimalDto
{
    public string Name { get; set; } = "";
}

public class CatDto  : AnimalDto { public bool Indoor { get; set; } }
public class DogDto  : AnimalDto { public string Breed { get; set; } = ""; }

// Serializes with "$type": "cat"
// Deserializes to correct subtype automatically
AnimalDto animal = JsonSerializer.Deserialize<AnimalDto>(json, options)!;
```

---

## 5. Versioning / Contract Evolution Rules

```csharp
// DTO rules for safe versioning:

// ✅ Adding new optional properties is backward compatible
// ✅ Use nullable or default-valued new fields
// ❌ Never remove or rename existing properties (breaking change)
// ❌ Never change a property type

public class UserDtoV2
{
    public int    Id        { get; init; }
    public string Name      { get; init; } = "";
    public string? Timezone { get; init; }  // New in v2 — nullable, safe addition
}

// [JsonExtensionData] collects unknown properties (forward-compatibility)
public class FlexibleDto
{
    public string Name { get; set; } = "";

    [JsonExtensionData]
    public Dictionary<string, JsonElement>? Extra { get; set; }
}
```

---

## 6. Streaming JSON with Utf8JsonReader / Utf8JsonWriter

Use these for large payloads where you don't want to allocate full object graphs.

```csharp
// Utf8JsonWriter — write JSON directly to a stream
using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream, new JsonWriterOptions { Indented = false });

writer.WriteStartObject();
writer.WriteString("name", "Alice");
writer.WriteNumber("age", 30);
writer.WriteBoolean("active", true);
writer.WriteEndObject();
writer.Flush();

// Utf8JsonReader — pull-parse without allocating model objects
byte[] jsonBytes = File.ReadAllBytes("large.json");
var reader = new Utf8JsonReader(jsonBytes);

while (reader.Read())
{
    if (reader.TokenType == JsonTokenType.PropertyName)
    {
        string? propName = reader.GetString();
        reader.Read();
        if (propName == "id")
            Console.WriteLine(reader.GetInt32());
    }
}
```

---

## 7. Source Generation (Zero Reflection, AOT-ready)

```csharp
// Define a context — avoids reflection at runtime
[JsonSerializable(typeof(OrderDto))]
[JsonSerializable(typeof(List<OrderDto>))]
[JsonSourceGenerationOptions(
    PropertyNamingPolicy = JsonKnownNamingPolicy.CamelCase,
    WriteIndented = false)]
internal partial class AppJsonContext : JsonSerializerContext { }

// Use the context
string json = JsonSerializer.Serialize(order, AppJsonContext.Default.OrderDto);
OrderDto? dto = JsonSerializer.Deserialize(json, AppJsonContext.Default.OrderDto);
```

> ✅ Required for NativeAOT compatibility. Also faster cold-start and lower allocation.

---

## 8. CSV Parsing — Robust Patterns

> ❌ Don't manually split on commas — quoted fields with commas will break.

```csharp
// Install: CsvHelper
using CsvHelper;
using CsvHelper.Configuration;
using System.Globalization;

// Read
var config = new CsvConfiguration(CultureInfo.InvariantCulture)
{
    HasHeaderRecord = true,
    MissingFieldFound = null,     // tolerate missing columns
    BadDataFound = null,          // skip bad rows (log in prod)
};

using var reader = new StreamReader("data.csv");
using var csv    = new CsvReader(reader, config);

IEnumerable<OrderRow> orders = csv.GetRecords<OrderRow>();
// GetRecords is lazy — iterate once, process in a pipeline

// Write
using var writer    = new StreamWriter("output.csv");
using var csvWriter = new CsvWriter(writer, CultureInfo.InvariantCulture);
csvWriter.WriteRecords(orders);
```

```csharp
// Class map for custom column names
public class OrderMap : ClassMap<OrderRow>
{
    public OrderMap()
    {
        Map(m => m.Id).Name("order_id");
        Map(m => m.Amount).Name("total_amount").TypeConverter<DecimalConverter>();
    }
}
csv.Context.RegisterClassMap<OrderMap>();
```

---

## 9. XML — Minimal Practical Patterns

```csharp
using System.Xml;
using System.Xml.Serialization;

// XmlSerializer — class-based
[XmlRoot("Order")]
public class OrderXml
{
    [XmlAttribute("id")]
    public int Id { get; set; }

    [XmlElement("Customer")]
    public string Customer { get; set; } = "";
}

var serializer = new XmlSerializer(typeof(OrderXml));

// Serialize
using var sw = new StringWriter();
serializer.Serialize(sw, order);
string xml = sw.ToString();

// Deserialize
using var sr = new StringReader(xml);
var restored = (OrderXml?)serializer.Deserialize(sr);

// XDocument — LINQ-based for reading/querying
XDocument doc  = XDocument.Load("data.xml");
var ids        = doc.Descendants("Order")
                    .Select(e => (int)e.Attribute("id")!)
                    .ToList();
```

---

## 10. DTO Design Rules for Safe Contract Evolution

```
✅ Use init-only properties for immutable DTOs
✅ Use nullable types for optional fields
✅ Avoid exposing internal types — map to dedicated DTOs
✅ Version your DTOs explicitly if breaking changes are needed (UserDtoV2)
✅ Apply [JsonExtensionData] when consuming DTOs from external APIs
✅ Keep DTOs plain — no business logic, no EF navigation properties
✅ Use records for DTOs — value equality and conciseness

❌ Don't share the same DTO between input and output if they differ
❌ Don't expose database IDs directly in public-facing DTOs (use typed wrappers)
❌ Don't add [Required] to existing fields in an API already in production
```

```csharp
// Ideal DTO structure
public record CreateOrderRequest(
    string CustomerId,
    IReadOnlyList<OrderLineItem> Items,
    string? PromoCode = null
);

public record OrderResponse(
    int    OrderId,
    string Status,
    decimal Total
);
```

---

## Anti-Patterns

```
❌ Creating new JsonSerializerOptions per request (kills reflection cache)
❌ Manually splitting CSV on commas — use CsvHelper
❌ Using Newtonsoft.Json without a reason in new .NET 7+ projects
❌ Deserializing untrusted JSON into object or dynamic (injection risk)
❌ Trusting JsonElement values without null checking
❌ XmlSerializer on types with circular references — hang risk
❌ Storing deserialized objects in shared state without cloning
```

---

## Quick Reference Summary

| Task | API / Package |
|------|--------------|
| JSON serialize | `JsonSerializer.Serialize(obj, options)` |
| JSON deserialize | `JsonSerializer.Deserialize<T>(json, options)` |
| JSON stream write | `Utf8JsonWriter` |
| JSON stream read | `Utf8JsonReader` |
| JSON AOT / source gen | `JsonSerializerContext` |
| CSV read | `CsvHelper.CsvReader` |
| CSV write | `CsvHelper.CsvWriter` |
| XML serialize | `XmlSerializer` |
| XML query | `XDocument.Descendants(...)` |
| DTO versioning | `[JsonExtensionData]` + nullable new fields |

---

**Guide Complete!** Master System.Text.Json for JSON, CsvHelper for CSV, and keep DTOs thin and versioned. Source generators are the future for performance-critical paths. 📘
