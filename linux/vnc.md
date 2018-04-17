# VNC

## Connecting

```text
vncviewer <remote ip>:<desktop>
```

`<desktop>` is an integer signifying the xterm session attached to a port. e.g. 5902 would be :2. You can use the following command to see if the VNC session has an authentication

```text
nmap -sV -sC <target> -p <port>
```

## Cracking Passwords

Passwords in VNC are obfuscated but not heavily encrypted. We can use a tool like [vncpwd ](https://github.com/jeroennijhof/vncpwd)to crack any hashes we find.

