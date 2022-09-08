

# Configuration

## Add custom configuration via extension

`CustomOptions.CustomOptionsSection` is assumed to be the string of the top level correspoding element within the settings json file.

```cs
   public static IServiceCollection AddCrossynServices(
        this IServiceCollection services, HostBuilderContext context)
    {
        return services
            .Configure<CustomOptions>(context.Configuration.GetSection(CustomOptions.CustomOptionsSection))
    }
```

## 2 quick ways to create App with appsettings.json configured

Create a host container:
```cs
IHost app = Host.CreateDefaultBuilder(args).Build();
var config = app.Services.GetService<IConfiguration>();
```
Create a configuration only without any other service:
```cs
var config = new ConfigurationBuilder().AddJsonFile("appsettings.json").Build();
```

## Default loggging

Default logging json does not affect Serilog.
```json
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Warning"
    }
  }
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
                services
                //not needed for default builder
                //.AddLogging(o => o.AddConsole())
                .Configure<UcareOptions>(context.Configuration.GetSection(UcareOptions.UcareOptionsSection))
                //.Configure<HostOptions>(options => options.ShutdownTimeout = TimeSpan.FromSeconds(15))
                .....  
                ;
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



