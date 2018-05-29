# Pivoting/Tunelling

## Port Forwarding

There are a myriad of ways to forward ports from one host to another.  By far the most popular is known as SSH tunelling in which the traffic from one host to another is tunelled over the SSH protocol.

We can perform local port forwards if we have SSH access to the machine we want to compromise, where we forward that remote port to a port on our local computer.  Reverse port forwards do exactly the same but the initiation of the SSH connection ocurrs in reverse.  You'd use this if you have code execution on a device, but no SSH credentials and you want to port forward one of it's local ports.

```text
ssh -L <localport>:<remotehost>:<remoteport> username@remotehost    # local portfwd
ssh -R <remoteport>:<localhost>:<localport> username@remotehost     # reverse portfwd
```

These can also be used to bind locally if an inbound firewall is blocked.  The following take advantage of netcat and socat to achieve this.

```bash
socat TCP-LISTEN:8080,fork,reuseaddr TCP:127.0.0.1:80
nc -l -p 8080 -c "nc 127.0.0.1 80"
```

## Tunelling 

### sshuttle

If you're in a situation where you have access to a device which you can connect to via SSH, and want to pivot deeper into the network painlessly, then [sshuttle ](https://github.com/sshuttle/sshuttle)is the tool for you.  This was indispensable during my OSCP course, as it allows you to forgo having to forward a myriad of ports, and allows you to just SSH to a device and will automatically set up a route on your local Kali.

## References

[https://pentest.blog/explore-hidden-networks-with-double-pivoting/](https://pentest.blog/explore-hidden-networks-with-double-pivoting/)  
[https://github.com/sshuttle/sshuttle](https://github.com/sshuttle/sshuttle)  
[https://www.offensive-security.com/metasploit-unleashed/portfwd/](https://www.offensive-security.com/metasploit-unleashed/portfwd/)  
[https://unix.stackexchange.com/questions/10428/simple-way-to-create-a-tunnel-from-one-local-port-to-another](https://unix.stackexchange.com/questions/10428/simple-way-to-create-a-tunnel-from-one-local-port-to-another)







