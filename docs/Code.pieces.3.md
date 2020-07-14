## argparse inconsistent behavior 

```py
    parser.add_argument("-n","--nodes", nargs="+", action = 'append',dest="NodeList", help="a list of nodes seperated by space")
    parser.add_argument("-g","--group",nargs=1, metavar="",action='store',default="nodes",dest="groupFile", help="a file for list of nodes, one line per node")
```

In the above code, the variable stores input arguments are lists, not variables. (I had assumed the 2nd statement would store the input to  variable  but it was not). And the format of values in the list are dependent on how argument was given.

For example: 

command -n a b -n c  

would produce a list of lists and individual elements options.NodeList like this: [[a,b],c], and options.groupFile='nodes' - groupFile is not a list. 

command -g abc

would produce a list options.groupFile like this: options.groupFile=['abc'] - now groupFile is a list


So options.NodeList need to be flattened. And groupFiles need to be always set to a list. 

newNodeList=sum(options.NodeList,[]) # sum a list with [] would flatten [[a,b],c] to [a, b, c].

ps. to convert list to string seperated by "," :  strnewNodeList = ','.join(newNodeList)


and,

set the default groupFile to ["nodes"] instead of "nodes" would make the input format more consistent. 

```py
    parser.add_argument("-n","--nodes", nargs="+", action = 'append',dest="NodeList", help="a list of nodes seperated by space")
    parser.add_argument("-g","--group",nargs=1, metavar="",action='store',default=["nodes"],dest="groupFile", help="a file for list of nodes, one line per node")
```

* a simple way to process flg argument 
  ```py
     parser.add_argument("--listroles", action = 'store_true',help='list roles in the cluster')
     options = parser.parse_args()
     if options.listroles:
        pass
     
  ```

## Parallel

threading
multiprosessing.dummy.pool
multiprocessing.pool


## Regular Expression

#### Replace part of the match:

Capture the part that you want to keep with parentheses, and include it in the substitution string as $1.

import re
inp = '653433,78'
out = re.sub(r'([0-9]{6}),([0-9]{2})', r'\1.\2', inp)
print out

#### non-greedy

The non-greedy ? works perfectly fine. It's just that you need to select dot matches all option in the regex engines (regexpal, the engine you used, also has this option) you are testing with. This is because, regex engines generally don't match line breaks when you use .. You need to tell them explicitly that you want to match line-breaks too with .

For example,

<img\s.*?>

t_str = re.sub(r'PART 2.*\n(.*\n)*?.*(EFFECTIVE|E F F E C T).*\n','',t_str, flags=re.MULTILINE)

#### re.match

find the first occurence in the first line: (but the first occurence may have multiple elements)
```py
a_str = "guru99 get"
z = re.match("(g\w+)\W(g\w+)", a_str)
if z:
    print((z.groups()))
===========
('guru99','get')
```


#### re.search

return True if find the pattern

```py
if re.search(pattern, text):
    pass

```

#### re.findall


find all occurence 

```py

emails = re.findall(r'[\w\.-]+@[\w\.-]+,abc)

for email in emails:
    print (email)
```


#### match until a pattern apprears
if for one character
[^z]*  -- any character until 'z', 'z' not inlcuded

for multiple charater
(?:(?!X).)* -- where X can be any re expression


