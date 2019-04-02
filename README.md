

[![logo](https://raw.githubusercontent.com/amoffat/sh/master/logo-230.png)](https://github.com/pwwang/sh)


Forked from https://github.com/amoffat/sh, see [original documentation](https://amoffat.github.com/sh)

## Install this fork
```
pip install git+https://github.com/pwwang/sh.git
```

## Features being added to this fork:
### Simulate pipe commands:
```python
from sh import ls, grep
# list all files containing 'a'
# Have to set _piped = True to send the process to next command
ls(_piped = True) | grep('a')
# Also work for chaining
ls(_piped = True) | grep('a', _piped = True) | grep('b')
```

### Simulate shell redirection
```python
# use content of a.txt as input
cat(_in = '<') < 'a.txt'
# You can also send the output of another command to current one
cat(_in = '<') < ls('a.txt')
# redirect stdout to a file
# Have to set `_out = '>'` to hold the spawning
ls(_out = '>') > 'a.txt'
ls(_out = '>') > ls('a.txt')
# append stdout to a file
ls(_out = '>') >> 'a.txt'

# or you can bake a new command:
ls_out = ls.bake(_out = '>')
ls_out() > 'a.txt'
ls_out() >> 'a.txt'

# you can do the same for _err, but you have to use *, and **
ls_err = ls.bake(_err = '>')
ls_err() * 'a.txt'
ls_err() ** 'a.txt'
```

### Change `_long_sep` and `_long_prefix` to `_sep` and `_prefix`, respectively 
`_sep` will then affect both short and long options and defaults to `" "`. It can be set as `auto`, so that space (`" "`) will be used for short options and `"="` will be used for long.  
`_prefix` defaults to `auto`, meaning '"-"' for short options and `"--"` for long.  
Say we have this command:
```python
cat(_in = '<') < 'a.txt'
# a.2
# b.1
# c.0
```
Then we want to sort it by the first part and then second part seperated by `"."`:
```python
(cat(_in = '<', _piped = True) < 'a.txt') | sort(t = '.', k = ['1,1', '2,2n'])
```
An exception will be raised:
```
ErrorReturnCode_2: 

  RAN: /bin/sort -k 1,1 2,2n -t .
                               ^
```
Notice the space, here we need to set to seperator as an empty string, not only for long options. To make it work, we also add another arugment `_duplistkey`, which allows the key to be duplicated when a `list`/`tuple` to be assign as the value for keyword option:
```python
(cat(_in = '<', _piped = True) < 'a.txt') | sort(t = '.', k = ['1,1', '2,2n'], _sep = '', _duplistkey = True)
# ['/bin/sort', '-k1,1', '-k2,2n', '-t.'], notice the duplicated '-k'
```

### Allow configurations for commonly used commands
Since some commands always favor a certain way to assemble options, we could save it to a configuration file, so that we don't need to worry about it in the future any more.  
This required `pyyaml` to be installed

#### Global configuration
The default global configuration file is `~/.sh.yaml`
```yaml
ll:
    l: True
sort:
    _sep: ''
    _duplistkey: True
cat_in:
    path: cat
    _in: <
```
```python
(cat_in(_piped = True) < 'a.txt') | sort(t = '.', k = ['1,1', '2,2n'])
# no need to worry about _sep and _duplistkey any more!
```

#### Project configuration
You may also save some configurations in current work directory: `./.sh.yaml`
```yaml
cat_in:
    path: cat
    _in: <
    _piped: True
```
```python
(cat_in() < 'a.txt') | sort(t = '.', k = ['1,1', '2,2n'])
```
The configuration for `sort` is remained, but `cat_in` is updated.  
Note that it is not a recursive update, which means that the whole configuration will be replaced.  
So in the above case if `./.sh.yaml` is like:
```yaml
cat_in:
    _in: <
    _piped: True
```
Then the executable for `cat_in` will default to `cat_in` instead of `cat`, which will raise `CommandNotFound` exception.

#### Loading other configuration file
```python
from sh import BAKED_ARGS
BAKED_ARGS['_load'](<config file>)
# this has to be placed before importing other commands
```

#### In-memory configuration
```python
from sh import BAKED_ARGS
BAKED_ARGS['ll'] = dict(path = 'ls', l = True)
# this has to be placed before importing other commands
from sh import ll
```
Of course you bake a `sh` module for a set of commands like the original `sh` package:
```python
import sh
sh2 = sh(_sep = '')
from sh2 import sort, cut
```

