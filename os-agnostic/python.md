---
description: Exploring exploitation of the Python interpreter and associated scripts.
---

# Python

Python is a language powering an extraordinary number of applications, but here we're just going to focus on very common coding errors you're likely to find. These can be useful in cases of both web applications as well as local and exploiting any of these is incredibly context specific.

## python2 input

In python 2, the input function worked in an interesting manner. It queried for user input, executed the input given to it directly, allowing us to, for example, have an integer returned directly from the result of the function. However, this also allowed any arbitrary code to be injected. In cases where you have a python 2 style input mechanism, simply write the following:

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

