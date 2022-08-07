Jenkins meet Monk
===

This repository contains Monk.io template to deploy Jenkins system either locally or on cloud of your choice (AWS, GCP, Azure, Digital Ocean).  

This template uses [Traefik](https://traefik.io/) as a reverse proxy to provide full HTTPS encryption.  
Traefik will try to obtain full SSL certificate from Lets Encrypt(assuming valid hostname is provided in the `manifest.yaml`) and if not it will fallback to a default self generated SSL certificate.  

If you wish to disable Traefik(for example you have your own reverse proxy) please look at [Variables](#variables) section.  

  - [Prerequisites](#prerequisites)
    - [Make sure monkd is running.](#make-sure-monkd-is-running)
    - [Clone Repository](#clone-repository)
    - [Load Template](#load-template)
    - [Verify if it's loaded correctly](#verify-if-its-loaded-correctly)
  - [Deploy Stack](#deploy-stack)
    - [Deploy Jenkins Server](#deploy-jenkins-server)
    - [Get admin credentials](#get-admin-credentials)
  - [Variables](#variables)
  - [Stop, remove and clean up workloads and templates](#stop-remove-and-clean-up-workloads-and-templates)
  - [Persistency](#persistency)

## Prerequisites
- [Install Monk](https://docs.monk.io/docs/get-monk)
- [Register and Login Monk](https://docs.monk.io/docs/acc-and-auth)
- [Add Cloud Provider](https://docs.monk.io/docs/cloud-provider)
- [Add Instance](https://docs.monk.io/docs/multi-cloud)

### Make sure monkd is running.

```bash
foo@bar:~$ monk status
daemon: ready
auth: logged in
not connected to cluster
```

### Clone Repository

```bash
git clone git@github.com:CuteAnonymousPanda/monk-jenkins.git
```

### Load Template

```bash
cd monk-jenkins
monk load manifest.yaml
```

### Verify if it's loaded correctly

```bash
$ monk list -l jenkins

✔ Got the list
Type      Template        Repository  Version  Tags  
runnable  jenkins/server  local       -        -     
```

## Deploy Stack

### Deploy Jenkins Server

```bash
$ monk run jenkins/server
? Select tag to run [local/jenkins/server] on: aws
✔ Starting the job: local/jenkins/server... DONE
✔ Preparing nodes DONE
✔ Checking/pulling images...
✔ [================================================] 100% docker.io/library/traefik:v2.8 set0
✔ [================================================] 100% docker.io/jenkins/jenkins:lts set0
✔ Checking/pulling images DONE
✔ Started local/jenkins/server

🔩 templates/local/jenkins/server
 └─🧊 Peer set0
    ├─🔩 templates/local/jenkins/server 
    │  └─📦 cee658ced4af3594fb80bc3a44f387a1-local-jenkins-server-traefik 
    │     ├─🧩 docker.io/library/traefik:v2.8             
    │     ├─💾 /var/run/podman/podman.sock -> /var/run/docker.sock
    │     ├─🔌 open 54.221.107.115:443 -> 443  
    │     └─🔌 open 54.221.107.115:80 -> 80    
    └─🔩 templates/local/jenkins/server 
       └─📦 3742bed532158b00f1bc4f0273fffbb2-local-jenkins-server-jenkins 
          ├─🧩 docker.io/jenkins/jenkins:lts              
          ├─💾 /var/lib/monkd/volumes/jenkins -> /var/jenkins_home
          └─🔌 open 54.221.107.115:50000 -> 50000

💡 You can inspect and manage your above stack with these commands:
        monk logs (-f) local/jenkins/server - Inspect logs
        monk shell     local/jenkins/server - Connect to the container's shell
        monk do        local/jenkins/server/action_name - Run defined action (if exists)
💡 Check monk help for more!
```

### Get admin credentials

``` bash
$ monk do jenkins/server/getAdminCredentials
✔ Get templates/local/jenkins/server actions list success
✔ Got action parameters
✔ Parse parameters success
✔ Running action: 
ed09f15dd060481fa01cc2e2db388611
✨ Took: 2s
```

## Variables

The variables are stored in `manifest.yaml` file.  
You can quickly setup by editing the values there.

| Variable                     | Description                                                                   | Default              |
|------------------------------|-------------------------------------------------------------------------------|----------------------|
| hostname                     | Your Jenkins master hostname                                                  | <- ip-address-public |
| enable-traefik               | If disabled traffic will be routed only to Jenkins via non-ssl encrypted port | true                 |
| http-port                    | Port that Traefik/Jenkins will listen for non SSL traffic                     | 80                   |
| https-port                   | Port that Traefik will listen for SSL traffic                                 | 443                  |

## Stop, remove and clean up workloads and templates

```bash
monk purge    jenkins/server
monk purge -x jenkins/server
```

## Persistency
If you're using any of the clouds available via Monk. You can use volume definition to spin a disk block device to make your Jenkins instance independent from the node it's running on.  
To do simply uncomment the `volume` block in `manifest.yaml`
