# INSTALL ICP CE or EE

* [Prepare](#prepare)
* [SSH](#ssh)
* [Install CE](#ce)
* [Install EE](#ee)
* [Config](#config)
* [Config Kubectl](#kubectl)
* [K8S Dashboard](#dash)
* [HELM](#helm)
* [Image preload](#image)
* [LDAP](#ldap)

## <a name="prepare"></a>Prepare
### Update / Upgrade
```bash
sudo apt-get update
sudo apt-get upgrade
```

### INSTALL SOME STUFF
```bash
sudo apt-get install tree
sudo apt-get install htop
sudo apt install curl
sudo apt-get install unzip
sudo apt-get install nethogs
sudo nethogs ens33
sudo apt-get install bmon
sudo apt-get install iftop
```


### KEYBOARD
```bash
sudo dpkg-reconfigure keyboard-configuration
```

### NETWORK CONFIG
```bash
ifconfig
sudo nano /etc/network/interfaces
```

--> Modify with this:

```bash
# interfaces(5) file used by ifup(8) and ifdown(8)
# auto lo
# iface lo inet loopback
auto ens33
iface ens33 inet static
address 192.168.232.127
netmask 255.255.255.0
gateway 192.168.232.2
dns-nameservers 192.168.232.2
```

### Modify Host Name
```bash
sudo nano /etc/hosts

sudo hostname master.icp
sudo hostname
sudo nano /etc/hostname
sudo hostname master.icp
```



### Modify some performance stuff
```bash
sudo sysctl -w vm.max_map_count=262144
sudo nano /etc/sysctl.conf
```

Add the line

```bash
vm.max_map_count=262144
```



## <a name="ssh">Install and configure SSH
```bash
sudo apt-get install openssh-server
sudo apt-get install ssh
sudo apt-get install openssh-server

sudo nano /etc/ssh/sshd_config
Change to -->
PermitRootLogin yes

sudo systemctl enable ssh
sudo service ssh restart
sudo su -
passwd
sudo mkdir -p /root/.ssh
sudo reboot
```


### Create SSH Key
```bash
ssh-keygen -b 4096 -t rsa -f ~/.ssh/master.id_rsa -N ""
cat ~/.ssh/master.id_rsa.pub | sudo tee -a ~/.ssh/authorized_keys
ssh-copy-id -i ~/.ssh/master.id_rsa.pub root@192.168.232.127
```

```bash
ssh root@192.168.232.127
cat ~/.ssh/master.id_rsa.pub | sudo tee -a ~/.ssh/authorized_keys
sudo systemctl restart sshd
```

```bash
sudo cp ~/.ssh/master.id_rsa ~/cluster/ssh_key
sudo chmod 400 ~/cluster/ssh_key

```



## Install Docker
```bash
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64]
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce
sudo systemctl enable docker
sudo usermod -aG docker demo
```



## Install Python
```bash
sudo apt install python
sudo apt install python-pip
#sudo pip install docker-py    --> don't install this
```


## Install K8s
### Install Snap
```bash
sudo apt-get install snapd
```



### Install KUBECTL
```bash
sudo snap install kubectl --classic
```



### Install HELM
```bash
sudo apt-get install socat

#sudo kill 63454 | sudo snap remove helm
```


```bash
docker run -t --entrypoint=/bin/cp -v /usr/local/bin:/data ibmcom/helm:v2.5.0  /helm /data/
sudo mkdir -p /var/lib/helm; export HELM_HOME=/var/lib/helm

helm init --client-only
kubectl config view > /home/demo/snap/helm/common/kube/config
#sudo reboot
```




# <a name="ce">Install CE
```bash
sudo docker pull ibmcom/icp-inception:2.1.0-beta-3

sudo docker run -e LICENSE=accept \
  -v "$(pwd)":/data ibmcom/icp-inception:2.1.0-beta-3 cp -r cluster /data
  
docker run -e LICENSE=accept --net=host \
 -t -v "$(pwd)":/installer/cluster \
 ibmcom/icp-inception:2.1.0-beta-3 install | tee install.log  
```





# <a name="ee">Install BETA EE
```bash
tar xf ibm-cloud-private-x86_64-2.1.0-beta-2.tar.gz -O | docker load

docker run -v $(pwd):/data -e LICENSE=accept ibmcom/cfc-installer:2.1.0-beta-2-ee cp -r cluster /data

mkdir -p cluster/images
mv ibm-cloud-private-x86_64-2.1.0-beta-2.tar.gz cluster/images/

sudo cp ~/.ssh/master.id_rsa cluster/ssh_key

docker run --net=host -t -e LICENSE=accept -v $(pwd):/installer/cluster ibmcom/cfc-installer:2.1.0-beta-1-ee install
```



# <a name="config">Configs
### Adapt hosts
```bash
[master]
192.168.232.127

[worker]
192.168.232.127

[proxy]
192.168.232.127
```



### Adapt config.yaml
```bash
cluster_access_ip: 192.168.232.127

```



---
---
---
---
---
---
---
---
---
# <a name="kubectl">KUBECTL Configs
Get Token that doesn’t expire

```bash
kubectl get secrets
kubectl get secret default-token-2n5sf --namespace default -o json > token.json

echo XYZ | base64 -D > token.txt
```

```bash
kubectl config set-cluster mycluster.icp --server=https://192.168.27.100:8001 --insecure-skip-tls-verify=true
kubectl config set-context mycluster.icp-context --cluster=mycluster.icp
kubectl config set-credentials mycluster.icp-user --token=XYZ
kubectl config set-context mycluster.icp-context --user=mycluster.icp-user --namespace=default
kubectl config use-context mycluster.icp-context
```


# <a name="dash">Install Dashboard

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

```bash
# Copyright 2015 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Configuration to deploy release version of the Dashboard UI compatible with
# Kubernetes 1.7.
#
# Example usage: kubectl create -f <this_file>

# ------------------- Dashboard Secret ------------------- #

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque

---
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Role & Role Binding ------------------- #

kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to create and watch for changes of 'kubernetes-dashboard-key-holder' secret.
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create", "watch"]
- apiGroups: [""]
  resources: ["secrets"]
  # Allow Dashboard to get, update and delete 'kubernetes-dashboard-key-holder' secret.
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      initContainers:
      - name: kubernetes-dashboard-init
        image: gcr.io/google_containers/kubernetes-dashboard-init-amd64:v1.0.1
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
      containers:
      - name: kubernetes-dashboard
        image: gcr.io/google_containers/kubernetes-dashboard-amd64:v1.7.1
        ports:
        - containerPort: 9090
          protocol: TCP
        args:
          - --tls-key-file=/certs/dashboard.key
          - --tls-cert-file=/certs/dashboard.crt
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          readOnly: true
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

{
  "apiVersion": "v1",
  "kind": "Service",
  "metadata": {
    "name": "kubernetes-dashboard",
    "namespace": "kube-system",
    "labels": {
      "app": "kubernetes-dashboard"
    }
  },
  "spec": {
    "ports": [
      {
        "name": "k8s-dashboard",
        "protocol": "TCP",
        "port": 9991,
        "targetPort": 8443,
        "nodePort": 30783
      }
    ],
    "selector": {
      "k8s-app": "kubernetes-dashboard"
    },
    "clusterIP": "10.0.0.39",
    "type": "NodePort",
    "sessionAffinity": "None",
    "externalTrafficPolicy": "Cluster"
  }
}

```


# <a name="helm">HELM
```bash

export HELM_HOST=10.1.219.68:44134
helm init
  
helm repo add DEMO https://raw.githubusercontent.com/niklaushirt/chartslocal/master/repo/stable/
helm repo update

helm fetch https://raw.githubusercontent.com/niklaushirt/chartslocal/master/repo/stable/nodered-0.17.5.tgz

helm list

helm search urban


helm install DEMO/urbancode
helm install DEMO/nodered

helm delete sullen-crocodile



Create HELM Package

sudo usermod -aG docker demo

helm init

helm create urbancode
helm lint --strict urbancode


helm package urbancode

helm search -l

wget https://raw.githubusercontent.com/niklaushirt/charts/master/index.yaml

helm repo index --merge index.yaml --url https://raw.githubusercontent.com/niklaushirt/charts/master/repo/stable/ ./


helm repo index --merge index.yaml --url https://raw.githubusercontent.com/niklaushirt/charts/master/repo/stable/ ./


git add .
git commit -m "Urbancode"
git push

##PREPARE
sudo docker pull bitnami/mariadb:10.1.26-r0
sudo docker pull bitnami/mariadb:10.1.23-r2
sudo docker pull bitnami/wordpress:4.8.1-r0
sudo docker pull bitnami/ghost:0.11.10-r2
sudo docker pull gcr.io/google-samples/gb-frontend:v4
sudo docker pull gcr.io/google_samples/gb-redisslave:v1
sudo docker pull gcr.io/google_containers/redis:e2e
sudo docker pull bitnami/mariadb:10.1.23-r2
sudo docker pull ibmcom/ucds:6.2.5.2.929926
sudo docker pull ibmcom/ucda:6.2.5.2.929926
sudo docker pull ibmcom/ucdr:6.2.5.2.929926
sudo docker pull nginx
##


bitnami/mariadb                                       10.1.23-r2
bitnami/wordpress                                     4.8.1-r0
bitnami/wordpress                                     4.8.1-r0



docker pull nginx
docker login master.cfc:8500
docker tag nginx master.cfc:8500/default/nginx:0.1
docker push master.cfc:8500/default/nginx:0.1
docker pull master.cfc:8500/namespace/image1:0.1

docker pull nginx
docker login mycluster:8500
docker tag nginx mycluster:8500/default/nginx:0.1
docker push mycluster:8500/default/nginx:0.1
docker pull mycluster:8500/namespace/image1:0.1

docker tag ibmcom/ucds:6.2.2.0 mycluster:8500/devops/ucds:6.2.2.0
docker tag ibmcom/ucda:6.2.2.0 mycluster:8500/devops/ucda:6.2.2.0
docker tag ibmcom/ucdr:6.2.2.0 mycluster:8500/devops/ucdr:6.2.2.0

docker push mycluster:8500/devops/ucds:6.2.2.0
docker push mycluster:8500/devops/ucda:6.2.2.0
docker push mycluster:8500/devops/ucdr:6.2.2.0



1) on master, get `helm serve container` via `docker ps | grep helm`
2) copy your chart via `docker cp x.tar.gz dockerID:/local-repo`
3) docker restart <helm container id>
4) go to dashboard, click `sync up repo`, you will get your chart in app center


POD=$(kubectl get pods -l app=test-chaoskube --namespace default --output name)
kubectl logs -f $POD --namespace=default
```


# Set HELM HOST - get from Tiller service
```bash
export HELM_HOST=10.1.243.194:44134
```




# <a name="image">Image preload
```bash
docker login mycluster.icp:8500

sudo docker pull gcr.io/google_containers/redis:e2e
sudo docker pull bitnami/mariadb:10.1.23-r2
sudo docker pull ibmcom/ucds:6.2.5.2.929926
sudo docker pull ibmcom/ucda:6.2.5.2.929926
sudo docker pull ibmcom/ucdr:6.2.5.2.929926
sudo docker pull bitnami/mariadb:10.1.26-r0
sudo docker pull bitnami/wordpress:4.8.1-r0
sudo docker pull bitnami/ghost:0.11.10-r2
sudo docker pull gcr.io/google-samples/gb-frontend:v4
sudo docker pull gcr.io/google_samples/gb-redisslave:v1
sudo docker pull nginx
sudo docker pull nodered/node-red-docker:0.17.5

docker tag ibmcom/ucds:6.2.5.2.929926 mycluster.icp:8500/devops/ucds:6.2.5.2.929926
docker push mycluster.icp:8500/devops/ucds:6.2.5.2.929926

docker tag ibmcom/ucda:6.2.5.2.929926 mycluster.icp:8500/devops/ucda:6.2.5.2.929926
docker push mycluster.icp:8500/devops/ucda:6.2.5.2.929926

docker tag ibmcom/ucdr:6.2.5.2.929926 mycluster.icp:8500/devops/ucdr:6.2.5.2.929926
docker push mycluster.icp:8500/devops/ucdr:6.2.5.2.929926

docker tag gcr.io/google_containers/redis:e2e mycluster.icp:8500/default/google_containers/redis:e2e
docker push mycluster.icp:8500/default/google_containers/redis:e2e

docker tag bitnami/mariadb:10.1.26-r0 mycluster.icp:8500/default/mariadb:10.1.26-r0
docker push mycluster.icp:8500/default/mariadb:10.1.26-r0

docker tag bitnami/mariadb:10.1.23-r2 mycluster.icp:8500/default/mariadb:10.1.23-r2
docker push mycluster.icp:8500/default/mariadb:10.1.23-r2

docker tag bitnami/wordpress:4.8.1-r0 mycluster.icp:8500/default/wordpress:4.8.1-r0
docker push mycluster.icp:8500/default/wordpress:4.8.1-r0

docker tag bitnami/ghost:0.11.10-r2 mycluster.icp:8500/default/ghost:0.11.10-r2
docker push mycluster.icp:8500/default/ghost:0.11.10-r2

docker tag gcr.io/google-samples/gb-frontend:v4 mycluster.icp:8500/default/gb-frontend:v4
docker push mycluster.icp:8500/default/gb-frontend:v4

docker tag gcr.io/google_samples/gb-redisslave:v1 mycluster.icp:8500/default/gb-redisslave:v1
docker push mycluster.icp:8500/default/gb-redisslave:v1

docker tag nginx mycluster.icp:8500/default/nginx
docker push mycluster.icp:8500/default/nginx

docker tag nodered/node-red-docker:0.17.5 mycluster.icp:8500/default/node-red-docker:0.17.5
docker push mycluster.icp:8500/default/node-red-docker:0.17.5
```

## <a name="ldap"></a>LDAP
### What are CN, OU, DC?

From RFC2253 (UTF-8 String Representation of Distinguished Names):

```bash
String  X.500 AttributeType
------------------------------
CN      commonName
L       localityName
ST      stateOrProvinceName
O       organizationName
OU      organizationalUnitName
C       countryName
STREET  streetAddress
DC      domainComponent
UID     userid
```

#### What does the string from that query mean?

The string ("CN=Dev-India,OU=Distribution Groups,DC=gp,DC=gl,DC=google,DC=com") is a path from an hierarchical structure (DIT = Directory Information Tree) and should be read from right (root) to left (leaf).

It is a DN (Distinguished Name) (a series of comma-separated key/value pairs used to identify entries uniquely in the directory hierarchy). The DN is actually the entry's fully qualified name.

Here you can see an example where I added some more possible entries.
The actual path is represented using green.

The following paths represent DNs (and their value depends on what you want to get after the query is run):

"DC=gp,DC=gl,DC=google,DC=com"
"OU=Distribution Groups,DC=gp,DC=gl,DC=google,DC=com"
"OU=People,DC=gp,DC=gl,DC=google,DC=com"
"OU=Groups,DC=gp,DC=gl,DC=google,DC=com"
"CN=QA-USA,OU=Distribution Groups,DC=gp,DC=gl,DC=google,DC=com"
"CN=Dev-India,OU=Distribution Groups,DC=gp,DC=gl,DC=google,DC=com"
"CN=Ted Owen,OU=People,DC=gp,DC=gl,DC=google,DC=com"



### Install and configure OpenLDAP

Get ICP LDAP Log

```bash
docker ps | grep platform-identity-mgmt_auth
docker logs e3e93ab118b0
```

Install

```bash
sudo apt install slapd ldap-utils

sudo dpkg-reconfigure slapd
```

Add Content


```bash
nano  add_content.ldif

dn: ou=People,dc=ibm,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=ibm,dc=com
objectClass: organizationalUnit
ou: Groups

dn: cn=miners,ou=Groups,dc=ibm,dc=com
objectClass: posixGroup
cn: miners
gidNumber: 5000

dn: cn=icp,ou=People,dc=ibm,dc=com
objectClass: posixGroup
cn: icp
gidNumber: 5001

dn: uid=john,ou=People,dc=ibm,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
objectClass: person
uid: john
sn: Doe
givenName: John
cn: John Doe
displayName: John Doe
uidNumber: 10000
gidNumber: 5000
userPassword: johnldap
gecos: John Doe
loginShell: /bin/bash
homeDirectory: /home/john
```

Search

```bash
ldapadd -x -D cn=admin,dc=ibm,dc=com -W -f add_content.ldif

ldapsearch -x -LLL -b dc=ibm,dc=com 'uid=john' cn gidNumber
```

### Config ICP

```bash
cn=admin,dc=ibm,dc=com

ID:					openLDAP
Realm:				openLDAPRealm
Server host:		192.168.27.100
Port:				389
Base DN:			dc=ibm,dc=com
Bind DN:			cn=admin,dc=ibm,dc=com
Admin password:	Enter password
```

```bash
LDAP Filters

Group filter:				(&(cn=%v)(objectclass=groupOfUniqueNames))
Group ID map:				*:cn
Group member ID map:	groupOfUniqueNames:uniquemember
User filter:				(&(uid=%v)(objectclass=person))
User ID map:				*:uid

```


## OLD
```bash


ssh-keygen -b 4096 -t rsa -f ~/.ssh/master.id_rsa -N ""
scp ~/.ssh/master.id_rsa.pub root@192.168.232.127:~/.ssh/master.id_rsa.pub
cat ~/.ssh/master.id_rsa.pub | sudo tee -a /root/.ssh/authorized_keys
sudo ufw disable

sudo docker pull ibmcom/icp-inception:2.1.0-beta-2

ssh-keygen -t rsa -P ''
ssh-copy-id -i ~/.ssh/id_rsa root@localhost

sudo docker run -e LICENSE=accept -v "$(pwd)":/data ibmcom/icp-inception:2.1.0-beta-2 cp -r cluster /data

sudo cp ~/.ssh/id_rsa ~/cluster/ssh_key
sudo chmod 400 ~/cluster/ssh_key

docker run -e LICENSE=accept --net=host -t -v "$(pwd)":/installer/cluster ibmcom/icp-inception:2.1.0-beta-2 install
```

