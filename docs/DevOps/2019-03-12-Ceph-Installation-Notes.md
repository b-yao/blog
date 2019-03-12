# Ceph Installation Notes

Ceph is a storage cluster and can be install in many ways. Using ceph deploy is suggested for new users.

By following the [docs][1], there are the notes to be kept in mind:
- The documentation link is different fo each version:
  * Newest: http://docs.ceph.com/docs/master/start/
  * Version Mimic: http://docs.ceph.com/docs/mimic/start/
- *Don't use the ceph user* to do deploy since this will conflict with the Ceph daemon
- Better configure all the hostname as it is in `/etc/hosts`, otherwise ceph deploy will complain
- The configuration is pushed from the admin node to others by the following command
  ```
  ceph-deploy --overwrite-conf config push {node}
  ```
- Using ceph admin command in nodes mostly only available as sudoer since the key is not openly available:
  ```
  /etc/ceph/ceph.client.admin.keyring
  ```
- Useful commands:
  ```
  ceph osd pool create rdb 128 # 128 is the pg_num
  rbd pool init rdb
  rbd create cephbdi --size 4096 --image-feature layering # Size in MB
  rbd map cephbd --name client.admin
  mkfs.ext4 -m0 /dev/rbd/rbd/cephbd
  mkdir /mnt/ceph-block-device
  mount /dev/rbd/rbd/cephbd /mnt/ceph-block-device
  cd /mnt/ceph-block-device
  # Create ceph Filesystem
  ceph osd pool create cephfs_data 28
  ceph osd pool create cephfs_metadata 28
  ceph fs new seffs cephfs_metadata cephfs_data

  # Set up a ceph rgw in node1
  # In admin node:
  ceph-deploy install --rgw node1
  tee -a ceph.conf << EOF
  [client.rgw.node1]
  rgw_frontends = "civetweb port=80"
  EOF
  ceph-deploy --overwrite-conf config push node1
  # In node1
  sudo systemctl restart ceph-radosgw@rgw.node1
  # In admin node? creat an API user
  sudo radosgw-admin user create --uid="testuser" --display-name="First User"
  # Anywhere:
  aws --endpoint-url {rgw url} --no-verify-ssl s3api list-buckets
  ```

[1]: http://docs.ceph.com/docs/master/start/
