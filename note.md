# check storage
lsblk

# check yang sudah di mount
blkid

# format
mkfs.ext4 /dev/vdb

# mounting
mkdir -f /kafka/zookeper-data/data
mkdir -f /kafka/zookeper-data/logs
mkdir -f /kafka/kafka-data
mount dev/vdb /kafka (ini tergantung dari lsblk nya)

# cr