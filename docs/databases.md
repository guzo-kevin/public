Updated 4/30/2019

# Oracle XE (18)
[Installation Guide](https://docs.oracle.com/en/database/oracle/oracle-database/18/xeinl/procedure-installing-oracle-database-xe.html)

install preisntall package as root:
```
curl -o oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm
yum -y localinstall oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm

```

Download rpm from download site (it is probably already in my download folder), then install
```
yum -y localinstall oracle-database-xe-18c-1.0-1.x86_64.rpm
```

A very simple configuration file is here: /etc/sysconfig/oracle—xe–18c.conf

run service configuration:

```
# /etc/init.d/oracle-xe-18c configure
Specify a password to be used for database accounts. Oracle recommends that the password entered should be at least 8 characters in length, contain at least 1 uppercase character, 1 lower case character and 1 digit [0-9]. Note that the same password will be used for SYS, SYSTEM and PDBADMIN accounts:
Confirm the password:
Configuring Oracle Listener.
Listener configuration succeeded.
Configuring Oracle Database XE.

........

100% complete
Database creation complete. For details check the logfiles at:
 /opt/oracle/cfgtoollogs/dbca/XE.
Database Information:
Global Database Name:XE
System Identifier(SID):XE
Look at the log file "/opt/oracle/cfgtoollogs/dbca/XE/XE.log" for further details.

Connect to Oracle Database using one of the connect strings:
     Pluggable database: srvdc03c01-lnxd/XEPDB1
     Multitenant container database: srvdc03c01-lnxd
Use https://localhost:5500/em to access Oracle Enterprise Manager for Oracle Database XE
```

Make it autostart
```
systemctl enable oracle-xe-18c
```

Set ORACLE_HOME and PATH for convinience

```
export ORACLE_HOME=/opt/oracle/product/18c/dbhomeXE
export PATH=$ORACLE_HOEM/bin:$PATH
```


Configure tnsnames.ora file from client machine
```
myxe =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 11.22.33.44 )(PORT = 1521))
    (CONNECT_DATA = 
      (SERVER = DEDICATED)
      (SERVICE_NAME = XE)
    )
  )

myxepdb =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 11.22.33.44 )(PORT = 1521))
    (CONNECT_DATA = 
      (SERVER = DEDICATED)
      (SERVICE_NAME = xepdb1)
    )
  )
```

Remove default pdb
```
 alter pluggable database xepdb1 close immediate;
 alter pluggable database xepdb1 unplug into '/opt/oracle/oradata/XE/xepdb1.xml';
 drop pluggable database xepdb1 keep datafiles;
```

Create myOwn pdb
```
CREATE PLUGGABLE DATABASE mynamedev1pdb
  ADMIN USER myname IDENTIFIED BY mypass
  ROLES = (dba)
  DEFAULT TABLESPACE mynamedev1pdb_ts
    DATAFILE '/opt/oracle/oradata/XE/mynamedev1pdb/mynamedev1_ts.dbf' SIZE 1G AUTOEXTEND ON
  FILE_NAME_CONVERT = ('/opt/oracle/oradata/XE/pdbseed/',
                       '/opt/oracle/oradata/XE/mynamedev1pdb/')
  STORAGE (MAXSIZE 16G)
  PATH_PREFIX = '/opt/oracle/oradata/XE/mynamedev1pdb/';
```

By default pluggable database will not open after reboot. Need to save state
```
alter pluggable database pdb_name save state;
```

alter local_listner 
```
alter system set local_listener='(ADDRESS = (PROTOCOL=TCP)(HOST=srvdc03c01-lnxd)(PORT=1521))' scope=both;
```

# SQL Server Developer Edition for Linux (2017)

# MariaDB (5.7)


# MySQL on my Linux Sandbox

#### Install

```bash
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
# yum update
yum install mysql-server
```