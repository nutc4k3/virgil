# Web Applications

## Toolkit

The list of tools for attacking web applications is pretty extensive, but here I'll include the ones I most commonly use:

### wfuzz

[Wfuzz](https://github.com/xmendez/wfuzz), the web fuzzer, is pretty much the Swiss Army Knife of fuzzing applications, allowing you to fuzz a ridiculous number of parameters in a web request.  This can be anything from directories to Host headers.  

#### Subdomain Discovery

```bash
wfuzz -c  -Z -z file,/root/namelist.txt --hh 0,560 -H "Host: FUZZ.app.com" http://192.168.0.72 
```

#### Login Brute-Force

```bash
wfuzz -c -z file,./passwords.txt -d 'user=admin&pass=FUZZ' --hh 45 http://192.168.0.75:8080/login.php 
```

### thc-hydra

If you have to, use [hydra](https://github.com/vanhauser-thc/thc-hydra).  The query syntax is archaic as hell, but it's almost a standard at this point so it's always worth having it in your toolkit.  Modules include options for everything from web login forms to SSH.



