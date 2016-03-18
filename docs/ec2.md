#### Deploy on Amazon EC2
```bash
#!/bin/bash

case $1 in

  "masters")

    ETCD_TOKEN=$(curl -s https://discovery.etcd.io/new?size=3 | awk -F '/' '{print $NF}')

    for i in 1 2 3; do

      coreseed udata \
      --hostname core-${i} \
      --domain cell-1.dc-1.demo.com \
      --role master \
      --ns1-api-key xxx \
      --ca-cert path/to/cert.pem \
      --etcd-token ${ETCD_TOKEN} |

      gzip --best | coreseed run-ec2 \
      --region eu-west-1 \
      --image-id ami-95bb00e6 \
      --instance-type t2.medium \
      --key-pair xxx \
      --vpc-id vpc-xxx \
      --subnet-id subnet-xxx \
      --sec-group-ids sg-xxx,sg-xxx

    done ;;

  "nodes")

    for i in 4 5 6; do

      coreseed udata \
      --hostname core-${i} \
      --domain cell-1.dc-1.demo.com \
      --role node \
      --ns1-api-key xxx \
      --ca-cert path/to/cert.pem \
      --flannel-network 10.128.0.0/21 \
      --flannel-subnet-len 27 \
      --flannel-subnet-min 10.128.0.192 \
      --flannel-subnet-max 10.128.7.224 \
      --flannel-backend vxlan |

      gzip --best | coreseed run-ec2 \
      --region eu-west-1 \
      --image-id ami-95bb00e6 \
      --instance-type t2.medium \
      --key-pair xxx \
      --vpc-id vpc-xxx \
      --subnet-id subnet-xxx \
      --sec-group-ids sg-xxx,sg-xxx

    done ;;

esac
```