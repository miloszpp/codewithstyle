---
title: Setting up coverage reports on TFS with OpenCover
tags:
  - csharp
  - tfs
  - powershell
url: 181.html
id: 181
categories:
  - .NET
  - Other topics
  - Tutorials
date: 2017-03-30 18:27:25
---

![](https://i.stack.imgur.com/KF67A.jpg)

Code coverage is a metric which indicates the percentage of volume of your source code covered by your tests. It is certainly a good idea to have code coverage reports generated as part of Continuous Integration - it allows you to keep track of quality of your tests or even set requirements for your builds to have a certain coverage. Code coverage in Visual Studio is only available in the Enterprise edition. Fortunately, thanks to OpenCover you can still generate coverage reports even if you don't have access to the Enterprise license. In this article I will show you how to configure a Build Definition on Team Foundation Server 2015/2017 to use OpenCover to produce code coverage reports. 

**UPDATE: The full script is available [here](https://gist.github.com/miloszpp/629f6756185bbd8e3e8c0670f681b8c6).** 

**UPDATE 2: Christian Klutz has created a [VSTS task](https://github.com/cklutz/my-vsts-tasks) based on this article. You may want to check it out. Unfortunately, I won't be able to offer any help on the topic since I haven't been using TFS for some time.**

### Preparations

We are going to put some files on TFS. We will need:

*   RunOpenCover.ps1 - PowerShell script that will run OpenCover - we are going to write it in a moment
*   [vsts-task-lib](https://github.com/Microsoft/vsts-task-lib) \- a PowerShell script library which provides some helpful util functions
*   [OpenCover](https://www.nuget.org/packages/opencover) executable
*   [OpenCoverToCoberturaConverter](https://www.nuget.org/packages/OpenCoverToCoberturaConverter/) \- a tool to convert the report to a format understandable by Visual Studio
*   (optional) [ReportGenerator](https://www.nuget.org/packages/ReportGenerator/) \- a tool do generate HTML reports

The last three items are available as NuGet packages. I suggest organizing all these files into the following directory structure:

```
BuildTools
\* Packages
  \* OpenCover.4.6.519 - the contents of OpenCover package goes here
  \* OpenCoverToCoberturaConverter.0.2.6.0 - the contents of OpenCoverToCoberturaConverter package goes here
  \* ReportGenerator.2.5.6 - the contents of ReportGenerator package goes here
\* Scripts
  \* vsts-task-lib - the contents of vsts-task-lib / powershell / VstsTaskSdk goes here
  \* RunOpenCover.ps1 - the script that we are going to write
```

Once done, check it in to your TFS instance. I've put the BuildTools directory on the top level of the repository. Next, I've added a mapping to my Build Definition in order to make that directory available during the build.

### Create the PowerShell script

Let's now write the PowerShell script. The script is going to perform a couple of steps:

*   We would like our script to use a file pattern to scan for test assemblies in the same way that the "native" _Visual Studio Tests_ task does. For that, we can use _Find-Files_ cmdlet available in _vsts-task-lib_.
*   Next, we run OpenCover and use the list of paths with test assemblies as parameters.
*   Next, we need to convert the results file produced by OpenCover to Cobertura - a file format which TFS can understand.
*   Finally, we can use the same results file to produce an HTML, human-readable report.

The script will take a couple of parameters as input:

```
Param(
    \[string\]$sourcesDirectory, #the root of your project
    \[string\]$testAssembly, #the file pattern describing test assemblies to look for
    \[string\]$testFiltercriteria="", #test filter criteria (as in Run Visual Studio Tests task)
    \[string\]$openCoverFilters="+\[*\]*" #OpenCover-specific filters
)
```

Next, let's run the _Find-Files_ utility to search against the pattern defined in _$testAssembly_. This code is copied from the original [Run Visual Studio Tests task source code](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/VsTest/VSTest.ps1).

```
\# load helper functions from the vsts-task-lib library
. $PSScriptRoot\\vsts-task-lib\\LegacyFindFunctions.ps1
\# resolve test assembly files (copied from VSTest.ps1)
$testAssemblyFiles = @()
\# check for solution pattern
if ($testAssembly.Contains("*") -Or $testAssembly.Contains("?"))
{
    Write-Host "Pattern found in solution parameter. Calling Find-Files."
    Write-Host "Calling Find-Files with pattern: $testAssembly"    
    $testAssemblyFiles = Find-Files -LegacyPattern $testAssembly -LiteralDirectory $sourcesDirectory
    Write-Host "Found files: $testAssemblyFiles"
}
else
{
    Write-Host "No Pattern found in solution parameter."
    $testAssembly = $testAssembly.Replace(';;', "`0") # Barrowed from Legacy File Handler
    foreach ($assembly in $testAssembly.Split(";"))
    {
        $testAssemblyFiles += ,($assembly.Replace("`0",";"))
    }
}
\# build test assembly files string for vstest
$testFilesString = ""
foreach ($file in $testAssemblyFiles) {
    $testFilesString = $testFilesString + " ""$file"""
}
```

We can finally run OpenCover. The command to do this is pretty complicated. OpenCover supports different test runners (VSTest being only one of them) so we need to specify the path to VSTest as one of the arguments. The path below `(%VS140COMNTOOLS%..\\IDE\\CommonExtensions\\Microsoft\\TestWindow\\vstest.console.exe`) is valid for Visual Studio 2015 installation. Another important argument is -mergebyhash . It forces OpenCover to treat assemblies with the same hash as one. I've spent a few hours figuring out why my coverage score is so low. It turned out that OpenCover analyzed few copies of the same assembly.

```
Start-Process "$PSScriptRoot\\..\\Packages\\OpenCover.4.6.519\\OpenCover.Console.exe" -wait -NoNewWindow -ArgumentList "-register:user -filter:""$OpenCoverFilters"" -target:""%VS140COMNTOOLS%\\..\\IDE\\CommonExtensions\\Microsoft\\TestWindow\\vstest.console.exe"" -targetargs:""$testFilesString /TestCaseFilter:$testFiltercriteria /logger:trx"" -output:OpenCover.xml -mergebyhash" -WorkingDirectory $PSScriptRoot
```

Next, let's convert the results generated by OpenCover to Cobertura format.

```
Start-Process "$PSScriptRoot\\..\\Packages\\OpenCoverToCoberturaConverter.0.2.6.0\\tools\\OpenCoverToCoberturaConverter.exe" -Wait -NoNewWindow -ArgumentList "-input:""$PSScriptRoot\\OpenCover.xml"" -output:""$PSScriptRoot\\Cobertura.xml"" -sources:""$sourcesDirectory"""
```

Finally, we will generate a HTML report based on the results from OpenCover.

```
Start-Process "$PSScriptRoot\\..\\Packages\\ReportGenerator.2.5.6\\tools\\ReportGenerator.exe" -Wait -NoNewWindow -ArgumentList "-reports:""$PSScriptRoot\\OpenCover.xml"" -targetdir:""$PSScriptRoot\\CoverageReport"""
```

And that's it.

### Configure the Build Definition

We will need to add three build steps to our Build Definition. If you have a _Visual Studio Tests_ task in it, remove it - you will no longer need it.

*   _PowerShell_ task - set the Script Path to point to RunOpenCover.ps1 and specify the Arguments:
```
-sourcesDirectory "$(Build.SourcesDirectory)" -testAssembly "**\\*.Tests.dll;-:**\\obj\\**" -testFiltercriteria "TestCategory!=INTEGRATION"
```
*   _Publish Test Results_ task - configure it as on the image below; as a by-product of generating coverage reports, we produce test results - we need to tell TFS where to find them

![](/images/2017/03/opencover1-1.png)

*   _Publish Code Coverage Results_ task - configure it as on the image below; thanks to this task the results will be visible  on the build summary page

![](/images/2017/03/opencover2-1.png) 

And that's it! Run the build definition and enjoy your code coverage results. You can find the on the build summary page. The HTML report is available as one of the build artifacts.