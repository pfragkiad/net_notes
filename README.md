`Da Notes`

# Configuration

## Add custom configuration files
```cs
private static void Initialize()
{
host = Host.CreateDefaultBuilder()
    .ConfigureAppConfiguration(builder => CreateBuilder(builder))
    .ConfigureServices((context, services) =>
    {
	services
	.AddLogging(o => o.AddConsole())
	.AddTransient<IRecorder,Recorder>()
	.AddSingleton<IRecorderFormatFactory,RecorderFormatFactory>()
	.AddSingleton<IRecorderFactory,RecorderFactory>()
       ;
    })
    .Build();
}

private static IConfigurationBuilder CreateBuilder(IConfigurationBuilder builder)
{
return builder
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Production"}.json", optional: true)
    .AddEnvironmentVariables();
}
```

## Add custom config options

`CustomOptions.CustomOptionsSection` is assumed to be the string (as a static field) of the top level correspoding element within the settings json file.
Sample call to add to services:

```cs
    
var app = Host
.CreateDefaultBuilder()
.ConfigureServices( (context, services) =>
{
    //to enable tihs Microsoft.Extensions.Hosting must be referenced
    services.Configure<CustomOptions>(context.Configuration.GetSection(CustomOptions.CustomOptionsSection));
}
).Build()
...
var options = app.Services.GetRequiredService<IOptions<CustomOptions>>().Value;
```
To instantiate a `CustomOptions` object we can alternatively use the `IConfiguration` like the example below:
```cs
var configuration = app.Services.GetRequiredService<IConfiguration>();
CustomOptions options = new CustomOptions();
configuration.GetSection(CustomOptions.CustomOptionsSection).Bind(options);

//or
options = configuration.GetSection(CustomOptions.CustomOptionsSection).Get<CustomOptions>();
```

## 2 quick ways to create App with appsettings.json configured

Create a host container:
```cs
IHost app = Host.CreateDefaultBuilder(args).Build();
var config = app.Services.GetService<IConfiguration>();
```
Or create a configuration only without any other service:
```cs
var config = new ConfigurationBuilder().AddJsonFile("appsettings.json").Build();
```

## Default logging

Default logging json does not affect Serilog.
```json
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Warning"
    }
  }
```

## Create custom logger in static context

```cs
IHost app = Host.CreateDefaultBuilder(args).Build();
...
var logger = app.Services.GetRequiredService<ILoggerFactory>().CreateLogger("Program");
```

## Serilog

To use Serilog, include the package `Serilog.AspNetCore`. In the `appsettings.json` configuration file, include the following section:

```json
 "Serilog": {
    "MinimumLevel": "Debug",
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "restrictedToMinimumLevel": "Information"
        }
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/logfile_project.txt",
          "rollingInterval": "Hour"
        }
      }
    ]
  }
```

To use in C# configure the `IHostBuilder`:

```cs
 var builder = 
    Host.
    CreateDefaultBuilder().
    ConfigureServices((context, services) =>
    {
	//services
	//not needed for default builder
	//.AddLogging(o => o.AddConsole());
    }).UseSerilog((context, configuration) =>
    {
	//NUGET: Serial.Settings.Configuration
	//for configuration see: https://github.com/serilog/serilog-settings-configuration
	configuration.ReadFrom.Configuration(context.Configuration); //read Serilog options from appsettings.json

	//configuration.MinimumLevel.Debug();
	//configuration.WriteTo.Console(restrictedToMinimumLevel:Serilog.Events.LogEventLevel.Information);
	//configuration.WriteTo.File(path: "logs/myapp.txt", rollingInterval: RollingInterval.Hour);
    }); //IHostBuilder

```
To configure the `WebApplicationBuilder`:

```cs
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);
builder.Host.UseSerilog((context, configuration) =>
{
    //NUGET: Serial.Settings.Configuration
    //for configuration see: https://github.com/serilog/serilog-settings-configuration
    configuration.ReadFrom.Configuration(context.Configuration); //read Serilog options from appsettings.json

    //configuration.MinimumLevel.Debug();
    //configuration.WriteTo.Console(restrictedToMinimumLevel:Serilog.Events.LogEventLevel.Information);
    //configuration.WriteTo.File(path: "logs/myapp.txt", rollingInterval: RollingInterval.Hour);
}); //IHostBuilder
```
## Add HttpCLientFactory

Add named `HttpClientFactory`:
```cs
WebApplicationBuilder builder = WebApplication.CreateBuilders(args);
IServiceCollection services = builder.Services;
IConfiguration configuration = builder.Configuration;
services.AddHttpClient("road", c => c.BaseAddress = new Uri(configuration["RoadRoutingUrl"] ?? "http://localhost:8969/"));
```

Add typed `HttpClientFactory` that accepts all certificates:
```cs
services.AddHttpClient<MyClient>().ConfigurePrimaryHttpMessageHandler(() =>
new HttpClientHandler
{
    ClientCertificateOptions = ClientCertificateOption.Manual,
    ServerCertificateCustomValidationCallback = (httpRequestMessage, cert, cetChain, policyErrors) => true
});

public class MyClient
{
    private readonly MyOptions _options;
    private readonly HttpClient _httpClient;

    public MyClient(HttpClient httpClient, IOptions<MyOptions> options)
    {
        _options = options.Value;
        _httpClient = httpClient;
        _httpClient.BaseAddress = new Uri(_options.Url);
    }
}
```

# Winforms and Core

## .NET Core project with Windows.Forms support

Edit the project file as shown below (the `PropertyGroup` is a child node of `Project`:
We change the target framework to `net6.0-windows` and we add the `UseWindowsForms` node and set the content to `true`.

```xml
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <UseWindowsForms>true</UseWindowsForms>
  </PropertyGroup>
```

## Winforms logging with `ILogger`

The project file of the class library should be edited to allow the use of Windows Forms controls. Also, the project type should be bound to Windows:
```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">
	<PropertyGroup>
		<TargetFramework>net6.0-windows</TargetFramework>
		<ImplicitUsings>enable</ImplicitUsings>
		<Nullable>enable</Nullable>
		<UseWindowsForms>true</UseWindowsForms>
	</PropertyGroup>
</Project>

```

The dependencies should appear as below:

![Project dependencies](img/windowsforms.png "WindowsForms dependency").

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

# Edit C# project file

Copy item to output directory (preserve newest). Other options are (`Always`, `Never`):

```xml
<ItemGroup>
	<None Update="pers.html">
		<CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
	</None>
</ItemGroup>

```
