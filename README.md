# ASP.NET Core Microservice - Self-Contained Deployment

Dieses Projekt demonstriert, wie eine ASP.NET Core 10.0 Anwendung als frameworkunabhängiger Windows Service bereitgestellt wird. Es basiert auf dem Standard-MVC WeatherForecast Template.

## 🚀 Schnellstart

### Schritt 1: Projekt builden
```bash
# Windows
dotnet publish -c Release -r win-x64 --output "C:\Deploy\MyApp"

# Linux (falls gewünscht)
dotnet publish -c Release -r linux-x64 --output "./deploy-linux"

# Multi-Platform
dotnet publish -c Release -r win-x64 --output "./deploy/windows"
dotnet publish -c Release -r linux-x64 --output "./deploy/linux"
```

### Schritt 2: Windows Service erstellen
```powershell
# PowerShell als Administrator öffnen (Windows + X → "PowerShell (Administrator)")
New-Service -Name "MyAPI" -BinaryPathName "C:\Deploy\MyApp\AuslieferungTestApp.exe --urls http://localhost:8080" -DisplayName "My API Service" -StartupType Automatic
```

### Schritt 3: Service starten
```powershell
Start-Service "MyAPI"
```

### Schritt 4: Anwendung testen
- Browser öffnen: http://localhost:8080
- API-Endpoint: http://localhost:8080/weatherforecast

## 📋 Service Management

```powershell
# Service-Status prüfen
Get-Service "MyAPI"

# Service starten
Start-Service "MyAPI"

# Service stoppen
Stop-Service "MyAPI"

# Service entfernen (für Updates)
Remove-Service "MyAPI"
```

## 🔧 Port konfigurieren

### Methode 1: Beim Service erstellen
```powershell
New-Service -Name "MyAPI" -BinaryPathName "C:\path\to\app.exe --urls http://localhost:9000" -DisplayName "My API" -StartupType Automatic
```

### Methode 2: appsettings.json
```json
{
  "Urls": "http://localhost:9000",
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

## 🛠️ Troubleshooting

### Service startet nicht?
```powershell
# Manuell testen
C:\Deploy\MyApp\AuslieferungTestApp.exe

# Event-Logs checken
Get-EventLog -LogName System -Source "Service Control Manager" -Newest 5
```

### Port bereits belegt?
```powershell
# Belegte Ports anzeigen
netstat -an | findstr LISTENING

# Anderen Port verwenden
Remove-Service "MyAPI"
New-Service -Name "MyAPI" -BinaryPathName "C:\path\to\app.exe --urls http://localhost:8081" -DisplayName "My API" -StartupType Automatic
```

## 📁 Projektstruktur

```
MeinProjekt/
├── Controllers/          # API Controller
├── Models/              # Datenmodelle  
├── Program.cs           # Haupteinstiegspunkt
├── appsettings.json     # Konfiguration
├── MeinProjekt.csproj   # Projektdatei
└── README.md           # Diese Datei
```

## 💻 Program.cs Konfiguration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Plattformspezifische Service-Unterstützung
if (OperatingSystem.IsWindows())
{
    builder.Host.UseWindowsService();
}
else if (OperatingSystem.IsLinux())
{
    builder.Host.UseSystemd();
}

builder.Services.AddControllers();
builder.Services.AddOpenApi();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}

app.UseHttpsRedirection();
app.UseAuthorization();

app.MapGet("/", () => "Welcome to My API! Try /api/...");
app.MapControllers();

app.Run();
```

## ⚙️ Projektkonfiguration (.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
    <PropertyGroup>
        <TargetFramework>net10.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
        <OutputType>exe</OutputType>
        <PublishSingleFile>true</PublishSingleFile>
        <SelfContained>true</SelfContained>
        <RuntimeIdentifier>win-x64</RuntimeIdentifier>
        <PlatformTarget>x64</PlatformTarget>
        <IncludeNativeLibrariesForSelfExtract>true</IncludeNativeLibrariesForSelfExtract>
        <IncludeAllContentForSelfExtract>true</IncludeAllContentForSelfExtract>
        <DebugType Condition="'$(Configuration)' == 'Release'">None</DebugType>
        <DebugSymbols Condition="'$(Configuration)' == 'Release'">false</DebugSymbols>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="10.0.0-preview.4.25258.110" />
        <PackageReference Include="Microsoft.Extensions.Hosting.WindowsServices" Version="9.0.8" />
        <PackageReference Include="Microsoft.Extensions.Hosting.Systemd" Version="9.0.8" Condition="$([MSBuild]::IsOSPlatform('Linux'))" />
    </ItemGroup>
</Project>
```

## 🔄 Update-Workflow

```powershell
# 1. Service stoppen
Stop-Service "MyAPI"

# 2. Neue Version builden
dotnet publish -c Release -r win-x64 --output "C:\Deploy\MyApp"

# 3. Service starten
Start-Service "MyAPI"
```

## 🌟 Vorteile dieser Lösung

✅ **Frameworkunabhängig** - Keine .NET Installation erforderlich  
✅ **Single-File** - Eine ausführbare Datei (~100MB)  
✅ **Cross-Platform** - Windows Service & Linux systemd Support  
✅ **Windows Service** - Automatischer Start beim Systemstart  
✅ **Einfaches Deployment** - Kopieren und Service erstellen  
✅ **Standard-Ports** - HTTP: 5000, HTTPS: 5001 (konfigurierbar)  
✅ **Development Tools** - Swagger UI für API-Dokumentation  

## 📚 API-Endpoints (Standard WeatherForecast)

| Endpoint | Methode | Beschreibung |
|----------|---------|-------------|
| `/` | GET | Willkommensnachricht |
| `/weatherforecast` | GET | Beispiel-Wetterdaten |
| `/swagger` | GET | API-Dokumentation (nur Development) |

## 🔐 Wichtige Hinweise

- **Administrator-Rechte** erforderlich für Service-Management
- **Firewall-Regeln** eventuell anpassen für externe Zugriffe
- **HTTPS-Zertifikate** für Production-Umgebungen konfigurieren
- **Logs** über Windows Event Log oder Application Insights

## 💡 Für Produktionsumgebungen

```json
// appsettings.Production.json
{
  "Urls": "https://+:443;http://+:80",
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  }
}
```

---

**Entwickelt für .NET 10.0 Preview** | Funktioniert ab Windows 10/Server 2016