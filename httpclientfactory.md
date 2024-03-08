
## Add HttpClientFactory

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
