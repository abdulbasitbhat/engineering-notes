# Creating jars vs dll

## The Pattern

In both Java and .NET the approach is the same — parser logic lives in a separate library project, and a separate runnable app imports that library and calls it. The library never runs on its own.

---

## Java — Maven

### Project Structure

```
connector-middleware/
├── pom.xml                                       ← parent pom, groups all modules
├── parsers/                                      ← library module, produces a JAR
│   ├── pom.xml
│   └── src/main/java/com/connector/parsers/
│       └── json/
│           └── JsonParser.java
└── connector-app/                                ← runnable module, imports parsers
    ├── pom.xml
    └── src/main/java/com/connector/app/
        └── Main.java
```

### 1. Create the parser class

```java
// parsers/src/main/java/com/connector/parsers/json/JsonParser.java
package com.connector.parsers.json;

public class JsonParser {
    public String parse(String input) {
        return input.trim();
    }
}
```

No `main` here. This is a library class — it only has logic.

### 2. Declare the dependency in the consumer module

This is done manually in `connector-app/pom.xml`:

```xml
<dependency>
    <groupId>com.connector</groupId>
    <artifactId>parsers</artifactId>
    <version>1.0.0</version>
</dependency>
```

Maven resolves this automatically — no file path needed.

### 3. Call it in Main

```java
// connector-app/src/main/java/com/connector/app/Main.java
package com.connector.app;

import com.connector.parsers.json.JsonParser;

public class Main {
    public static void main(String[] args) {
        JsonParser parser = new JsonParser();
        String result = parser.parse("{\"key\": \"value\"}");
        System.out.println(result);
    }
}
```

### 4. Run it

```bash
# From the parent project root — builds all modules
mvn clean package

# Run the app module
mvn exec:java -pl connector-app -Dexec.mainClass="com.connector.app.Main"
```

### Adding a new parser

Create a new class in the `parsers` module, no other config needed:

```java
// parsers/src/main/java/com/connector/parsers/xml/XmlParser.java
package com.connector.parsers.xml;

public class XmlParser {
    public String parse(String input) {
        return input.trim();
    }
}
```

Then import and call it in `Main.java`:

```java
import com.connector.parsers.xml.XmlParser;

XmlParser xmlParser = new XmlParser();
System.out.println(xmlParser.parse("<root><key>value</key></root>"));
```

---

## .NET — Class Library

### Project Structure

```
Dynamic-middleware-connector/
├── ConnectorMiddleware.sln                       ← solution file, groups all projects
└── src/
    ├── Parsers/                                  ← class library, produces a DLL, not runnable
    │   ├── Parsers.csproj
    │   └── Json/
    │       └── JsonParser.cs
    └── ConnectorApp/                             ← console app, entry point, imports Parsers
        ├── ConnectorApp.csproj
        └── Program.cs
```

### 1. Create the parser class

```csharp
// src/Parsers/Json/JsonParser.cs
namespace Parsers.Json;

public class JsonParser
{
    public string Parse(string input)
    {
        return input.Trim();
    }
}
```

No `Main` here. Parser classes only contain logic.

### 2. Declare the dependency in ConnectorApp

Run this command from inside `src/ConnectorApp`:

```powershell
dotnet add reference ../Parsers/Parsers.csproj
```

This automatically edits `ConnectorApp.csproj` and adds:

```xml
<ItemGroup>
  <ProjectReference Include="../Parsers/Parsers.csproj" />
</ItemGroup>
```

You never touch the `.csproj` manually for this — the command does it.

### 3. Call it in Program.cs

```csharp
// src/ConnectorApp/Program.cs
using Parsers.Json;

var parser = new JsonParser();
var result = parser.Parse("{\"key\": \"value\"}");
Console.WriteLine(result);
```

### 4. Run it

```powershell
# From src/ConnectorApp
dotnet run
```

### Adding a new parser

Create a new class in the `Parsers` project, no other config needed:

```csharp
// src/Parsers/Xml/XmlParser.cs
namespace Parsers.Xml;

public class XmlParser
{
    public string Parse(string input)
    {
        return input.Trim();
    }
}
```

Then import and call it in `Program.cs`:

```csharp
using Parsers.Json;
using Parsers.Xml;

var jsonParser = new JsonParser();
Console.WriteLine(jsonParser.Parse("{\"key\": \"value\"}"));

var xmlParser = new XmlParser();
Console.WriteLine(xmlParser.Parse("<root><key>value</key></root>"));
```

No extra wiring needed. Because `ConnectorApp` already references the `Parsers` project, any new class added to it is immediately available.

---

## Side by Side

| | Java | .NET |
|---|---|---|
| Library project | Maven module, `<packaging>jar</packaging>` | `dotnet new classlib` |
| Runnable project | Maven module with `Main` class | `dotnet new console` |
| Wire library to app | Add `<dependency>` in `pom.xml` manually | `dotnet add reference` — edits `.csproj` automatically |
| Entry point | `public static void main(String[] args)` in a class | top-level code in `Program.cs` |
| Run the app | `mvn exec:java` | `dotnet run` |
| Add new parser | New class in library module, import in `Main` | New class in Parsers project, `using` in `Program.cs` |
| Does the library have a `main`/`Main`? | No | No |
