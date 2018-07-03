---
title: Configure Kerberos Authentication for Oracle 12c in Windows 2016 AD
date: 2018-06-20 22:12:53
tags:
- Oracle
- Kerberos
- Active directory
---

# Why

In order to test if Progress Oracle wire-protocol ODBC driver supports Kerberos authentication, I spent several days configuring Kerberos authentication for our test Oracle server.

Basically, the configuration consists of 3 machine provisioned in Azure:
- A Windows 2016 AD Domain controller (with domain name SSIS.COM)
- A RHEL machine that runs Oracle 12.1 on it
- A Windows 2016 machine that acts as client.

# References

- Create Active Directory Forest in Azure
  - [Create a new Active Directory Forest in Azure](https://www.youtube.com/watch?v=_HmQO43vgNs&index=2&list=LLm2waTfwmV5e8m8oH2BFjrg&t=0s)
- Configure Kerberos Auth with Active Directory
  - [Configuring Kerberos Authentication](https://docs.oracle.com/database/121/DBSEG/asokerb.htm)
  - [Windows Single Sign-On to an Oracle database using Kerberos](https://jolliffe.hk/2018/02/24/windows-single-sign-on-to-an-oracle-database-using-kerberos/)
  - [Configuring Kerberos for Oracle Databases 11.2 with win2008R2 AD](https://bjornnaessens.wordpress.com/2012/12/21/configuring-kerberos-for-oracle-databases-11-2-with-win2008r2-ad/)
  - [Active Directory Authentication with an Oracle Database 
March 19, 2016](http://oracledbtales.blogspot.com/2016/03/active-directory-authentication-with.html)

# Step by step

<!-- more -->

## Create 3 VMs in Azure, under the same VNET

| Server name | Internal IP | Role | DNS Server | Domain |
| :--- | :--- | :--- | :--- | :--- |
| siduOracle0 (Win) | 10.10.12.4 | Domain controller | 10.10.12.4 / 5 | ssis.com |
| siduOracle1 (Win) | 10.10.12.5 | Windows client | 10.10.12.4 / 5 | ssis.com |
| siduOracle998 (RHEL) | 10.10.12.9 | Oracle server | 10.10.12.4, Auto | - |

- Open ports like 22, 88, 1521, 3389;

- Create AD forest on siduOracle0, create user test;

- Add siduOracle1 to domain, use test to login;

- Install Oracle database on siduOracle998.

## On siduOracle0

Create a service account `servora` in AD for database server to valiade the Kerberos tickets with:

Check option “Do not require Kerberos PreAuthentication” for this user;

Extract a keytab file for this user so we don’t need to enter password to create tickets:

```
ktpass -princ oracle/siduoracle998.xxx.cx.internal.cloudapp.net@SSIS.COM -ptype KRB5_NT_PRINCIPAL -crypto RC4-HMAC-NT -mapuser servora -pass Passw0rd -out C:\sidu\krb5.keytab
```

Make sure **oracle/siduoracle998.xxx.cx.internal.cloudapp.net** is in lower case!

Because on Linux side, when user uses the file to login to server, klist shows:

```
[oracle@siduOracle998 admin]$ klist
Ticket cache: FILE:/tmp/krb5cc_54321
Default principal: test@SSIS.COM
 
Valid starting     Expires            Service principal
06/12/18 23:10:20  06/13/18 09:10:23  krbtgt/SSIS.COM@SSIS.COM
renew until 06/13/18 23:10:20
06/12/18 23:10:25  06/13/18 09:10:23  oracle/siduoracle998.xxx.cx.internal.cloudapp.net@
renew until 06/13/18 23:10:20
06/12/18 23:10:25  06/13/18 09:10:23  oracle/siduoracle998.xxx.cx.internal.cloudapp.net@SSIS.COM
renew until 06/13/18 23:10:20
```

Service principals are always transferred to lower case. If use upper case we'll fail to get authenticated.

Copy this file to the DB server as `/etc/krb5.keytab`

## On siduOracle998

Install Kerberos client:

```
yum -y install krb5-workstation
```

Validate generated `krb5.keytab`:

```
[oracle@siduOracle998 admin]$ klist -kte /etc/krb5.keytab
Keytab name: FILE:/etc/krb5.keytab
KVNO Timestamp         Principal
---- ----------------- --------------------------------------------------------
   7 12/31/69 16:00:00 oracle/siduoracle998.xxx.cx.internal.cloudapp.net@SSIS.COM (arcfour-hmac)
```

Generate a kerberos ticket, this will be used for connection to the kerberos server for ticket validation:

```
[oracle@siduOracle998 admin]$ kinit -k -t /etc/krb5.keytab oracle/siduoracle998.xxx.cx.internal.cloudapp.net@SSIS.COM
[oracle@siduOracle998 admin]$ klist
Ticket cache: FILE:/tmp/krb5cc_54321
Default principal: oracle/siduoracle998.xxx.cx.internal.cloudapp.net@SSIS.COM

Valid starting     Expires            Service principal
06/13/18 00:09:50  06/13/18 10:09:50  krbtgt/SSIS.COM@SSIS.COM
renew until 06/14/18 00:09:50
[oracle@siduOracle998 admin]$ kdestroy
[oracle@siduOracle998 admin]$ klist
klist: No credentials cache found (ticket cache FILE:/tmp/krb5cc_54321)
```

Edit `/etc/krb5.conf` like this:

```
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log
 
[libdefaults]
 default_realm=SSIS.COM
 
[realms]
 SSIS.COM= {
  kdc=siduOracle0:88
  admin_server=siduOracle0:88
 }
 
[domain_realm]
 ssis.com=SSIS.COM
 .ssis.com=SSIS.COM
```

Validate config:

```
[oracle@siduOracle998 admin]$ kinit test
Password for test@SSIS.COM: 
[oracle@siduOracle998 admin]$ klist
Ticket cache: FILE:/tmp/krb5cc_54321
Default principal: test@SSIS.COM
 
Valid starting     Expires            Service principal
06/13/18 00:16:24  06/13/18 10:16:26  krbtgt/SSIS.COM@SSIS.COM
renew until 06/14/18 00:16:24
[oracle@siduOracle998 admin]$ kdestroy
```

Adjust the `sqlnet.ora`:

```
SQLNET.KERBEROS5_CONF=/etc/krb5.conf
SQLNET.KERBEROS5_CONF_MIT=true
SQLNET.KERBEROS5_KEYTAB=/etc/krb5.keytab
SQLNET.AUTHENTICATION_KERBEROS5_SERVICE=oracle
SQLNET.AUTHENTICATION_SERVICES=(BEQ,KERBEROS5)
 
# TRACE_LEVEL_CLIENT=16
# TRACE_LEVEL_SERVER=16
```

`tnsnames.ora`:

```
MAINORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = siduOracle998.xxx.cx.internal.cloudapp.net)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = maindb)
    )
  )
```

## On the database server (siduOracle998)

Clear OS_AUTHENT_PREFIX:

```
SQL> alter system set OS_AUTHENT_PREFIX=’’ scope=spfile;
```

Restart database (optional?):

```
[oracle@siduOracle998 admin]$ sqlplus / as sysdba
 
SQL*Plus: Release 12.1.0.2.0 Production on Tue Jun 12 20:08:12 2018
 
Copyright (c) 1982, 2014, Oracle.  All rights reserved.
 
 
Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
 
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.
 
Total System Global Area 5049942016 bytes
Fixed Size                    2934792 bytes
Variable Size                 1040189432 bytes
Database Buffers         3992977408 bytes
Redo Buffers                   13840384 bytes
Database mounted.
Database opened.
```

Create user `test@SSIS.COM`:

```
alter session set "_ORACLE_SCRIPT"=true;
...
create user "TEST@SSIS.COM" identified externally; // user name MUST be upper-case!
...
GRANT Connect, Resource, DBA to "TEST@SSIS.COM";
...
select * from dba_users; (or select username from dba_users;)
...
```

Validate on the server:
```
[oracle@siduOracle998 admin]$ kinit test
Password for test@SSIS.COM: 
[oracle@siduOracle998 admin]$ sqlplus /@mainorcl
 
SQL*Plus: Release 12.1.0.2.0 Production on Wed Jun 13 00:29:13 2018
 
Copyright (c) 1982, 2014, Oracle.  All rights reserved.
 
Last Successful login time: Tue Jun 12 2018 23:10:25 -07:00
 
Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
 
SQL> show user;
USER is "TEST@SSIS.COM"
SQL> exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
[oracle@siduOracle998 admin]$ klist
Ticket cache: FILE:/tmp/krb5cc_54321
Default principal: test@SSIS.COM
 
Valid starting     Expires            Service principal
06/13/18 00:16:24  06/13/18 10:16:26  krbtgt/SSIS.COM@SSIS.COM
renew until 06/14/18 00:16:24
06/13/18 00:29:13  06/13/18 10:16:26  oracle/siduoracle998.xxx.cx.internal.cloudapp.net@
renew until 06/14/18 00:16:24
06/13/18 00:29:13  06/13/18 10:16:26  oracle/siduoracle998.xxx.cx.internal.cloudapp.net@SSIS.COM
renew until 06/14/18 00:16:24
```

## On Windows client (siduOracle1)

Install Oracle client 12.

Create `c:\kerberos\krb5.conf`, identical as on the server except for the port numbers:

```
[libdefaults]
 default_realm=SSIS.COM
 
[realms]
 SSIS.COM= {
  kdc=siduOracle0.ssis.com
  admin_server=siduOracle0.ssis.com
 }
 
[domain_realm]
 ssis.com=SSIS.COM
 .ssis.com=SSIS.COM
```

Adjust `sqlnet.ora`：

```
SQLNET.AUTHENTICATION_SERVICES= (BEQ,KERBEROS5)
 
NAMES.DIRECTORY_PATH= (TNSNAMES, EZCONNECT)
 
SQLNET.KERBEROS5_CONF =c:\kerberos\krb5.conf
SQLNET.KERBEROS5_CONF_MIT = true
SQLNET.KERBEROS5_CC_NAME=MSLSA:
```

**IMPORTANT:**

- For 11.x clients authentication service KERBEROS5 is used, with Credential Cache (CC_NAME) **OSMSFT:**
- For 12.x client 12.x in theory, KERBROS5 service should be used with **MSLSA:** for the CC_NAME, ~~however due to bug 18895651, KERBEROS5PRE is required with CC_NAME OSMSFT:~~ (seems fixed)
 
Edit `C:\Windows\System32\drivers\etc\services`:

```
kerberos           88/tcp    kerberos5 krb5 kerberos-sec      #Kerberos
kerberos           88/udp    kerberos5 krb5 kerberos-sec      #Kerberos
```

### Validate:

Login to siduOracle1 with `ssis\test`;

```
C:\Users\sizhong.SSIS>sqlplus /@mainorcl
 
SQL*Plus: Release 12.2.0.1.0 Production on Wed Jun 13 09:04:12 2018
 
Copyright (c) 1982, 2017, Oracle.  All rights reserved.
 
Last Successful login time: Wed Jun 13 2018 07:55:40 +00:00
 
Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
 
SQL>
```
