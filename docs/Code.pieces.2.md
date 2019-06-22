## Realtime output from Shell command

Copied from [here](https://zaiste.net/realtime_output_from_shell_command_in_python/)

Running a shell command in Python usually waits until the process is finished and only then sends its entire output. In the following example it is process.communicate() that blocks till given command is completed.
```
import subprocess
import shlex

command = shlex.split("ping -c 5 google.com")
process = subprocess.Popen(command, stdout=subprocess.PIPE)
output, err = process.communicate()
print output
```
subprocess module allows to capture realtime output as well - here's how:

```
from subprocess import Popen, PIPE

def run(command):
    process = Popen(command, stdout=PIPE, shell=True)
    while True:
        line = process.stdout.readline().rstrip()
        if not line:
            break
        yield line


if __name__ == "__main__":
    for path in run("ping -c 5 google.com"):
        print path
```


## Print to the same line on screen

```python
for i in range(10):
    print(i, end = " ")

Output:
0 1 2 3 4 5 6 7 8 9
```

To overwrite a line
```
for i in range(10):
    print(i, end = "\r")
```

## Strange behavior of gitbash handling / in command line arguments

git bash some how converts command line arguments starting with "/" to a local path. This is really annoying. 

for example when run the following application:
```py
import sys
print (str(sys.argv))
```

```bash
$ python args1.py /tmp
['args1.py', 'C:/Users/dev_t/AppData/Local/Temp']
```

good thing is I was not alone. Here is a link to a bug report but it was not seen as a bug. 
https://github.com/git-for-windows/git/issues/1462

the fix is to export MSSYS_NO_PATHCONV=1

```bash
$ export MSYS_NO_PATHCONV=1

$ python args1.py /tmp
['args1.py', '/tmp']
```



## argparse List and Variable in consistant behavior

to be added