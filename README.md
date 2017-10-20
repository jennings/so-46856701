# FileNotFoundException when preprocessing T4 templates from build server

This is a reproduction for [Stack Overflow question
46856701](https://stackoverflow.com/q/46856701).

I'm trying to follow the instructions in Microsoft's document [Code Generation
in a Build Process][codegen] to rebuild T4 templates on our build server. When
I build, template generation is failing with the error (the full MSBuild output
is later in this question):

> error MSB4018: The "TransformTemplates" task failed unexpectedly.
>
> error MSB4018: System.IO.FileNotFoundException: Could not load file or
> assembly 'Microsoft.VisualStudio.TextTemplating.14.0, Version=14.0.0.0,
> Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies.
> The system cannot find the file specified.

I created [a minimal project that reproduces the problem][repro]. The script
_build.cmd_ simply executes MSBuild.

[codegen]: https://msdn.microsoft.com/en-us/library/ee847423.aspx
[repro]: https://github.com/jennings/so-46856701

## Things I've tried:

* Copied the MSBuild and Visual Studio assemblies to the build server, as
  specified by the _Code Generation_ document.

* Added the Visual Studio assemblies to the GAC.

* Set the `VSSDK140Install` environment variable to "C:\Program Files
  (x86)\Microsoft Visual Studio 14.0\VSSDK\" (because that's how it's set on
  my workstation).

* Set `HKLM\SOFTWARE\WOW6432Node\Microsoft\VisualStudio\14.0!InstallDir` to
  `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\ `

I'm trying to avoid installing Visual Studio 2015, because that's a pretty
large dependency to add to our build server, and the Microsoft documentation
explicitly says this should be possible.


### Copied assemblies to C:\Program Files (x86)

