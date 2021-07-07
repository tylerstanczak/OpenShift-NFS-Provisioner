sudo yum -y update
sudo hostnamectl set-hostname server.example.com --static

sudo yum -y install nfs-utils

sudo systemctl enable --now nfs-server rpcbind

lsblk  | grep sdb

sudo parted -s -a optimal -- /dev/sdb mklabel gpt
sudo parted -s -a optimal -- /dev/sdb mkpart primary 0% 100%
sudo parted -s -- /dev/sdb align-check optimal 1
sudo mkfs.xfs /dev/sdb1

sudo mkdir /nfs
echo "/dev/sdb1 /nfs xfs defaults 0 0" | sudo tee -a /etc/fstab
sudo mount -a

df -hT | grep /nfs

echo "/nfs	*(rw,no_subtree_check,sync,no_wdelay,insecure,no_root_squash)" >> /etc/exports

sudo exportfs -rav

oc login

oc new-project nfs

oc adm policy add-scc-to-user privileged -z nfs-pod-provisioner-sa

edit the provisioner.yaml deployment file

oc apply -f .

oc patch storageclass nfs -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'

oc patch configs.imageregistry.operator.openshift.io cluster -p '{"spec":{"managementState":"Managed","storage":{"pvc":{"claim":"image-registry-storage"}}}}' --type=merge
