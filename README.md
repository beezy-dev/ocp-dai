# Assisted Installer for Red Hat OpenShift

This example provides the steps to deploy a single node bare metal Red Hat OpenShift to test the Virtualization capabilities. 

## Requirements

The hardware requirements are:   
* 4 cores (+ the cores required to run your workloads)
* 16 GB (+ the RAM required to run your workloads)
* one disk of 120GB for RHCOS installation
* one disk of 50GB+ for Red Hat OpenShift Virtualization

The infrastructure requirements:
* a static IP address or reserved in a DHCP 
* A or CNAME record for api.<cluster-name>.<subdomain>.<domain>.<tld> pointing to the IP axdress
* A or CNAME record for api-int.<cluster-name>.<subdomain>.<domain>.<tld> pointing to the IP axdress
* A or CNAME record for *.apps.<cluster-name>.<subdomain>.<domain>.<tld> pointing to the IP axdress

Recover the pull secret for the Red Hat Container Registries:
* The ```pullSecret``` is linked to your customer account and can be restrieved [here](https://console.redhat.com/openshift/install/pull-secret).

In this example, our DNS parameters are:
* <cluster-name> -> pocvirt1
* <subdomain> -> test
* <domain> -> org
* <tld> -> int
* or pocvirt1.test.org.int

## Tooling
### OCP CLIs

Grab the OC CLI:
```
curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz -o oc.tar.gz
```

Grab the OC installer:
```
curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz -o openshift-install-linux.tar.gz
```

Decompress the archives:
```
tar xfv oc.tar.gz
tar xfv openshift-install-linux.tar.gz
```

Deploy the binaries in ```/usr/local/bin```:
```
sudo install kubectl /usr/local/bin/
sudo install oc /usr/local/bin/
sudo install openshift-install /usr/local/bin/
```

### Full disconnected deployment

When deploying Red Hat OpenShift, images will be fetched on container registeries. If a disconnected environment is required, the followings are required:
* A or CNAME record for registry.<subdomain>.<domain>.<tld> pointing to the IP where the registry will be hosted.
* Those images needs to be mirrored locally using ```oc mirror```.

In this example, our DNS parameters are:
* registry.test.org.int 

Grab the OC mirror:
```
curl -k https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/latest/oc-mirror.tar.gz -o oc-mirror.tar.gz
``` 

Grab the mirror registry:
```
curl -k https://developers.redhat.com/content-gateway/file/pub/openshift-v4/clients/mirror-registry/1.3.10/mirror-registry.tar.gz -o mirror-registry.tar.gz
```

Decompress the archive:
```
tar xfv oc-mirror.tar.gz
tar xfv mirror-registry.tar.gz 
```

Deploy the binaries in ```/usr/local/bin```:
```
sudo install oc-mirror /usr/local/bin/
``` 

Create the registry directory and change ownership to current user:
```
sudo mkdir /opt/ocp-registry
sudo chown -R rovandep:rovandep /opt/ocp-registry
```

Start the registry service installation:
```
./mirror-registry install --quayHostname registry.test.org.int --quayRoot /opt/ocp-registry --initUser admin --initPassword redhat123
``` 

Expected output:
```

   __   __
  /  \ /  \     ______   _    _     __   __   __
 / /\ / /\ \   /  __  \ | |  | |   /  \  \ \ / /
/ /  / /  \ \  | |  | | | |  | |  / /\ \  \   /
\ \  \ \  / /  | |__| | | |__| | / ____ \  | |
 \ \/ \ \/ /   \_  ___/  \____/ /_/    \_\ |_|
  \__/ \__/      \ \__
                  \___\ by Red Hat
 Build, Store, and Distribute your Containers

INFO[2024-05-19 12:20:33] Install has begun                            
INFO[2024-05-19 12:20:33] Found execution environment at /home/rovandep/Downloads/dev/redhat/execution-environment.tar 
INFO[2024-05-19 12:20:33] Loading execution environment from execution-environment.tar 
INFO[2024-05-19 12:20:34] Detected an installation to localhost        
INFO[2024-05-19 12:20:34] Found SSH key at /home/rovandep/.ssh/quay_installer 
INFO[2024-05-19 12:20:34] Attempting to set SELinux rules on /home/rovandep/.ssh/quay_installer 
INFO[2024-05-19 12:20:34] Found image archive at /home/rovandep/Downloads/dev/redhat/image-archive.tar 
INFO[2024-05-19 12:20:34] Detected an installation to localhost        
INFO[2024-05-19 12:20:34] Unpacking image archive from /home/rovandep/Downloads/dev/redhat/image-archive.tar 
INFO[2024-05-19 12:20:37] Loading pause image archive from pause.tar   
INFO[2024-05-19 12:20:38] Loading redis image archive from redis.tar   
INFO[2024-05-19 12:20:38] Loading postgres image archive from postgres.tar 
INFO[2024-05-19 12:20:39] Loading Quay image archive from quay.tar     
INFO[2024-05-19 12:20:43] Attempting to set SELinux rules on image archive 
INFO[2024-05-19 12:20:43] Running install playbook. This may take some time. To see playbook output run the installer with -v (verbose) flag. 
INFO[2024-05-19 12:20:43] Detected an installation to localhost        

PLAY [Install Mirror Appliance] **********************************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Expand variables] ***********************************************************
included: /runner/project/roles/mirror_appliance/tasks/expand-vars.yaml for rovandep@spectre

TASK [mirror_appliance : Expand pg_storage] **********************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Expand quay_root] ***********************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Expand quay_storage] ********************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Set expanded variables] *****************************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Install Dependencies] *******************************************************
included: /runner/project/roles/mirror_appliance/tasks/install-deps.yaml for rovandep@spectre

TASK [mirror_appliance : Create user service directory] **********************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Set SELinux Rules] **********************************************************
included: /runner/project/roles/mirror_appliance/tasks/set-selinux-rules.yaml for rovandep@spectre

TASK [mirror_appliance : Set container_manage_cgroup flag on and keep it persistent across reboots] ***
skipping: [rovandep@spectre]

TASK [mirror_appliance : Install Quay Pod Service] ***************************************************
included: /runner/project/roles/mirror_appliance/tasks/install-pod-service.yaml for rovandep@spectre

TASK [mirror_appliance : Copy Quay Pod systemd service file] *****************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Check if pod pause image is loaded] *****************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Pull Infra image] ***********************************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Start Quay Pod service] *****************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Autodetect Image Archive] ***************************************************
included: /runner/project/roles/mirror_appliance/tasks/autodetect-image-archive.yaml for rovandep@spectre

TASK [mirror_appliance : Checking for Image Archive] *************************************************
ok: [rovandep@spectre -> localhost]

TASK [mirror_appliance : Create install directory for image-archive.tar dest] ************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Copy Images if /runner/image-archive.tar exists] ****************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Unpack Images if /runner/image-archive.tar exists] **************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Loading Redis if redis.tar exists] ******************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Loading Quay if quay.tar exists] ********************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Loading Postgres if postgres.tar exists] ************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Install Postgres Service] ***************************************************
included: /runner/project/roles/mirror_appliance/tasks/install-postgres-service.yaml for rovandep@spectre

TASK [mirror_appliance : Create necessary directory for Postgres persistent data] ********************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Set permissions on local storage directory] *********************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Copy Postgres systemd service file] *****************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Check if Postgres image is loaded] ******************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Pull Postgres image] ********************************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Create Postgres Storage named volume] ***************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Start Postgres service] *****************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Wait for pg_trgm to be installed] *******************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Install Redis Service] ******************************************************
included: /runner/project/roles/mirror_appliance/tasks/install-redis-service.yaml for rovandep@spectre

TASK [mirror_appliance : Copy Redis systemd service file] ********************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Check if Redis image is loaded] *********************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Pull Redis image] ***********************************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Start Redis service] ********************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Install Quay Service] *******************************************************
included: /runner/project/roles/mirror_appliance/tasks/install-quay-service.yaml for rovandep@spectre

TASK [mirror_appliance : Create necessary directory for Quay local storage] **************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Set permissions on local storage directory] *********************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Create necessary directory for Quay config bundle] **************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Copy Quay config.yaml file] *************************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Check if SSL Cert exists] ***************************************************
ok: [rovandep@spectre -> localhost]

TASK [mirror_appliance : Check if SSL Key exists] ****************************************************
ok: [rovandep@spectre -> localhost]

TASK [mirror_appliance : Create necessary directory for Quay rootCA files] ***************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Create OpenSSL Config] ******************************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Create root CA key] *********************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Create root CA pem] *********************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Create ssl key] *************************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Create CSR] *****************************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Create self-signed cert] ****************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Create chain cert] **********************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Replace ssl cert with chain cert] *******************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Copy SSL certificate] *******************************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Copy SSL key] ***************************************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Set permissions for key] ****************************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Set permissions for cert] ***************************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Copy Quay systemd service file] *********************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Check if Quay image is loaded] **********************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Pull Quay image] ************************************************************
skipping: [rovandep@spectre]

TASK [mirror_appliance : Create Quay Storage named volume] *******************************************
ok: [rovandep@spectre]

TASK [mirror_appliance : Start Quay service] *********************************************************
changed: [rovandep@spectre]

TASK [mirror_appliance : Wait for Quay] **************************************************************
included: /runner/project/roles/mirror_appliance/tasks/wait-for-quay.yaml for rovandep@spectre

TASK [mirror_appliance : Waiting up to 3 minutes for Quay to become alive at https://registry.test.org.int:8443/health/instance] ***
FAILED - RETRYING: [rovandep@spectre]: Waiting up to 3 minutes for Quay to become alive at https://registry.test.org.int:8443/health/instance (10 retries left).
FAILED - RETRYING: [rovandep@spectre]: Waiting up to 3 minutes for Quay to become alive at https://registry.test.org.int:8443/health/instance (9 retries left).
```


Configure the pull secret:
```
cat pull-secret.json | jq . > pull-mirror.json
```

Generate local credentials for your mirror:
```
echo -n 'admin:redhat123' | base64
```

Expected output:
```
YWRtaW46cmVkaGF0MTIz
```

Edit your pull-mirror.json file to add your local registry;
```JSON
  "auths": {
    "registry.test.org.int": {
      "auth": "YWRtaW46cmVkaGF0MTIz",
      "email": "rovandep@redhat.com",
    },
```
MIND THE SYNTAX!

Authenticate with the Red Hat registries:

```
podman login registry.redhat.io
podman login registry.connect.redhat.com
```

Create an initial template:
```
oc mirror init --registry example.com/mirror/oc-mirror-metadata > imageset-config.yaml 
```

Expected output:
```YAML

```



### ISO

Get the latest RHCOS ISO version:
```
openshift-install coreos print-stream-json | grep location |grep x86|grep iso
```

Expected similar output:
```
"location": "https://rhcos.mirror.openshift.com/art/storage/prod/streams/4.15-9.2/builds/415.92.202402201450-0/x86_64/rhcos-415.92.202402201450-0-live.x86_64.iso",
```

Grab the ISO:
```
curl -k https://rhcos.mirror.openshift.com/art/storage/prod/streams/4.15-9.2/builds/415.92.202402201450-0/x86_64/rhcos-415.92.202402201450-0-live.x86_64.iso -o rhcos-live.iso
```

## Configuration files

The basic template out of the documentation provides the followings:

```YAML
apiVersion: v1
baseDomain: <domain> 
compute:
- name: worker
  replicas: 0 
controlPlane:
  name: master
  replicas: 1 
metadata:
  name: <name> 
networking: 
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16 
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
bootstrapInPlace:
  installationDisk: /dev/disk/by-id/<disk_id> 
pullSecret: '<pull_secret>' 
sshKey: |
  <ssh_key> 
```

A working example is:
```YAML
apiVersion: v1
baseDomain: test.org.int 
compute:
- name: worker
  replicas: 0 
controlPlane:
  name: master
  replicas: 1 
metadata:
  name: pocvirt1 
networking: 
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16 
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
bootstrapInPlace:
  installationDisk: /dev/disk/by-id/<disk_id> 
pullSecret: '<pull_secret>' 
sshKey: |
  <ssh_key> 
```

Notes:  
* The ```installationDisk``` parameter has to be set to your first local disk that you want to use. The second disk will be assigned to the LVM Manager to provide local storage. 
* The ```pullSecret``` is linked to your customer account and can be restrieved [here](https://console.redhat.com/openshift/install/pull-secret).
* The ```sshKey``` parameter is linked to the root account on the RHCOS, if access is required.
* If you require a proxy to access the internet, this can be configured by adding the followings:

```YAML
proxy:
  httpProxy: http://USERNAME:PASSWORD@proxy.example.com:PORT
  httpsProxy: https://USERNAME:PASSWORD@proxy.example.com:PORT
  noProxy: <WILDCARD_OF_DOMAIN>,<PROVISIONING_NETWORK/CIDR>,<BMC_ADDRESS_RANGE/CIDR>
```