I've copied the following files to the build server, as specified in the _Code
Generation_ document (these are all in the _Program Files (x86)_ folder, since
that's where they are on my machine):

* $(ProgramFiles)\MSBuild\Microsoft\VisualStudio\v*.0\TextTemplating
    * Microsoft.VisualStudio.TextTemplating.Sdk.Host.14.0.dll
    * Microsoft.TextTemplating.Build.Tasks.dll
    * Microsoft.TextTemplating.targets
* $(ProgramFiles)\Microsoft Visual Studio 14.0\VSSDK\VisualStudioIntegration\Common\Assemblies\v4.0
    * Microsoft.VisualStudio.TextTemplating.14.0.dll
    * Microsoft.VisualStudio.TextTemplating.Interfaces.14.0.dll (several files)
    * Microsoft.VisualStudio.TextTemplating.VSHost.14.0.dll
* $(ProgramFiles)\Microsoft Visual Studio 14.0\Common7\IDE\PublicAssemblies\
    * Microsoft.VisualStudio.TextTemplating.Modeling.14.0.dll


### Added to the GAC

I saw it's failing trying to load
`Microsoft.VisualStudio.TextTemplating.14.0.dll`, and since that assembly is in
the GAC on my workstation I guessed maybe that was required, so I tried using
[GAC Manager][gacmanager] to add
`Microsoft.VisualStudio.TextTemplating.14.0.dll` to the GAC, but the error is
the same.

[gacmanager]: https://github.com/dwmkerr/gacmanager


## The full MSBuild output

When I run `build.cmd`, I get the following output:

    D:\TTTest>"C:\Program Files (x86)\MSBuild\14.0\Bin\MSBuild.exe" ClassLibrary1.sln /t:Rebuild
    Microsoft (R) Build Engine version 14.0.25420.1
    Copyright (C) Microsoft Corporation. All rights reserved.

    Building the projects in this solution one at a time. To enable parallel build, please add the "/m" switch.
    Build started 10/20/2017 11:05:16 AM.
    Project "D:\TTTest\ClassLibrary1.sln" on node 1 (Rebuild target(s)).
    ValidateSolutionConfiguration:
      Building solution configuration "Debug|Any CPU".
    Project "D:\TTTest\ClassLibrary1.sln" (1) is building "D:\TTTest\ClassLibrary1\ClassLibrary1.csproj" (2) on node 1 (Rebuild target(s)).
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: The "TransformTemplates" task failed unexpectedly. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: System.IO.FileNotFoundException: Could not load file or assembly 'Microsoft.VisualStudio.TextTemplating.14.0, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies. The system cannot find the file specified. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: File name: 'Microsoft.VisualStudio.TextTemplating.14.0, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018:    at Microsoft.VisualStudio.TextTemplating.Sdk.Host.GenericTextTemplatingHost..ctor(IServiceProvider serviceProvider) [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018:    at Microsoft.VisualStudio.TextTemplating.Build.Tasks.TransformTemplatesBase.GetConfiguredTextTemplatingHost() [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018:    at Microsoft.VisualStudio.TextTemplating.Build.Tasks.TransformTemplatesBase.Execute() [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018:    at Microsoft.Build.BackEnd.TaskExecutionHost.Microsoft.Build.BackEnd.ITaskExecutionHost.Execute() [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018:    at Microsoft.Build.BackEnd.TaskBuilder.<ExecuteInstantiatedTask>d__26.MoveNext() [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018:  [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: === Pre-bind state information === [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: DisplayName = Microsoft.VisualStudio.TextTemplating.14.0, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018:  (Fully-specified) [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Appbase = file:///C:/Program Files (x86)/MSBuild/14.0/Bin/ [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Initial PrivatePath = NULL [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: Calling assembly : Microsoft.VisualStudio.TextTemplating.Sdk.Host.14.0, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: === [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: This bind starts in LoadFrom load context. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: WRN: Native image will not be probed in LoadFrom context. Native image will only be probed in default load context, like with Assembly.Load(). [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Using application configuration file: C:\Program Files (x86)\MSBuild\14.0\Bin\MSBuild.exe.Config [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Using host configuration file:  [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Using machine configuration file from C:\Windows\Microsoft.NET\Framework\v4.0.30319\config\machine.config. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Post-policy reference: Microsoft.VisualStudio.TextTemplating.14.0, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Attempting download of new URL file:///C:/Program Files (x86)/MSBuild/14.0/Bin/Microsoft.VisualStudio.TextTemplating.14.0.DLL. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Attempting download of new URL file:///C:/Program Files (x86)/MSBuild/14.0/Bin/Microsoft.VisualStudio.TextTemplating.14.0/Microsoft.VisualStudio.TextTemplating.14.0.DLL. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Attempting download of new URL file:///C:/Program Files (x86)/MSBuild/14.0/Bin/Microsoft.VisualStudio.TextTemplating.14.0.EXE. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Attempting download of new URL file:///C:/Program Files (x86)/MSBuild/14.0/Bin/Microsoft.VisualStudio.TextTemplating.14.0/Microsoft.VisualStudio.TextTemplating.14.0.EXE. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Attempting download of new URL file:///C:/Program Files (x86)/MSBuild/Microsoft/VisualStudio/v14.0/TextTemplating/Microsoft.VisualStudio.TextTemplating.14.0.DLL. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Attempting download of new URL file:///C:/Program Files (x86)/MSBuild/Microsoft/VisualStudio/v14.0/TextTemplating/Microsoft.VisualStudio.TextTemplating.14.0/Microsoft.VisualStudio.TextTemplating.14.0.DLL. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Attempting download of new URL file:///C:/Program Files (x86)/MSBuild/Microsoft/VisualStudio/v14.0/TextTemplating/Microsoft.VisualStudio.TextTemplating.14.0.EXE. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018: LOG: Attempting download of new URL file:///C:/Program Files (x86)/MSBuild/Microsoft/VisualStudio/v14.0/TextTemplating/Microsoft.VisualStudio.TextTemplating.14.0/Microsoft.VisualStudio.TextTemplating.14.0.EXE. [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v14.0\TextTemplating\Microsoft.TextTemplating.targets(396,5): error MSB4018:  [D:\TTTest\ClassLibrary1\ClassLibrary1.csproj]
    Done Building Project "D:\TTTest\ClassLibrary1\ClassLibrary1.csproj" (Rebuild target(s)) -- FAILED.

    Done Building Project "D:\TTTest\ClassLibrary1.sln" (Rebuild target(s)) -- FAILED.


    Build FAILED.
