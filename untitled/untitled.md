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



