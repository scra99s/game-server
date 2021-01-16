# DOCKER
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf install docker-ce --nobest -y
systemctl enable --now docker
usermod -aG docker scrapps

# DOCKER-COMPOSE
curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Docker Nodes
pvcreate /dev/sdb
vgcreate vg0 /dev/sdb
lvcreate -n dockerstorage -l 100%FREE vg0
mkfs -t ext4 /dev/vg0/dockerstorage
e2label /dev/vg0/docker-storage dockerstorage
echo "/dev/mapper/vg0-dockerstorage /var/lib/docker ext4 defaults 0 0" >> /etc/fstab
mount -a

# Storage Node
pvcreate /dev/sdc
vgcreate vg0 /dev/sdc
lvcreate -n nfsstorage -l 100%FREE vg1
mkfs -t ext4 /dev/vg1/nfsstorage
e2label /dev/vg1/nfsstorage nfsstorage
echo "/dev/mapper/vg1-nfsstorage /var/lib/docker ext4 defaults 0 0" >> /etc/fstab
mount -a

docker node update --label-add node=storage swarm-storage.jerra.io
docker node update --label-add node=general swarm-node-1.jerra.io
docker node update --label-add node=general swarm-node-2.jerra.io
docker node update --label-add node=general swarm-node-3.jerra.io
