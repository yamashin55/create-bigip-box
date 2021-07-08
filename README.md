# create-bigip-box

1. download ova file from download.f5.com
2. create box
  '''
  OVA_FILE=BIGIP-16.1.0-0.0.19.ALL_1SLOT-vmware.ova
  BOX_FILE=BIGIP-16.1.0.box
  VBoxManage import ${OVA_FILE} --vsys 0 --memory 2048 --cpus 2 --eula accept
  VBoxManage list vms
  VM_NAME=vm
  vagrant package --base ${VM_NAME} --output ${BOX_FILE}
  VBoxManage unregistervm  vm --delete
  VBoxManage list vms
  vagrant box add BIGIP-16.1.0.box --name BIGIP-16.1.0
  vagrant init -m bigip
  '''
