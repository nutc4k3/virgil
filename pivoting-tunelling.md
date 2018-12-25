# Pivoting/Tunelling

## Tunneling

There are a number of useful tools for getting a route deeper into a network, so here I will attempt to summarise and describe the most useful.

### 

## Port Forwarding

There are a myriad of ways to forward ports from one host to another.  By far the most popular is known as SSH tunelling in which the traffic from one host to another is tunelled over the SSH protocol.

We can perform local port forwards if we have SSH access to the machine we want to compromise, where we forward that remote port to a port on our local computer.  Reverse port forwards do exactly the same but the initiation of the SSH connection ocurrs in reverse.  You'd use this if you have code execution on a device, but no SSH credentials and you want to port forward one of it's local ports.

```text
ssh -L <localport>:<remotehost>:<remoteport> username@remotehost    # local portfwd
ssh -R <remoteport>:<remotehost>:<localport> username@remotehost     # reverse portfwd
```

On Windows \(and Linux if no SSH client is available\) the same is achieved with [plink.exe](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html):

```text
plink.exe -L <localport>:<remotehost>:<remoteport> username@remotehost
plink.exe -R <remoteport>:<remotehost>:<localport> username@remotehost
```

These can also be used to bind locally if an inbound firewall is blocked.  We can also use netcat and socat to achieve this:

```bash
socat TCP-LISTEN:8080,fork,reuseaddr TCP:127.0.0.1:80
nc -l -p 8080 -c "nc 127.0.0.1 80"
```

### sshuttle

If you're in a situation where you have access to a device which you can connect to via SSH, and want to pivot deeper into the network painlessly, then [sshuttle ](https://github.com/sshuttle/sshuttle)is the tool for you.  This was indispensable during my OSCP course, as it allows you to forgo having to forward a myriad of ports, as you just SSH to a device and will automatically set up a route on your local Kali.

## Useful Tools

### busybox

The [busybox](https://www.busybox.net/) client is commonly used in embedded Linux to package up a number of useful tools, but in cases where you're pivoting through docker containers it gives a number of incredibly useful commands in environment where a lot of the more common tools aren't available.

It even offers a cut down version of nc with the `-e` flag available:

```bash
BusyBox v1.28.1 (2018-02-15 14:34:02 CET) multi-call binary.

Usage: nc [OPTIONS] HOST PORT  - connect
nc [OPTIONS] -l -p PORT [HOST] [PORT]  - listen

	-e PROG	Run PROG after connect (must be last)
	-l	Listen mode, for inbound connects
	-lk	With -e, provides persistent server
	-p PORT	Local port
	-s ADDR	Local address
	-w SEC	Timeout for connects and final net reads
	-i SEC	Delay interval for lines sent
	-n	Don't do DNS resolution
	-u	UDP mode
	-v	Verbose
	-o FILE	Hex dump traffic
	-z	Zero-I/O mode (scanning)

```

### static-binaries

{% embed url="https://github.com/andrew-d/static-binaries" %}

An incredibly useful repository containing a number of tools in static-binary form.  Needless to say this is very useful when you have need of a specific tool not contained in something like busybox.  It contains static python and socat binaries.

### dropbear

{% embed url="http://matt.ucc.asn.au/dropbear/" %}

In the same vein as the above, dropbear provides a tiny self-contained SSH client and server which can be used in systems with no native SSH client available.  Again, perfect for pivoting between bespoke or cut-down systems.

## References

[https://pentest.blog/explore-hidden-networks-with-double-pivoting/](https://pentest.blog/explore-hidden-networks-with-double-pivoting/)  
[https://github.com/sshuttle/sshuttle](https://github.com/sshuttle/sshuttle)  
[https://www.offensive-security.com/metasploit-unleashed/portfwd/](https://www.offensive-security.com/metasploit-unleashed/portfwd/)  
[https://unix.stackexchange.com/questions/10428/simple-way-to-create-a-tunnel-from-one-local-port-to-another](https://unix.stackexchange.com/questions/10428/simple-way-to-create-a-tunnel-from-one-local-port-to-another)







