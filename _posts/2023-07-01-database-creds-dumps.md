---
title: "Database Credential Dumps"
date: 2023-07-01T15:34:30-04:00
categories:
  - blog
tags:
  - Databases
  - Hashcat
---

The following is a compilation of queries to list database user hashes, which are stored locally in tables. The hashes obtained can be cracked using password cracking tools like Hashcat.

### MySQL :
#### mysql_native_password :
```
SELECT * FROM mysql.user;
```

Hashcat Mode : `300` 

#### caching_sha2_password :
```
SELECT CONCAT('\$mysql',LEFT(authentication_string,6),'*',INSERT(HEX(SUBSTR(authentication_string,8)),41,0,'*')) AS hash FROM mysql.user WHERE plugin = 'caching_sha2_password' AND authentication_string NOT LIKE '%INVALIDSALTANDPASSWORD%' AND authentication_string !='';
```
Hashcat Mode : `7401`


### Postgres

```
SELECT * FROM pg_shadow;
SELECT username,passwd FROM pg_roles;
```

Hash Prep :
* Strip `md5` from the prefix.
* Add the username to the hash file : `01dfae6e5d4d90d9892622325959afbe:USERNAME`

Hashcat Mode : `10`


### Cassandra 
```
select * FROM system_auth.roles;
```
Hashcat Mode : `1800`


### SQL server 2014
`SELECT name,password_hash FROM sys.sql_logins;`

### MongoDB Shell :
```
db.system.users.find().pretty()
db.getUsers({showCredentials: true}).users[0]
```


###  VerticaDB :
```
select *  from v_catalog.passwords;
```

Hash Prep :
* Strip `md5` from the prefix.
* Add the username to the hash file : `01dfae6e5d4d90d9892622325959afbe:USERNAME`

Hashcat Mode : `10`
