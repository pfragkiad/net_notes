# Add item to Output

Copy item to output directory (preserve newest). Other options are (`Always`, `Never`):

```xml
<ItemGroup>
	<None Update="pers.html">
		<CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
	</None>
</ItemGroup>

```

# ASP.NET Core support in Class library

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

# Winforms and Core

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
