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
