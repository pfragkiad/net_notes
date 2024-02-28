# Dependency injection (simple)

```sh
dotnet add package Microsoft.Extensions.DependencyInjection
```
Usage example:

```cs
using System;

using Microsoft.Extensions.DependencyInjection;

namespace DependencyInjectionExample;

public interface IMessageService
{
    void SendMessage(string message);
}

public class ConsoleMessageService : IMessageService
{
    public void SendMessage(string message)
    {
        Console.WriteLine(message);
    }
}

class Program
{
    static void Main(string[] args)
    {
        // Create a service collection
        var services = new ServiceCollection();

        // Register services
        services.AddSingleton<IMessageService, ConsoleMessageService>();

        // Build the service provider
        var serviceProvider = services.BuildServiceProvider();

        // Resolve the service
        var messageService = serviceProvider.GetRequiredService<IMessageService>();

        // Use the service
        messageService.SendMessage("Hello, Dependency Injection!");

        Console.ReadLine();
    }
}

```

# Configuration

```sh
dotnet add package Microsoft.Extensions.Configuration
dotnet add package Microsoft.Extensions.Configuration.Json
```

Example code:
```cs
using System;
using Microsoft.Extensions.Configuration;

namespace ConfigurationExample;

class Program
{
    static void Main(string[] args)
    {
        // Setup configuration
        IConfiguration config = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
            .Build();

        // Access configuration values
        string connectionString = config["ConnectionStrings:DefaultConnection"];
        int timeout = config.GetValue<int>("TimeoutSeconds");

        Console.WriteLine($"Connection String: {connectionString}");
        Console.WriteLine($"Timeout: {timeout}");

        Console.ReadLine();
    }
}

```

Add `IConfiguration` to `ServiceCollection`:
```cs
var services = new ServiceCollection();
services.AddSingleton<IConfiguration>( _ =>  new ConfigurationBuilder()
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: false)
    .Build());
var provider = services.BuildServiceProvider();

var c = provider.GetRequiredService<IConfiguration>();
System.Console.WriteLine(c["genericBasePath"]);
```
Remember to add the `appsettings.json` file in the project file in order to copy it to the executable directory:
```xml
<ItemGroup>
	<None Update="appsettings.json">
		<CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
	</None>
</ItemGroup>
```


# Hosting

```sh
dotnet add package Microsoft.Extensions.Hosting
```

```cs
IHost host = Host
.CreateDefaultBuilder(args)
.ConfigureServices((context, services) => { })
.Build();
```

# ASP.NET Core stuff

Types like `IResult` are available with a `FrameworkReference`, not a `PackageReference`. The project file should be modified, like the example below:

```xml
<PropertyGroup>
	<TargetFramework>net8.0</TargetFramework>
	<ImplicitUsings>enable</ImplicitUsings>
	<Nullable>enable</Nullable>
	<PublishAot>true</PublishAot>
</PropertyGroup>

<ItemGroup>
	<PackageReference Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
</ItemGroup>

<ItemGroup>
	<FrameworkReference Include="Microsoft.AspNetCore.App" />
</ItemGroup>
```
