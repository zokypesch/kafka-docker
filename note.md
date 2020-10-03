# installation
sudo apt-get update

apt-get install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"

apt-get update
apt-get install docker-ce

# install git
sudo apt install git


# docker compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

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
umount /dev/vdb

# clone this repo
git clone https://github.com/zokypesch/kafka-docker

# running
cd kafka-docker
edit file docker-compose-single-broker.yml -> change kafka_advertise_host to IP address
docker-compose -f docker-compose-single-broker.yml up -d --build

# setting
docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1001 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_trx_server -e BOOTSTRAP_SERVERS=172.28.14.199:9092 -d debezium/connect:latest

docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1002 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_general_server -e BOOTSTRAP_SERVERS=172.28.14.200:9092 -d debezium/connect:latest

docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1003 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_batch_server -e BOOTSTRAP_SERVERS=172.28.14.201:9092 -d debezium/connect:latest

# running for trx_server
curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/ -d \
'{"name":"transaction","config":{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9j026wg1l28z60gp66640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"db_admin","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_trx","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"transaction","database.history.kafka.bootstrap.servers":"172.28.14.199:9092","database.history.kafka.topic":"dbhistory.transaction","snapshot.mode": "when_needed", "gtid.new.channel.position": "latest", "table.whitelist": "transaction.trans,transaction.trans_reconcile,transaction.user_active_course,transaction.user_bank_va,transaction.purchase_invoice,transaction.certificate,transaction.credit_user,transaction.incentive"}}'

<!-- "snapshot.mode": "when_needed", "gtid.new.channel.position": "latest" -->

curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/transaction/tasks/0/restart

mysql-bin.001007	
mysql-bin.001008	
mysql-bin.001009	
mysql-bin.001010	
mysql-bin.001011	

curl -i -X PUT \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/transaction/config -d \
'{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9j026wg1l28z60gp66640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"db_admin","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_trx","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"transaction","database.history.kafka.bootstrap.servers":"172.28.14.199:9092","database.history.kafka.topic":"dbhistory.transaction","snapshot.mode": "initial", "table.whitelist": "transaction.trans,transaction.trans_reconcile,transaction.user_active_course,transaction.user_bank_va,transaction.purchase_invoice,transaction.certificate,transaction.credit_user,transaction.incentive,transaction.transfer"}}'

# running for users
curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/ -d \
'{"name":"users","config":{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9j026wg1l28z60gp66640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"db_admin","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_users","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"users","database.history.kafka.bootstrap.servers":"172.28.14.199:9092","database.history.kafka.topic":"dbhistory.users","snapshot.mode": "when_needed", "gtid.new.channel.position": "latest", "table.whitelist": "user_details,users,onboarding_history,user_additionals"}}'

curl -i -X PUT \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/users/config -d \
'{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9j026wg1l28z60gp66640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"db_admin","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_users","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"users","database.history.kafka.bootstrap.servers":"172.28.14.199:9092","database.history.kafka.topic":"dbhistory.users","snapshot.mode": "when_needed", "gtid.new.channel.position": "latest", "table.whitelist": "users.user_details,users.users,users.user_additionals"}'


curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/users/tasks/0/restart

curl -i -X GET \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/users/tasks/0/status

docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=trans' -e 'MAX_ROUTINE=20' --name trans  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=trans_reconcile' -e 'MAX_ROUTINE=20' --name trans_reconcile  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=user_active_course' -e 'MAX_ROUTINE=20' --name user_active_course  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=user_bank_va' -e 'MAX_ROUTINE=20' --name user_bank_va  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=purchase_invoice' -e 'MAX_ROUTINE=20' --name purchase_invoice  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=certificate' -e 'MAX_ROUTINE=20' --name certificate  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=credit_user' -e 'MAX_ROUTINE=20' --name credit_user  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=incentive' -e 'MAX_ROUTINE=20' --name incentive  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=transfer' -e 'MAX_ROUTINE=20' --name transfer  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=cron' -e 'CODE=transfer' -e 'MAX_ROUTINE=20' -e 'CRON_LISTEN=trans,trans_reconcile,user_active_course,user_bank_va,purchase_invoice,certificate,credit_user,incentive,transfer' --name cron_all  -d zokypesch/data_mesh:latest

docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=survey-incentive' -e 'MAX_ROUTINE=20' --name survey_incentive  -d zokypesch/data_mesh:latest

