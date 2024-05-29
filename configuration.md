# Configuration

# Manual configuration

Add the required packages first:

```bash
dotnet add package Microsoft.Extensions.Configuration
dotnet add package Microsoft.Extensions.Configuration.Json
dotnet add package Microsoft.Extensions.Configuration.EnvironmentVariables
```

```cs
Configuration = new ConfigurationBuilder()
	.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
	.AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Production"}.json", optional: true)
	.AddEnvironmentVariables()
	.Build();
```

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


JSON settings for Serilog with exceptions (e.g. Microsoft.AspNetCore in this case):

```json
 "Serilog": {
   "MinimumLevel": {
     "Default": "Debug",
     "Override": {
       "Microsoft.AspNetCore": "Warning",
       "System": "Warning"
     }
   },
     "WriteTo": [
       {
         "Name": "Console",
         "Args": {
           "restrictedToMinimumLevel": "Debug"
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
