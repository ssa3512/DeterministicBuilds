# Deterministic Builds

This repo shows how to get a fully deterministic build using [Source Link](https://github.com/dotnet/sourcelink).
Deterministic builds are important as they enable verification that the resulting binary was built from 
the specified source and provides traceability. 

Deterministic builds require a property to be set to true during CI:
`ContinuousIntegrationBuild`. These should not be enabled during local dev or the debugger
won't be able to find the local source files.

Therefore, you should use your CI system's variable to set them conditionally. For Azure Pipelines, it 
looks like this

```xml
<PropertyGroup Condition="'$(TF_BUILD)' == 'True'">
  <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
</PropertyGroup>
```

For GitHub Actions, the variable is `GITHUB_ACTIONS`, so the result would be:
```xml
<PropertyGroup Condition="'$(GITHUB_ACTIONS)' == 'true'">
  <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
</PropertyGroup>
```


`EmbedUntrackedSources` should also be set to true so that compiler-generated source, like AssemblyInfo, are included
in the PDB. Note that there's a [workaround](https://github.com/dotnet/sourcelink/issues/572) needed for many SDK's prior to 3.1.300. You'll need to add
a `Directory.Build.targets` file with the following:

```xml
<Project>
  <PropertyGroup>
    <TargetFrameworkMonikerAssemblyAttributesPath>$([System.IO.Path]::Combine('$(IntermediateOutputPath)','$(TargetFrameworkMoniker).AssemblyAttributes$(DefaultLanguageSourceExtension)'))</TargetFrameworkMonikerAssemblyAttributesPath>
  </PropertyGroup>
  <ItemGroup>
    <EmbeddedFiles Include="$(GeneratedAssemblyInfoFile)"/>
  </ItemGroup>
</Project>

```

 
## Building locally
To see/test this locally, build with `dotnet build /p:TF_BUILD=True`. If you examine the resulting package in [NuGet Package Explorer](https://github.com/NuGetPackageExplorer/NuGetPackageExplorer),
it will pass.
