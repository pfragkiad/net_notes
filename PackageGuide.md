# Dependency injection (simple)

```sh
dotnet add package Microsoft.Extensions.DependencyInjection
```
Usage example:

```cs
using System;

using Microsoft.Extensions.DependencyInjection;

namespace DependencyInjectionExample
{
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
}

```
