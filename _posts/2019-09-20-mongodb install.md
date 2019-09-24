# 

##for each server check edit hosts 
vi /etc/hosts
IP1    uidmongo1
IP2    uidmongo2
IP3    uidmongo3

##OS para config
ulimit -f unlimited  
ulimit -t unlimited  
ulimit -v unlimited  
ulimit -n 64000  
ulimit -m unlimited  
ulimit -u 64000  


##create mongo dir
mkdir -p /DATA1/mongodata
mkdir -p /DATA1/mongodata/logs
mkdir -p /DATA1/mongodata/mongos
mkdir -p /DATA1/mongodata/config
mkdir -p /DATA1/mongodata/shard1
mkdir -p /DATA1/mongodata/shard2
mkdir -p /DATA1/mongodata/shard3

##upload source file to /usr/local

tar -zxvf mongodb-linux-x86_64-rhel62-3.4.4.tgz                    

mv  mongodb-linux-x86_64-rhel62-3.4.4/ /usr/local/mongodb  

export PATH=/usr/local/mongodb/bin:$PATH
#####add path 
vi ~/.bashrc
export PATH=/usr/local/mongodb/bin:$PATH


#######copy *.conf file to each server
scp config.conf mongos.conf shard1.conf shard2.conf shard3.conf root@IP3:/usr/local/mongodb/bin

#######start shardsrv for each server of each shard
mongod -f /usr/local/mongodb/bin/shard1.conf
mongod -f /usr/local/mongodb/bin/shard2.conf
mongod -f /usr/local/mongodb/bin/shard3.conf

######start config server for 3 servers
mongod -f /usr/local/mongodb/bin/config.conf

###### start replicaset for config server
mongo uidmongo1:27019
rs.initiate({_id:"configRS",configsvr:true,members:[{_id:0,host:"uidmongo1:27019"},{_id:1,host:"uidmongo2:27019"},{_id:2,host:"uidmongo3:27019"}]})

######set uidmongo1 as primary node
cfg = rs.conf()
cfg.members[0].priority = 2
rs.reconfig(cfg, { force: true})

##### start mongos for each server
mongos -f mongos.conf

#####init replicaset
##for shard1's primary node uidmongo1
mongo uidmongo1:22001/admin
config = {_id:"shard1", members:[{_id:0,host:"uidmongo1:22001",priority:2},{_id:1,host:"uidmongo2:22001",priority:1},{_id:2,host:"uidmongo3:22001",priority:1}]}
rs.initiate(config)
##for shard2's primary node uidmongo2
mongo uidmongo2:22002/admin
config = {_id:"shard2", members:[{_id:0,host:"uidmongo1:22002",priority:1},{_id:1,host:"uidmongo2:22002",priority:2},{_id:2,host:"uidmongo3:22002",priority:1}]}
rs.initiate(config)
##for shard3's primary node uidmongo3
mongo uidmongo3:22003/admin
config = {_id:"shard3", members:[{_id:0,host:"uidmongo1:22003",priority:1},{_id:1,host:"uidmongo2:22003",priority:1},{_id:2,host:"uidmongo3:22003",priority:2}]}
rs.initiate(config)


#####start shard cluster
mongo uidmongo1:27017/admin
db.runCommand( { addshard : "shard1/uidmongo1:22001,uidmongo2:22001,uidmongo3:22001"});
db.runCommand( { addshard : "shard2/uidmongo1:22002,uidmongo2:22002,uidmongo3:22002"});
db.runCommand( { addshard : "shard3/uidmongo1:22003,uidmongo2:22003,uidmongo3:22003"});

####check shard
db.runCommand({listshards : 1 });


####check install status
mongo uidmongo1:27017/admin
use test
sh.enableSharding("test")
db.users.createIndex( { username: 1 })
###config shard Key£º
sh.shardCollection("test.users",{_id: "hashed" })

 use config
 db.settings.save( { _id:"chunksize", value: 1})
####insert 50000 rows
use test
for (var i=0; i<50000; i++) {db.users.insert({"username" : "user"+i, "create_at": new Date()})}
###check status
sh.status()

####reconfig config server
 use config
 db.settings.save( { _id:"chunksie", value: 64})

##################security config
####create administrator user for mongo
use admin
db.createUser({ user: "admin" , pwd: "Mgo1!dbT", roles: ["root"]})

####db.createUser({ user: "admin" , pwd: "Mgo1!dbT", roles: ["userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase"]}

###gen key for mongo
openssl rand -base64 741 > /DATA1/mongodata/keyfile
###copy to others nodes
scp keyfile root@IP3:/DATA1/mongodata/

###add properties for config.conf and mongos.conf
#for mongos.conf
security:
 keyFile: /DATA1/mongodata/keyfile
#for config.conf
security:
 authorization: enabled
 keyFile: /DATA1/mongodata/keyfile
##for shard*.conf
security:
 authorization: enabled
 keyFile: /DATA1/mongodata/keyfile
chmod 600 /DATA1/mongodata/keyfile

####check test login with super user
mongo "mongodb://IP1:27017/admin" -username admin --password 'Mgo1!dbT'

use admin
db.createUser({ user: "chengl" , pwd: "MgocTdbT", roles: ["userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase"]})


mongo "mongodb://IP1:27017/admin" -username chengl --password 'MgocTdbT'


db.grantRolesToUser("chengl", [ { role: "userAdminAnyDatabase", db: "admin" } ])
db.grantRolesToUser("chengl", [ { role: "root", db: "admin" } ])
db.grantRolesToUser("chengl", [ { role: "dbAdminAnyDatabase", db: "admin" } ])


mongo "mongodb://IP1:27017/admin" -username chengl --password 'MgocTdbT'


db.runCommand( { enablesharding :"wp_ibitauto"});






###########create another user for songk admin role
mongo "mongodb://IP1:27017/admin" -username admin --password 'Mgo1!dbT'


use admin
db.createUser({ user: "songk" , pwd: "SKoc1TdbT", roles: ["userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase"]})
db.grantRolesToUser("songk", [ { role: "root", db: "admin" } ])







################monitor

/usr/local/mongodb/bin/mongostat --username=admin --password='Mgo1!dbT' --authenticationDatabase=admin


/usr/local/mongodb/bin/mongostat --host=IP1:27017 --username=admin --password='Mgo1!dbT' --authenticationDatabase=admin




/usr/local/mongodb/bin/mongostat --host=IP1:27019 --username=admin --password='Mgo1!dbT' --authenticationDatabase=admin
