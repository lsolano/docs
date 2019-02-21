> ﻿NOTE:
> As of the 3.0 final release, the registry settings are no longer recognized. Instead, use settings in the `.runsettings` file. 


## NUnit 3.x

### VS Test .Runsettings configuration
Certain NUnit Test Adapter settings are configurable using a .runsettings file. 
The following options are available:

|Key|Type|Options| Default|
|---|----|-------|--------------|
|[InternalTraceLevel](#InternalTraceLevel)| string |  Off, Error, Warning, Info, Verbose,  Debug| Nothing => Off|
|[NumberOfTestWorkers](#NumberOfTestWorkers)| int | nr of workers | -1|
|[ShadowCopyFiles](#ShadowCopyFiles)| bool |True, False | False|
|[Verbosity](#Verbosity)| int | -1, 0 -5 . -1 means quiet mode | 0|
|[UseVsKeepEngineRunning](#UseVsKeepEngineRunning)| bool | True, False| False|
|[BasePath](#BasePath)| string | path| ?|
|[PrivateBinPath](#PrivateBinPath) | string| directory1;directory2;etc |?|
|[RandomSeed](#RandomSeed) int | seed integer| random|
|[DefaultTimeout](#DefaultTimeout)|int|timeout in mS, 0 means infinite|0|
|[DefaultTestNamePattern](#DefaultTestNamePattern)|string|Pattern for display name|{m}{a}|
|[DomainUsage](#DomainUsage)|string| None, Single, Multiple|Single|
|[WorkDirectory](#WorkDirectory)|string|specify directory|Test assembly location|
|[TestOutputXml](#TestOutputXml)|string|specify directory|Test Result Xml output folder|
|[DumpXmlTestDiscovery](#DumpXmlTestDiscovery-and-DumpXmlTestResults)|bool|Enable dumping of NUnit discovery response xml|false|
|[DumpXmlTestResults](#DumpXmlTestDiscovery-and-DumpXmlTestResults)|bool|Enable dumping of NUnit execution response xml|false|
|[ShowInternalProperties](#ShowInternalProperties)| bool | Turn on showing internal NUnit properties in Test Explorer| false|


### Visual Studio templates for runsettings
You can install [item templates for runsettings](https://marketplace.visualstudio.com/items?itemName=OsirisTerje.Runsettings-19151) in Visual Studio (applies to version 2017, 2019 and upwards) which includes the NUnit settings mentioned here.  Note that there are available seperate installs for earlier Visual Studio versions, links to these can be found in the above. 


### Example implementation
See https://github.com/nunit/nunit3-vs-adapter/blob/8a9b8a38b7f808a4a78598542ddaf557950c6790/demo/demo.runsettings

### NUnit .runsettings implementation

https://github.com/nunit/nunit3-vs-adapter/blob/master/src/NUnitTestAdapter/AdapterSettings.cs#L143


### Details

#### WorkDirectory
Our WorkDirectory is the place where output files are intended to be saved for the run, whether created by NUnit or by the user, who can access the work directory using TestContext. It's different from TestDirectory, which is the directory containing the test assembly. For a run with multiple assemblies, there could be multiple TestDirectories, but only one WorkDirectory.
User sets work directory to tell NUnit where to put stuff like the XML or any text output. User may also access it in the code and save other things there. Think of it as the directory for saving stuff. 

#### TestOutputXml
If this is specified, the adapter will generate NUnit Test Result Xml data in the folder specified here.  If not specified, no such output will be generated.  
The folder can be

1) An absolute path
2) A relative path, which is then relative to either WorkDirectory, or if this is not specified, relative to the current directory, as defined by .net runtime.

#### InternalTraceLevel
This setting is a diagnostic setting forwarded to NUnit, and not used by the adapter itself.  For further information see the [NUnit Tracelevel documentation](https://github.com/nunit/docs/wiki/Internal-Trace-Spec)

#### NumberOfTestWorkers
This  setting is sent to NUnit to determine how  [parallelization](https://github.com/nunit/docs/wiki/Parallelizable-Attribute) should be performed.  

#### ShadowCopyFiles
This setting is sent to NUnit to enable/disable shadow-copying. 

#### Verbosity
This controls the outputs from the adapter to the Visual Studio Output/Tests window.
A higher number includes the information from the lower numbers.
It has the following actual levels:

-1 : Quiet mode.  Only shows errors and warnings.  

0 : Default, normal information verbosity

1-3: Some more information from setting are output (in particular regarding parallelization)

4: Outputs the values from the  runsettings it has discovered.

5: Outputs all debug statements in the adapter



#### UseVsKeepEngineRunning
This setting is used by the adapter to signal to the VSTest.Execution engine to keep running after the tests have finished running.  This can speed up execution of subsequent test runs, as the execution engine already is loaded, but running the risks of either holding onto test assemblies and having some tests not properly cleaned out.   The settings is the same as using the Visual Studio  Test/Test Settings/Keep Test Execution Engine running. 

#### DumpXmlTestDiscovery and DumpXmlTestResults
These settings are used to dump the output from NUnit, as it is received by the adapter, before any processing in the adapter is done, to disk.  It is part of the diagnostics tools for the adapter. 
You can find the files under your current outputfolder, in a subfolder named Dump. 
(Note: This is not the same as the TestResults folder, this data is not testresults, but diagnostics dumps)


#### ShowInternalProperties
The [NUnit internal properties](https://github.com/nunit/nunit/blob/master/src/NUnitFramework/framework/Internal/PropertyNames.cs) have been "over-populating" in the Test Explorer.  These are default filtered out, although you may still see these when you have [Source Based Discovery (SBD)](https://docs.microsoft.com/en-us/visualstudio/test/test-explorer-faq?view=vs-2017) turned on (which is the default in VS).  Once you have run test execution, they will be gone. We expect this part of the issue (SBD) to be fixed in VS.  If you still want to see them, set this property to true. 


#### Some further information on directories (From [comment on issue 575](https://github.com/nunit/nunit3-vs-adapter/issues/575#issuecomment-445786421) by [Charlie](https://github.com/CharliePoole) )

NUnit also supports TestContext.TestDirectory, which is the directory where the current test assembly is located. Note that if you have several test assemblies in different directories, the value will be different when each one of them accesses it. Note also that there is no way you can set the TestDirectory because it's always where the assembly is located.

The BasePath is a .NET thing. It's the base directory where assemblies are searched for. You can also have subdirectories listed in the PrivateBinPath. NUnit take scare of all this automatically now, so the old console options are no longer supported. For finding things you want to read at runtime, the TestDirectory and the BasePath will usually be the same thing.



## NUnit 2.x


### Registry Settings 

Certain settings in the registry affect how the adapter runs. All these settings are added by using RegEdit under the key **HKCU\Software\nunit.org\VSAdapter**.

#### ShadowCopy

By default NUnit no longer uses shadowcopy. If this causes an issue for you shadowcopy can be enabled by setting the DWORD value UseShadowCopy to 1.   
  
#### KeepEngineRunning

By default the NUnit adapter will "Kill" the Visual Studio Test Execution engine after each run. Visual Studio 2013 and later has a new setting under its top menu, **Test | Test Settings | Keep Test Execution Engine Running**. The adapter normally ignores this setting.

In some cases it can be useful to have the engine running, e.g. during debugging of the adapter itself. You can then set the adapter to follow the VS setting by setting the DWORD value UseVsKeepEngineRunning to 1.

#### Verbosity

Normally the adapter reports exceptions using a short format, consisting of the message only. You can change it to report a verbose format that includes the stack trace, by setting a the DWORD value Verbosity to 1.




