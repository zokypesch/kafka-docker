# check storage
lsblk

# check yang sudah di mount
blkid

# format
mkfs.ext4 /dev/vdb

# mounting
mkdir -p /kafka/zookeper-data/data
mkdir -p /kafka/zookeper-data/logs
mkdir -p /kafka/kafka-data
mount /dev/vdb /kafka (ini tergantung dari lsblk nya)

# running
docker-compose -f docker-compose-single-broker.yml up -d

# setting
docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1001 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_trx_server -e BOOTSTRAP_SERVERS=172.28.14.196:9092 -d debezium/connect:latest

docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1002 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_general_server -e BOOTSTRAP_SERVERS=172.28.14.197:9092 -d debezium/connect:latest

docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1003 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_batch_server -e BOOTSTRAP_SERVERS=172.28.14.198:9092 -d debezium/connect:latest

# running for trx_server
curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/ -d \
'{"name":"transaction","config":{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9j026wg1l28z60gp66640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"db_admin","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_trx","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"transaction","database.history.kafka.bootstrap.servers":"172.28.14.196:9092","database.history.kafka.topic":"dbhistory.transaction"}}'

curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/transaction/tasks/0/restart

curl -i -X PUT \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/transaction/config -d \
'{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9j026wg1l28z60gp66640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"db_admin","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_trx","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"transaction","database.history.kafka.bootstrap.servers":"172.28.14.196:9092","database.history.kafka.topic":"dbhistory.transaction"}'