# Windows

## Alternate Data Streams

This is more a  feature of the NTFS file system than it is a Windows feature, but it's important to understand from both a red-team and blue-team perspective.  

These are also disturbingly common in CTF's through the security via obscurity, so we need some method of viewing and potentially executing these alternate data streams.  To list all alternate data streams 

```text
dir /R
```

### Powershell

```bash
Get-Content -path C:\Users\Booj\ads -stream *
```

We can also view individual streams using

```text
Get-Content -path C:\Users\Booj\ads -stream <streamname>
```

### Windows Pre-Vista

In Windows prior to Vista, there's unfortunately no easy way of viewing these streams in default Windows without a GUI access.  In an engagement this may not be entirely practical.

The best version, working on XP, I found to be [LADS](https://www.aldeid.com/wiki/LADS), but a number of the links online appear to be dead.  The [Web Archive](http://web.archive.org/web/20150602054446/http://www.heysoft.de/download/lads.zip) version is still up so acquiring a copy is still possible.

