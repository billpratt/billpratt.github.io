---
layout: post
title: 'dotnet publish: Where are my XML docs?'
date: 2017-03-29 20:37:37.000000000 -04:00
permalink: /dotnet-publish-where-are-my-xml-docs/
categories:
  - dotnet
  - asp.net
  - asp.net-core
---
## An ASP.NET Core project.json to csproj migration gotcha

In ASP.NET Core project.json, you could generate XML documentation for your application by adding the following line:

```
"buildOptions": {
    "xmlDoc": true
}
```

When running `dotnet build` or `dotnet publish`, the XML documentation would appear in the build and publish output directories as shown here:

![](/assets/images/2017/03/sc1-1.png)

![](/assets/images/2017/03/sc2.png)

Life is great, your API Swagger documentation is lookin' good and even

**Chuck Norris approves!**

![](/assets/images/2017/03/chuck-norris-thumbs-up.jpg)

## Migrating to csproj
Microsoft made the decision to align ASP.NET Core projects with the rest of the .NET ecosystem by moving to the **csproj** format while still bringing over the good bits of project.json. It makes sense. I won't go into all of the details but there are plenty of good articles out there, including:

* <a href="https://docs.microsoft.com/en-us/dotnet/articles/core/tools/project-json-to-csproj" target="_blank">A mapping between project.json and csproj properties</a> 
* <a href="http://www.natemcmaster.com/blog/2017/01/19/project-json-to-csproj/" target="_blank">Project.json to MSBuild conversion guide</a>

Running `dotnet migrate project.json`, you will see a new **csproj** file and project.json is gone. If you open the csproj file you should see a ```<PropertyGroup>``` like the one below. Notice the `<GenerateDocumentationFile>true</GenerateDocumentationFile>` line. 

![](/assets/images/2017/03/sc3.png)

Running `dotnet build` on the csproj file, you should see the XML documentation in the build output directory as shown below. Great!

![](/assets/images/2017/03/sc4.png)

Next, let's verify that running `dotnet publish` on the csproj still outputs the XML documentation.

![](/assets/images/2017/03/sc5.png)

![](/assets/images/2017/03/huh-1.png)

As of this posting, when running `dotnet publish` against a **csproj** ASP.NET Core project, your application's XML documentation is nowhere to be found. There is an open <a href="https://github.com/dotnet/sdk/issues/795" target="_blank">Github issue #795</a> on the dotnet/sdk repository where you can track the progress of a fix.

## Workaround until fixed
All hope is not lost. There are a few different proposed workarounds on the Github issue. I have found the following solution to work for me. Open the **csproj** file and copy/paste the following:

```
<Target Name="PrepublishScript" BeforeTargets="PrepareForPublish">
  <ItemGroup>
    <DocFile Include="bin\$(Configuration)\$(TargetFramework)\*.xml" />
  </ItemGroup>
  <Copy SourceFiles="@(DocFile)" DestinationFolder="$(PublishDir)" SkipUnchangedFiles="false" />
</Target>
```

This will ensure that all XML documentation files in the build configuration (Debug|Release|etc) output directory you are targeting will get copied to the publish directory.

You can also remove the `<GenerateDocumentationFile>true</GenerateDocumentationFile>` line and replace it with two new new property groups:

```
<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
  <DocumentationFile>bin\Debug\netcoreapp1.1\[YOUR_APP_NAME].xml</DocumentationFile>
</PropertyGroup>
```
```
<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">
  <DocumentationFile>bin\Release\netcoreapp1.1\[YOUR_APP_NAME].xml</DocumentationFile>
</PropertyGroup>
```

## Microsoft in the open source space
While it never feels "great" to add in a workaround, I applaud the .NET teams at Microsoft for embracing open source and the faster release cycles of features. I have found it very enlightening to file a new issue on Github and actually get fast responses/fixes! While nothing is perfect, .NET in the open "is the beginning of a beautiful friendship".

Feedback is always welcomed. Cheers!
