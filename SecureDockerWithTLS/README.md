## Self signed SSL certificate generator and Configure Docker remote API with TLS.

### How to use this scripts?

Step 1: To install `openssl`,`git` and its dependencies. If already installed then don't do it again.
```
root@ubuntu-512mb-blr1-01:~# apt install openssl git
```

Step 2: Clone the repository from GitHub.
```
root@ubuntu-512mb-blr1-01:~# git clone https://github.com/cloudyuga/helperscripts.git
Cloning into 'helperscripts'...
remote: Counting objects: 36, done.
remote: Compressing objects: 100% (31/31), done.
remote: Total 36 (delta 2), reused 35 (delta 1), pack-reused 0
Unpacking objects: 100% (36/36), done.
Checking connectivity... done.
```
#### For generating TLS certificates on Local machine.

Step 3: Go to the `helperscripts/SecureDockerWithTLS/` directory
```
root@ubuntu-512mb-blr1-01:~# cd helperscripts/SecureDockerWithTLS/
```
Scripts shown in the above directory, these are executable files.(Note : Don't change the permission)
```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# ls
dockertls  generatetls  README.md
```
Run the script `generatetls` with output argument `-o`

Use `-o` argument for output and `~/certs` directory path where the certificates get generated.
```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# ./generatetls -o ~/certs
====> Certificate Authority
====> Generating location /root/certs
 ====> Generating CA key
Generating RSA private key, 4096 bit long modulus
.........................................................................................................................................................................................................................................++
................................................................................................................................................++
e is 65537 (0x10001)
 ====> Generating CA
 ====> Generating server key
Generating RSA private key, 4096 bit long modulus
...++
.............................................................................................................................................................................................................................................................++
e is 65537 (0x10001)
 ====> Generating server CSR
 ====> Creating subject alternative name
subjectAltName = IP:45.55.226.76,IP:127.0.0.1
 ====> Signing server CSR with CA
Signature ok
subject=/CN=dockerhost.example.com
Getting CA Private Key
 ====> Generating client key
Generating RSA private key, 4096 bit long modulus
.....................................................................................................................................................................................++
..........................................................................++
e is 65537 (0x10001)
 =====> Generating client CSR
 ===> Creating extended key usage
extendedKeyUsage = clientAuth
 =====> Signing client CSR with CA
Signature ok
subject=/CN=client
Getting CA Private Key
====> Change certificates permission
mode of 'ca-key.pem' changed from 0644 (rw-r--r--) to 0600 (rw-------)
mode of 'ca.pem' changed from 0644 (rw-r--r--) to 0600 (rw-------)
mode of 'cert.pem' changed from 0644 (rw-r--r--) to 0600 (rw-------)
mode of 'key.pem' changed from 0644 (rw-r--r--) to 0600 (rw-------)
mode of 'server-cert.pem' changed from 0644 (rw-r--r--) to 0600 (rw-------)
mode of 'server-key.pem' changed from 0644 (rw-r--r--) to 0600 (rw-------)
 ====> Done!
```
Check generated file in `~/certs/` directory.  There are following certificates files generated in this folder:

```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# ls ~/certs/
ca-key.pem  ca.pem  ca.srl  cert.pem  extfile_client.conf  extfile.conf  key.pem  server-cert.pem  server-key.pem
```
#### For configuring a secure Docker remote API with TLS 

Step 3: Run the script named  `dockertls` with an input argument `-i`

Use `-i` argument for an input and `~/certs` directory path where certificates are located .
```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# ./dockertls -i ~/certs/
====> Certificates location /root/certs/
====> Copy CA key to default location
'ca-key.pem' -> '/etc/docker/ca-key.pem'
====> Copy CA certificate to default location
'ca.pem' -> '/etc/docker/ca.pem'
====> Copy server key to remote location
'server-key.pem' -> '/etc/docker/server-key.pem'
====> Copy server certificate to remote location
'server-cert.pem' -> '/etc/docker/server-cert.pem'
====> Copy client key to remote location
'key.pem' -> '/etc/docker/key.pem'
====> Copy client certificate to remote location
'cert.pem' -> '/etc/docker/cert.pem'
====> Setup Docker remote API with TLS certificates
====> Reload system daemon service
====> Restart Docker service
====> Need to set the following environment variables before running the Docker client
'ca.pem' -> '/root/.docker/ca.pem'
'cert.pem' -> '/root/.docker/cert.pem'
'key.pem' -> '/root/.docker/key.pem'
 ====> Done!
```
After execution of this script Docker daemon is secured with TLS and now docker daemon is running on `2376` port.
Check Docker service status. 
```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/docker.service.d
           └─docker.conf
   Active: active (running) since Fri 2017-01-27 06:33:45 UTC; 33s ago
     Docs: https://docs.docker.com
 Main PID: 29397 (dockerd)
    Tasks: 15
   Memory: 15.6M
      CPU: 289ms
   CGroup: /system.slice/docker.service
           ├─29397 /usr/bin/dockerd -H fd:// -D --tls=true --tlscert=/etc/docker/ca.pem --tlscert=/etc/docker/server-cert.
           └─29404 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 -

Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.948812972Z" level=debug msg="Registerin
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.949148888Z" level=debug msg="Registerin
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.949476233Z" level=debug msg="Registerin
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.949775622Z" level=debug msg="Registerin
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.950183490Z" level=debug msg="Registerin
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.950525811Z" level=debug msg="Registerin
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.950836325Z" level=debug msg="Registerin
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 systemd[1]: Started Docker Application Container Engine.
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.956677649Z" level=info msg="API listen 
Jan 27 06:33:45 ved-ubuntu-512mb-blr1-01 dockerd[29397]: time="2017-01-27T06:33:45.957000498Z" level=info msg="API listen 
```
Test docker TLS connection

Check list of containers present on local machine :
```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# docker --tls=true --tlscert=/etc/docker/ca.pem  --tlscert=/etc/docker/server-cert.pem  --tlskey=/etc/docker/server-key.pem -H=127.0.0.1:2376 ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
Check list of images present on local machine :
```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# docker --tls=true --tlscert=/etc/docker/ca.pem  --tlscert=/etc/docker/server-cert.pem  --tlskey=/etc/docker/server-key.pem -H=127.0.0.1:2376 images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```
Pull an image from a repository:
```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# docker --tls=true --tlscert=/etc/docker/ca.pem  --tlscert=/etc/docker/server-cert.pem  --tlskey=/etc/docker/server-key.pem -H=127.0.0.1:2376 pull alpine
Using default tag: latest                                                                                                             
latest: Pulling from library/alpine                                                                                                   
0a8490d0dfd3: Pull complete 
Digest: sha256:dfbd4a3a8ebca874ebd2474f044a0b33600d4523d03b0df76e5c5986cb02d7e8
Status: Downloaded newer image for alpine:latest
```
Check the imgaes present on local machines:
```
root@ubuntu-512mb-blr1-01:~/helperscripts/SecureDockerWithTLS# docker --tls=true --tlscert=/etc/docker/ca.pem  --tlscert=/etc/docker/server-cert.pem  --tlskey=/etc/docker/server-key.pem -H=127.0.0.1:2376 images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              88e169ea8f46        4 weeks ago         3.98 MB
```  
#### If want to access this docker daemon of local machine which is TLS secured from the different remote machines then we have to copy the certificates on the remote machines.Execute following commands on the remote machine. For me IP of local machine on which certificates and secured docker daemon resides is `45.55.226.76`. Execute following steps on remote machines, with appropriate IP of your local machine.

```
$ mkdir -p /etc/docker
$ scp -r root@45.55.226.76:/etc/docker/* /etc/docker
```
Now check TLS secured Docker daemon at local machine from remote machine. 
```
$ docker --tls=true --tlscert=/etc/docker/ca.pem  --tlscert=/etc/docker/server-cert.pem  --tlskey=/etc/docker/server-key.pem -H=45.55.226.76:2376 ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

```

Lets check the list of images present on local machine.

```
$ docker --tls=true --tlscert=/etc/docker/ca.pem  --tlscert=/etc/docker/server-cert.pem  --tlskey=/etc/docker/server-key.pem -H=45.55.226.76:2376 images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              88e169ea8f46        4 weeks ago         3.98 MB

```
Pull an image or a repository from a registry on local machine from remote machine:

```      
$ docker --tls=true --tlscert=/etc/docker/ca.pem  --tlscert=/etc/docker/server-cert.pem  --tlskey=/etc/docker/server-key.pem -H=45.55.226.76:2376 pull nginx

Using default tag: latest
latest: Pulling from library/nginx
5040bd298390: Pull complete 
333547110842: Pull complete 
4df1e44d2a7a: Pull complete 
Digest: sha256:f2d384a6ca8ada733df555be3edc427f2e5f285ebf468aae940843de8cf74645
Status: Downloaded newer image for nginx:latest
```

If we check the list of images present on the local machine, We can see two images present there: 

```
$ docker --tls=true --tlscert=/etc/docker/ca.pem  --tlscert=/etc/docker/server-cert.pem  --tlskey=/etc/docker/server-key.pem -H=45.55.226.76:2376 images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              cc1b61406712        2 days ago          182 MB
alpine              latest              88e169ea8f46        4 weeks ago         3.98 MB
```

