# Swagger

```sh
dotnet add package Swashbuckle.AspNetCore
dotnet add package Microsoft.AspNetCore.OpenApi
```



Before building the host, add the `AddEndPointApiExplorer` and `AddSwaggerGen` extension calls. After the building, add the `UseSwagger`, `UseSwaggerUI` calls:

```cs
IServiceCollection services = builder.Services;
services.AddEndpointsApiExplorer();
services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new() { Title = "GreenLight v1", Version = "v1" });
});

WebApplication app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI(c =>
{
    c.DocumentTitle = "GreenLight API by e:misia";
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "GreenLight v1");
});
```

Add example endpoints at the end:

```cs
var gRouting = app.MapGroup("/routing").WithTags("Routing");


//https://learn.microsoft.com/en-us/aspnet/core/fundamentals/native-aot?view=aspnetcore-8.0
gRouting.MapGet("/status", GetEngineStatus)
.Produces<EngineStatus>()
.WithName("routineStatus")
.WithSummary("Get the status of the routing engine.")
.WithOpenApi()
;

gRouting.MapGet("/route", GetRoute)
.Produces<RouteResponse>(200)
.Produces<Error<string>>(400)
.WithName("getRoute")
.WithSummary("Get the route information for all sections.")
.WithOpenApi(o =>
{
    o.Parameters[0].Description = "Origin latitude in degrees.";
    o.Parameters[1].Description = "Origin longitude in degrees.";
    o.Parameters[2].Description = "Destination latitude in degrees.";
    o.Parameters[3].Description = "Destination longitude in degrees.";
    return o;
});
```
