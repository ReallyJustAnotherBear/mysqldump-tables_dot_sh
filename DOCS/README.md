#!/bin/sh

##MANDATORY MYSQL CLIENT CONNECTION USER CONFIG
##example  Using a mythtv servers db 'mythconverg', with host ip = 192.168.1.162

USER=mythtv
PASSWORD=mythtv
HOST="192.168.1.162"
DATABASE=mythconverg
##USER CONFIG END


##MYSQL BINARIES USER CONFIG IF MYSQL NOT IN STANDARD *NIX PREFIX /usr
MYSQL=/usr/bin/mysql
MYSQLDUMP=/usr/bin/mysqldump
MYSQLCHECK=/usr/bin/mysqlcheck

##logging
#LOGS=true
LOGS=false

#IFS=" "

ALLTABLES=$(for i in $($MYSQL -h$HOST -u$USER -p$PASSWORD $DATABASE -e 'show tables';);do echo $i;done)

#remove first nontable in list
#XTABLES will be the new modified list
XTABLES=$(echo ${ALLTABLES##Tables_in_mythconverg})

#check 
$MYSQLCHECK -v -h$HOST -u$USER -p$PASSWORD $DATABASE &>/dev/null
MCHKRVAL=$?
if [ $MCHKRVAL -ne 0 ];then 
	echo "db error on check, exiting"
	exit 1
fi

#full dump first, if none
#if [ ! -e $DATABASE ];then
#	$MYSQLDUMP -v -h$HOST -u$USER -p$PASSWORD $DATABASE -r ./$DATABASE.sql
#fi

#mkdir if not exist
if [ ! -e ALLTABLES_OUT ];then
	mkdir -p ALLTABLES_OUT
fi

if [ $LOGS = 'true' ] && [ ! -e $LOGS ] ;then
	mkdir -p LOGS
fi

cd ALLTABLES_OUT

if [ $LOGS = 'true' ];then
	for i in $XTABLES;do 
		$MYSQLDUMP -v -h$HOST -u$USER -p$PASSWORD $DATABASE $i -r $i.sql 2>>../LOGS/$DATABASE.sql.dumptable.log 
	done
else
	for i in $XTABLES;do 
		$MYSQLDUMP -v -h$HOST -u$USER -p$PASSWORD $DATABASE $i -r $i.sql 
	done
fi

cd ../

echo "$(ls ./ALLTABLES_OUT/*.sql|wc -l) tables dumped"

echo "done!"

