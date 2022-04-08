# magma-agw-docker-setup

Rename your network interfaces to `eth0` and `eth1`

Become root:
```bash
sudo su
cd /root
```

Copy `rootCA.pem` file in the desired location:
```bash
mkdir -p /var/opt/magma/certs
vim /var/opt/magma/certs/rootCA.pem
```

Verify `rootCA.pem` file:
```bash
openssl x509 -text -noout -in /var/opt/magma/certs/rootCA.pem
```

Download docker install script:
```bash
wget https://github.com/magma/magma/raw/master/lte/gateway/deploy/agw_install_docker.sh
```

Add the following command in `agw_install_docker.sh` file before `ansible-playbook` command:
```bash
sed -i 's/focal-1.7.0/focal-1.6.0/' /opt/magma/lte/gateway/deploy/roles/magma_deploy/vars/all.yaml
```

Install AGW:
```bash
bash agw_install_docker.sh
```

Add docker registry:
```bash
sed -i 's,public.ecr.aws/z2g3r6f7,magmacore,' /var/opt/magma/docker/.env
sed -i 's/latest/1.7.0/' /var/opt/magma/docker/.env
```

Revert chnages if needed:
```bash
sed -i 's,magmacore,public.ecr.aws/z2g3r6f7,' /var/opt/magma/docker/.env
sed -i 's/1.7.0/latest/' /var/opt/magma/docker/.env
```

Start AGW:
```bash
cd /var/opt/magma/docker
docker-compose up -d
```

Get Hardware ID and Challenge key and add AGW in your Orc8r:
```bash
docker exec magmad show_gateway_info.py
```

Stop AGW services:
```bash
docker-compose down
```

Update `control_proxy.yml` file:
```bash
cat << EOF > /var/opt/magma/configs/control_proxy.yml
cloud_address: controller.orc8r.magmacore.link
cloud_port: 443
bootstrap_address: bootstrapper-controller.orc8r.magmacore.link
bootstrap_port: 443
fluentd_address: fluentd.orc8r.magmacore.link
fluentd_port: 24224

rootca_cert: /var/opt/magma/certs/rootCA.pem
EOF
```

Start AGW again:
```bash
docker-compose up -d
```

Check if you got `gateway.crt` and `gateway.key` files after bootstrap:
```bash
ls /var/opt/magma/certs/
```

Check if you get `INFO:root:Bootstrapped Successfully!` message in the logs:
```bash
docker logs magmad -f
```

Stop AGW services again:
```bash
docker-compose down
```

Start AGW again:
```bash
docker-compose up -d
```

Check if you get `INFO:root:Checkin Successful!` message in the logs:
```bash
docker logs magmad
```

Check if checkin is successful: 
```bash
docker exec magmad checkin_cli.py
```

List all subscribers:
```bash
docker exec subscriberdb subscriber_cli.py list
```
---

Notes:
- delete `attach_reject` section in **/etc/magma/eventd.yml**
- delete `ipv6_solicitation` from `static_services` in **/etc/magma/pipelined.yml**