docker stop trans && docker rm trans && \
docker stop trans_reconcile && docker rm trans_reconcile && \
docker stop user_active_course && docker rm user_active_course && \
docker stop user_bank_va && docker rm user_bank_va && \
docker stop purchase_invoice && docker rm purchase_invoice && \
docker stop certificate && docker rm certificate && \
docker stop credit_user && docker rm credit_user && \
docker stop incentive && docker incentive incentive

docker run  -dit --restart unless-stopped  -e 'ROLE=cron' -e 'CODE=transfer' -e 'MAX_ROUTINE=20' --name transfer  -d zokypesch/data_mesh:latest

Error 1064: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near is = 8522163) at line 1

,,onboarding_history,user_additionals

docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=user_details' -e 'MAX_ROUTINE=20' --name user_details  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=users' -e 'MAX_ROUTINE=20' --name users  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=user_additionals' -e 'MAX_ROUTINE=20' --name user_additionals  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=cron' -e 'CODE=user_details' -e 'MAX_ROUTINE=20' -e 'CRON_LISTEN=user_details,users,user_additionals' --name cron_users_all  -d zokypesch/data_mesh:latest

# batch
curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/ -d \
'{"name":"batch","config":{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9js218880l8oh31a66640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"batch_svc","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_batch","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"batch","database.history.kafka.bootstrap.servers":"172.28.14.201:9092","database.history.kafka.topic":"dbhistory.batch", "table.whitelist": "batch.pool,batch.join_pool,batch.result"}}'

# running worker
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=pool' -e 'MAX_ROUTINE=20' --name pool  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=join_pool' -e 'MAX_ROUTINE=20' --name join_pool  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=result' -e 'MAX_ROUTINE=20' --name result  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=cron' -e 'CODE=pool' -e 'MAX_ROUTINE=20' -e 'CRON_LISTEN=pool,join_pool,result' --name cron_all  -d zokypesch/data_mesh:latest

# batch
curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/ -d \
'{"name":"batch","config":{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9js218880l8oh31a66640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"batch_svc","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_batch","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"batch","database.history.kafka.bootstrap.servers":"172.28.14.201:9092","database.history.kafka.topic":"dbhistory.batch", "table.whitelist": "batch.pool,batch.join_pool,batch.result"}}'

# cluster general

# platform-api
curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
localhost:8083/connectors/ -d \
'{"name":"pmo-platform","config":{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"100","database.hostname":"rm-d9j99s2po8fycw3t366640.mysql.ap-southeast-5.rds.aliyuncs.com","database.port":"3306","database.user":"pmo_user","database.password":"Pr4K3rj4S3laM4ny4","database.server.name":"dbserver_general","max.batch.size":16384,"max.queue.size":131072,"offset.flush.timeout.ms":60000,"offset.flush.interval.ms ":15000,"database.whitelist":"pmo-platform,wallet_prakerja,survey_tests","database.history.kafka.bootstrap.servers":"172.28.14.200:9092","database.history.kafka.topic":"dbhistory.general","snapshot.mode":"when_needed","table.whitelist":"survey_tests.user_agreement,survey_tests.user_agreement_questions,pmo-platform.companies,pmo-platform.courses,wallet_prakerja.wallet,wallet_prakerja.user_wallet,wallet_prakerja.user_wallet_history"}}'

curl -i -X GET -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/pmo-platform/tasks/0/status

# cluster general pmo platform
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=companies' -e 'MAX_ROUTINE=20' --name companies  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=courses' -e 'MAX_ROUTINE=20' --name courses  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=wallet' -e 'MAX_ROUTINE=20' --name wallet  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=user_wallet' -e 'MAX_ROUTINE=20' --name user_wallet  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=user_wallet_history' -e 'MAX_ROUTINE=20' --name user_wallet_history  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=user_agreement' -e 'MAX_ROUTINE=20' --name user_agreement  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=data_mesh' -e 'CODE=user_agreement_questions' -e 'MAX_ROUTINE=20' --name user_agreement_questions  -d zokypesch/data_mesh:latest && \
docker run  -dit --restart unless-stopped  -e 'ROLE=cron' -e 'CODE=companies' -e 'MAX_ROUTINE=20' -e 'CRON_LISTEN=companies,courses,wallet,user_wallet,user_wallet_history,user_agreement,user_agreement_questions' --name cron_all  -d zokypesch/data_mesh:latest