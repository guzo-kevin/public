# VS Code (1.33)

__Where is the settings.json__

* The settings are stored in settings.json
* It seemed the User/Master settings.json is in C:\Users\~\AppData\Roaming\Code\User
* Each directory can have .vscode\settings.json, which is also called workspace setting
* There are 2 ways to change setting, both can be launched through command panel \<ctrl-p\>. Both some settings, i.e Termial Args can only be set in Json.
    1. \> Preference: Open Setting (UI)
    2. \> Preference: Open Setting (JSON)
  

### Python environment

* In this configuration I used one of the virtual environment created in Anaconda
```
{
    "python.pythonPath": "C:\\Users\\username\\Programs\\Continuum\\anaconda3\\envs\\main\\python.exe"
}
```
### Terminal

* in this configuration I create a anaconda shell goes directly into "main" virtual environment 
```
{
    "terminal.integrated.shell.windows": "C:\\Windows\\System32\\cmd.exe",
    "terminal.integrated.shellArgs.windows": [
        "/K" "C:\\Users\\username\\Programs\\Continuum\\anaconda3\\Scripts\\activate.bat C:\\Users\\username\\Programs\\Continuum\\anaconda3\\envs\\main"
    ]
}
```

### Terminal - with Shell Launcher

* I need to use different type of shell for different needs. So I installed an extention called "Shell Launcher"
* I put following in user settings.json
```
{
    "shellLauncher.shells.windows": [
        {
            "shell": "C:\\Windows\\System32\\cmd.exe",
            "args": ["/K", "C:\\Users\\username\\Programs\\Continuum\\anaconda3\\Scripts\\activate.bat C:\\Users\\username\\Programs\\Continuum\\anaconda3\\envs\\main"],
            "label":"Anaconda Main",
            "launchName":"Anaconda Main",
        },
        {
            "shell": "C:\\Windows\\System32\\cmd.exe",
            "label":"Cmd",
            "launchName":"Windows Cmd",
        },
        {
            "shell": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
            "label":"PowerShell",
            "launchName":"PowerShell",
        },
        {
            "shell": "C:\\Program Files\\Git\\bin\\bash.exe",
            "label":"Git Bash",
            "launchName":"Git Bash",
        },
    ]
}
```

* I still need to customize the "terminal.integrated.shell.windows". Otherwise the default terminal when I press control-` will get me a Powershell window, which I don't know much about.
  * Note: the argument for git bash is "l" like lower case "L", not "1" 
```
{
    "shellLauncher.shells.windows": [
        {
            "shell": "C:\\Windows\\System32\\cmd.exe",
            "args": ["/K", "C:\\Users\\username\\Programs\\Continuum\\anaconda3\\Scripts\\activate.bat C:\\Users\\username\\Programs\\Continuum\\anaconda3\\envs\\main"],
            "label":"Anaconda Main",
            "launchName":"Anaconda Main",
        },
        {
            "shell": "C:\\Windows\\System32\\cmd.exe",
            "label":"Cmd",
            "launchName":"Windows Cmd",
        },
        {
            "shell": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
            "label":"PowerShell",
            "launchName":"PowerShell",
        },
        {
            "shell": "C:\\Program Files\\Git\\bin\\bash.exe",
            "label":"Git Bash",
            "launchName":"Git Bash",
        },
    ],

    "terminal.integrated.shell.windows": "C:\\Program Files\\Git\\bin\\bash.exe",
    "terminal.integrated.shellArgs.windows": [
        "-l"
    ],
}
```


### Extentions installed

* Markdown All in One
* PowerShell
* Python
* shell launcher


# Oracle sqlcl (18.4)

* I installed SQL CL, which does not work with MINGW64 terminal (git bash) very well. 

* I modified ~/Programs/sqlcl/bin/sql and added MINGW64* line in following section
```
        #
        # Check for Cygwin
        #
        cygwin=false
        case `uname` in
                CYGWIN*) cygwin=true;;
                MINGW64*) cygwin=true;;
        esac
```

# Oracle SQL Developer (18.4)
* I installed jtds jdbc client for SQL Developer to connect to SQL Server
  * the MS SQL Server JDBC 7.2 does not work with SQL Developer 
  * the SSO (dll) need to be in system path environment in order for Windows Integrated Authentidation to work
* 


# Oracle Thin Client (12.2)

# Oracle XE (18)

# SQL Server Developer Edition for Linux (2017)

# Python/Anaconda Environment

* I installed Anaconda Python 3.7 64-bit 2018.12 for Windows
* I created a virtual environment called "main", the default environment is "base(root)"
* In main I installed:
  * cx_oracle (for oracle)
  * pyodbc (for sql server)
  * m2-util-linux (for utilities)
  * numpy, matplotlib, pandas
  * jupyter
