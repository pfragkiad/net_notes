# net_notes

# Default loggging

Default logging json does not affect Serilog.
```json
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Warning"
    }
  }
```

# Serilog

Include in configuration:

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
 var builder = //_host = 
            Host.
            CreateDefaultBuilder().
            //UseWindowsService(options => //requires Microsoft.Extensions.Hosting.WindowsServices NUGET
            //options.ServiceName = "Ucare").
            //not needed for defaultbuilder
            //ConfigureAppConfiguration(builder => CreateBuilder(builder)).
            ConfigureServices((context, services) =>
            {
                services
                //not needed for default builder
                //.AddLogging(o => o.AddConsole())
                .Configure<UcareOptions>(context.Configuration.GetSection(UcareOptions.UcareOptionsSection))
                //.Configure<HostOptions>(options => options.ShutdownTimeout = TimeSpan.FromSeconds(15))
                //.Configure<HostOptions>(context.Configuration.GetSection("HostOptions"))
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
