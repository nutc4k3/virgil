---
description: Accessing Hidden Services and Networks
---

# Pivoting

## Port Forwarding

There are a myriad of ways to forward ports from one host to another.  By far the most popular is known as SSH tunelling in which the traffic from one host to another is tunelled over the SSH protocol.

We can perform local port forwards if we have SSH access to the machine we want to compromise, where we forward that remote port to a port on our local computer.  Reverse port forwards do exactly the same but the initiation of the SSH connection ocurrs in reverse.  You'd use this if you have code execution on a device, but no SSH credentials and you want to port forward one of it's local ports.

```text
ssh -L <localport>:<remotehost>:<remoteport> username@remotehost    # local portfwd
ssh -R <remoteport>:<localhost>:<localport> username@remotehost     # reverse portfwd
```

## References

[https://pentest.blog/explore-hidden-networks-with-double-pivoting/](https://pentest.blog/explore-hidden-networks-with-double-pivoting/)



