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

## Parallel

threading
multiprosessing.dummy.pool
multiprocessing.pool

