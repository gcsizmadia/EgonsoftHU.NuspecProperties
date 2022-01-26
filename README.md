# MSBuild support for deprecated NuGet metadata elements

According to [.nuspec File Reference for NuGet](https://docs.microsoft.com/en-us/nuget/reference/nuspec) the following metadata elements are deprecated:

- [owners](https://docs.microsoft.com/en-us/nuget/reference/nuspec#owners)
  > `owners` is deprecated. Use `authors` instead.
- [summary](https://docs.microsoft.com/en-us/nuget/reference/nuspec#summary)
  > `summary` is being deprecated. Use `description` instead.

However, you might still want to include these metadata elements in your NuGet package for whatever reason.

## Solution

I have created a `.targets` file for MSBuild which can utilize the `<Owners>` and the `<Summary>` MSBuild properties.

All you have to do is some simple steps:
- Save `EgonsoftHU.NuspecProperties.targets` file to a location from where it can be referenced from your solution/project.
- Import `EgonsoftHU.NuspecProperties.targets` file into your project file (e.g. `.csproj`) or into your `Directory.Build.targets` file, if you have such file.
- Set `<Owners>` and/or `<Summary>` MSBuild properties.

### Import the `.targets` file

Let's assume you have a Git repository with these files:
- `{gitrepo}\Directory.Build.targets`
- `{gitrepo}\Build\EgonsoftHU.NuspecProperties.targets`

The content of the `Directory.Build.targets` file may look like this:
```xml
<Project>

  <PropertyGroup>
    <EgonsoftHUNuspecPropertiesTargetsFile>$(MSBuildThisFileDirectory)Build\EgonsoftHU.NuspecProperties.targets</EgonsoftHUNuspecPropertiesTargetsFile>
  </PropertyGroup>

  <Import Project="$(EgonsoftHUNuspecPropertiesTargetsFile)" Condition="Exists('$(EgonsoftHUNuspecPropertiesTargetsFile)')" />

</Project>
```

### Set `<Owners>` and/or `<Summary>` MSBuild properties

Let's assume you have a Git repository with this file:
- `{gitrepo}\src\YourProject\YourProject.csproj`

The content of the `YourProject.csproj` file may look like this:
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <Authors>Your profile name on nuget.org</Authors>
    <Owners>Your profile name on nuget.org</Owners>
    <Description>A description of the package for UI display. When uploading a package to nuget.org, the description field is limited to 4000 characters.</Description>
    <Summary>A short description of the package for UI display. If omitted, a truncated version of description is used.</Summary>
  </PropertyGroup>

  <!-- The rest is omitted for clarity. -->

</Project>
```

## How it works

When the `Pack` MSBuild target finishes then the followings happen:
- The `.nupkg` file will be extracted to a temporary folder.
- If either the `<Owners>` or the `<Summary>` MSBuild property is set then the extracted `.nuspec` file will be modified.
  - The `owners` element will be added after `authors` element.
  - The `summary` element will be added adter `description` element.
- The content of the temporary folder will be zipped into the original `.nupkg` file by overwriting it.
- The temporary folder will be deleted.

### Example build output

```
======================================
Egonsoft.HU NuGet package updater v1.0
======================================
Author        : Gabor Csizmadia
Description   : Update .nuspec file by adding owners and summary elements using Owners and Summary MSBuild properties.
--------------------------------------
Creating temporary folder...
  Folder      : [P:\GIT\EgonsoftHU.Extensions\nupkgs\243a2f2f-35e3-4e88-b7f6-1c9f8876d145]
--------------------------------------
Unzipping .nupkg file to the temporary folder...
  .nupkg file : [P:\GIT\EgonsoftHU.Extensions\nupkgs\EgonsoftHU.Extensions.Bcl.1.0.0.nupkg]
  Folder      : [P:\GIT\EgonsoftHU.Extensions\nupkgs\243a2f2f-35e3-4e88-b7f6-1c9f8876d145]
--------------------------------------
Updating .nuspec file...
  .nuspec file: [P:\GIT\EgonsoftHU.Extensions\nupkgs\243a2f2f-35e3-4e88-b7f6-1c9f8876d145\EgonsoftHU.Extensions.Bcl.nuspec]
  Owners      : [Egonsoft.HU]
  Summary     : [Extension methods for BCL types]

The .nuspec file was found.
Loading .nuspec file...
Getting default XML namespace...
Getting <metadata> node...
Getting <authors> node...
Appending <owners> node...
Getting <description> node...
Appending <summary> node...
Saving .nuspec file...
--------------------------------------
Creating new .nupkg file from the contents of the temporary folder...
  .nupkg file : [P:\GIT\EgonsoftHU.Extensions\nupkgs\EgonsoftHU.Extensions.Bcl.1.0.0.nupkg]
  Folder      : [P:\GIT\EgonsoftHU.Extensions\nupkgs\243a2f2f-35e3-4e88-b7f6-1c9f8876d145]

Zipping directory "P:\GIT\EgonsoftHU.Extensions\nupkgs\243a2f2f-35e3-4e88-b7f6-1c9f8876d145" to "P:\GIT\EgonsoftHU.Extensions\nupkgs\EgonsoftHU.Extensions.Bcl.1.0.0.nupkg".
--------------------------------------
Deleting temporary folder...
  Folder      : [P:\GIT\EgonsoftHU.Extensions\nupkgs\243a2f2f-35e3-4e88-b7f6-1c9f8876d145]
--------------------------------------
The new .nupkg file has been created...
  .nupkg file : [P:\GIT\EgonsoftHU.Extensions\nupkgs\EgonsoftHU.Extensions.Bcl.1.0.0.nupkg]
======================================
```

## Notes

### `Directory.Build.props` and `Directory.Build.targets` files

If you are not yet familiar with these files then I suggest to check how to [Customize your build](https://docs.microsoft.com/en-us/visualstudio/msbuild/customize-your-build?view=vs-2022).
