# PHP

PHP is an interesting language with a number of quirks. Since it was designed for simple personal websites and not as a full blown enterprise grade language, this comes as no shock. Unfortunately, this means older version are easily exposed to a number of obvious exploits.

Some of these will be outdated, and in these situations I'll refer to the documentation. Other's are still relevant and quite common. In all situation when attacking any PHP application, or performing source code auditing, I can not recommend enough that you keep the [documentation](http://php.net/docs.php) close to hand. You can find a number of comments for each function which will describe best practices, and often ways in which a function can be misused.

## strcmp

## Type Juggling

With weak-comparisons in PHP, such as in the case of the `==` operator, the comparison of two hashes beginning with the integer 0, will cause the expression to reduce to `0==0`. These following table lists the most common hashes and the values you can input to produce them:

| Hash | Magic String |  |
| :--- | :--- | :--- |
| MD5 | 240610708 | 0e462097431906509019562988736854 |
| SHA1 | 10932435112 | 0e07766915004133176347055865026311692244 |

**Source**: [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/PHP%20juggling%20type](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/PHP%20juggling%20type)

[WhiteHatSec](https://www.whitehatsec.com/blog/magic-hashes/) maintains quite an extensive table also for various other hash functions.

## Regular Expressions

Take, as an example, `preg_replace`:

```text
preg_replace('/(.*)/', 'A', 'B');
```

This will just result in the string 'A' being output as everything is replaced in the second string. If we change the regular expression however:

```text
preg_replace('/(.*)/e', 'phpinfo()', 'B');
```

From the above, the second string will be replaced resulting in 'phpinfo\(\)'. The `/e` modifier appended to the regular expression, causes the result to be evaluated as PHP code, effectively giving you arbitrary command execution. In this instance we'll be returned the output of `phpinfo()`.

**References**    
[http://www.madirish.net/402](http://www.madirish.net/402)  
[https://bitquark.co.uk/blog/2013/07/23/the\_unexpected\_dangers\_of\_preg\_replace](https://bitquark.co.uk/blog/2013/07/23/the_unexpected_dangers_of_preg_replace)  




