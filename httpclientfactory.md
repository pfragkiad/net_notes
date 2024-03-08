
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

## Catch httpclient exceptions

If we want to catch connection-related exceptions, we should catch the `TaskCanceledException` (defined by the `httpclient.Timeout` property) and the `HttpRequestException` which happens if there is any connectivity issue and the Timeout has not already passed.

```cs
try
{

    var response = await _httpClient.GetAsync(url);

    string responseText = await response.Content.ReadAsStringAsync();

    if (response.IsSuccessStatusCode)
        return RouteResponse.Parse(responseText, includeCoordinates)!;

    ErrorResponse? errorResponse = ErrorResponse.Parse(responseText);
    return Fail(_logger, "Routing.UnknownError", responseText);
}
catch (TaskCanceledException) //Timeout
{
    return Fail(_logger, "Routing.Timeout",
        "Request to the routing engine failed due to timeout ({timeout} seconds).", ValidatorLogTypeIfError.Error,
        _httpClient.Timeout.TotalSeconds);
}
catch (HttpRequestException) //Connection issue
{
    return Fail(_logger, "Routing.NetworkIssue",
    "Request to the routing engine failed due to network connectivity issue.");
}

catch (Exception ex)
{
    return Fail(_logger, $"Routing.{ex.GetType().Name}", ex.Message);
}


```


