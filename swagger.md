# Swagger

```sh
dotnet add package Swashbuckle.AspNetCore
dotnet add package Microsoft.AspNetCore.OpenApi
```



Before building the host add the `AddEndPointApiExplorer` and `AddSwaggerGen` extension calls. After the building add the `UseSwagger`, `UseSwaggerUI` calls:

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
