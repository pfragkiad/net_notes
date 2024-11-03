`Da Notes`

# dotnet best commands

Original source with full info for this:

https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-publish


```powershell
dotnet publish -p:PublishSingleFile=true --self-contained false -c Release
```

# Ignore null entries for JSON output

```cs
var builder = WebApplication.CreateBuilder(args);
var services = builder.Services;
builder.Services.Configure<Microsoft.AspNetCore.Http.Json.JsonOptions>(options =>
   options.SerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull);
```

# Winforms and Core DI

The dependency injection for Winforms can be set as shown below:

```cs

// To customize application configuration such as set high DPI settings or default font,
// see https://aka.ms/applicationconfiguration.
ApplicationConfiguration.Initialize();

var builder = Host
   .CreateDefaultBuilder()
   .ConfigureServices((context, services) =>
   {
       services
       ...
       .AddSingleton<frmMain>()
       .AddSingleton<ILogForm,frmLog>()
       //the log form should not be the same with the form used in the Application.Run() below
       .AddSingleton(provider =>  provider.GetService<ILogForm>().LogTextBox) 

       .AddSingleton<ItemForTextLoggerTest>()
       .AddSingleton<ItemForTextLoggerTestFactory>()
       ;

   })
    .ConfigureLogging((HostBuilderContext context, ILoggingBuilder builder) =>
	 // builder.ClearProviders();
	 builder.AddTextBoxConsoleLogger(context))
 ;

host = builder.Build();

var form = host.Services.GetService<frmMain>();
Application.Run(form);
```
# Minimal API

## Custom endpoint that changes response based on input

```cs
app.MapPost("/api/create/{userId}", async (HttpContext context) =>
{
    // Read the path parameter
    var userId = context.Request.RouteValues["userId"] as string;

    // Read the query parameter
    var apiKey = context.Request.Query["apiKey"];

    // Read the header parameter
    var contentType = context.Request.Headers["Content-Type"].FirstOrDefault() ?? "application/json";

    // Default response data
    var responseData = $"Received data for User '{userId}' with API Key '{apiKey}' and Content Type '{contentType}': ";

    if (contentType.Contains("application/json"))
    {
        // If the content type is JSON, parse the JSON data
        using (var reader = new StreamReader(context.Request.Body, Encoding.UTF8))
        {
            var jsonBody = await reader.ReadToEndAsync();
            responseData += $"JSON: {jsonBody}";
        }
    }
    else if (contentType.Contains("text/plain"))
    {
        // If the content type is plain text, read it directly
        var plainTextBody = await context.Request.ReadAsStringAsync();
        responseData += $"Plain Text: {plainTextBody}";
    }
    else
    {
        // Unsupported content type
        context.Response.StatusCode = 415; // Unsupported Media Type
        responseData = "Unsupported Content Type";
    }

    // Set the response content type
    context.Response.ContentType = "text/plain";

    // Write the response
    await context.Response.WriteAsync(responseData);
});
```

# Powershell

List current/new versions of PowerShell:
```powershell
winget list --id Microsoft.Powershell
```

Update PowerShell (minor version):
```powershell
winget upgrade --id Microsoft.PowerShell
```

Upgrade PowerShell (major version):
```poweshell
winget uninstall Microsoft.PowerShell
winget install Microsoft.PowerShell
```

# .NET ADVENTURES

## Console project to WebApp

Guide to convert a console app to a full web app!

```cs
WebApplication? app = null; //cannot find a package to app here to make this valid.
```

We typically should change the Project SDK. Edit the project file:

Change 
```xml
<Project Sdk="Microsoft.NET.Sdk">
```
to:
```xml
<Project Sdk="Microsoft.NET.Sdk.Web"> 
```

By changing the project type - the VS will reload the project (after confirmation) and will generate a `launchSettings.json` file like the one below:
```json
{
  "profiles": {
    "TestLib": {
      "commandName": "Project",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "https://localhost:53511;http://localhost:53512"
    }
  }
}
```
## Publish and Run project WITHOUT server

Include the following in the appsettings.json

```json
"Kestrel": {
   "EndPoints": {
     "Http": {
       "Url": "http://0.0.0.0:5002"
     },
      "Https": {
        "Url": "https://0.0.0.0:5003"
      }
   }
 }
```

For the `launchSettings.json` we can setup by changing the `profiles` section:
```json
 "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "http://0.0.0.0:5257",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://0.0.0.0:7010;http://0.0.0.0:5257",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
```
