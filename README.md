# Build-CentOS5-image-for-GoolgeCloud

This project is to build CentOS 5 image by kickstart on KVM for Google Cloud(gce).

1. setup a build host with KVM and http server, install google-cloud-sdk from
 https://cloud.google.com/sdk/docs/quickstart-linux

2. setup a local respository on the build host, copy those dependents packages to the http server directory and run the following command to setup local respository

>sudo mkdir /var/www/html/centos5-ks

>sudo cp ickstart-cfg/* /var/www/html/centos5-ks

>sudo mkdir /var/www/html/centos5-cloud

>sudo cp respository/* /var/www/html/centos5-cloud

>cd /var/www/html/centos5-cloud

>sudo createrepo -s sha1 .

3. install a VM by kickstart as disk.raw

>sudo qemu-img create -f raw /var/lib/libvirt/images/disk.raw 10G

>sudo virt-install --name gce-centos-5.11-core \
     --ram 1024 --vcpus=2 --os-variant=rhel5 \
     --location=http://mirror.optus.net/centos/5/os/x86_64/  \
     --extra-args="ks=http://localhost/centos5-ks/gce-core-ks5.cfg" \
     --disk path=/var/lib/libvirt/images/disk.raw,size=10,format=raw,bus=virtio \
     --bridge=br0,model=virtio 

4. Compress the disk file
>tar -Szcf disk.raw gce-centos-5.11-core.tar.gz

5. Upload the compressed disk file to google cloud storage
>gsutil cp gce-centos-5.11-core.tar.gz gs://mygce-bucket

6. Create gce image
>gcloud compute images create centos-511-core-x86  --source-uri gs://mygce-bucket/gce-centos-5.11-core.tar.gz --description "CentOS 5.11 core x86_64"

7. lauch instance
>gcloud compute instances create centos-511-test --image centos-511-core-x86 --zone asia-east1-a --tags http-server
