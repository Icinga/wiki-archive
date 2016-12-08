#Clean removed instances from IDOUtils DB

##Why?
If you got more than one instance, and removed one from your database, that data will remain in there, seperated by instance_id. There is currently no automated way of removing everything as this is **very destructive**.

##How?
The relation in the schema works like the following, top down:

* instances
	* instance_name
	* instance_id (Oracle: id)
* hostchecks
	* instance_id

etc.

###instance name
Get into the idomod.cfg of the instance you want to remove.

```
# cat /etc/icinga/idomod.cfg | grep instance_name
instance_name=foo-instance
```

###tables

default example with dbname = "icinga", table_prefix = "icinga_", we don't wanna check the dbversion as well as it does not contain an instance_id.

**save the list for the delete call later on.**

#### MySQL
	echo show tables | mysql icinga | grep -v dbversion  | grep icinga_ > /tmp/tables
#### Postgresql
```
# su - postgres
$ echo "\dt" | psql icinga | sed 's/.*\(icinga_[a-z]*\).*/\1/' | grep -v dbversion | grep icinga > /tmp/tables
```
#### Oracle
output table format is expected uppercase.

```
$ echo "select table_name from user_tables;" | sqlplus -s icinga/icinga@ICINGADB | grep -v DBVERSION | grep '^[A-Z][A-Z]' > /tmp/tables
```

### instance id

Get the **active** instance_id and note the value for later.

#### MySQL
```
# echo "select instance_id from icinga_instances where instance_name='foo-instance'" | mysql icinga
```

#### Postgresql
```
# su - postgres
$ echo "select instance_id from icinga_instances where instance_name='foo-instance'" | psql icinga
```
#### Oracle
**Note that the table prefix is removed, as well as it's instances.id as primary key!**
	
	$ echo "select id from instances where instance_name='foo-instance'; | sqlplus -s icinga/icinga@ICINGADB
### clean all tables by instance id
**Create a backup before deleting data!!**

In this example we got instance_id!=**1** - change that accordingly before running this 

We will delete **all other than 1** instance_ids!

#### MySQL
```
# for t in `cat /tmp/tables` ; do echo "delete from $t where instance_id!=1;" | mysql icinga ; done
```
#### Postgresql
```
# su - postgres
$ for t in `cat /tmp/tables` ; do echo "delete from $t where instance_id!=1;" | psql icinga ; done
```
#### Oracle
```
# for t in `cat /tmp/tables` ; do echo "delete from $t where instance_id!=1;" | sqlplus -s icinga/icinga@ICINGADB ; echo "commit;" | sqlplus -s icinga/icinga@ICINGADB ; done
```
### clean up
```
# rm /tmp/tables
```
