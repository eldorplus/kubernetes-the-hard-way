# Provisionnement des ressources Compute

Kubernetes nécessite un ensemble de machines pour accueillir le plan de contrôle de Kubernetes et les nœuds de travail où les conteneurs sont finalement exécutés. Dans ce laboratoire, vous fournirez les ressources de compute nécessaires au fonctionnement d'un cluster Kubernetes sécurisé et hautement disponible dans une [zone compute unique](https://cloud.google.com/compute/docs/regions-zones/regions-zones).

> Assurez-vous qu'une zone compute et une région par défaut ont été définies comme décrit dans [les prérequis](01-prerequisites.md#définir-une-région-et-une-zone-compute-par-défaut).

## Mise en réseau

[Le modèle de réseau](https://kubernetes.io/docs/concepts/cluster-administration/networking/#the-kubernetes-network-model) Kubernetes suppose un réseau plat dans lequel les conteneurs et les nœuds peuvent communiquer entre eux. Dans les cas où cela n'est pas souhaité, [les politiques de réseau](https://kubernetes.io/docs/concepts/services-networking/network-policies/) peuvent limiter la manière dont les groupes de conteneurs sont autorisés à communiquer entre eux et avec les points terminaux du réseau externe.

> La mise en place de politiques de réseau n'entre pas dans le cadre de ce tutoriel.

### Réseau cloud privés virtuels [VPC]

Dans cette section, un réseau dédié [Virtual Private Cloud](https://cloud.google.com/compute/docs/networks-and-firewalls#networks) sera mis en place pour héberger le cluster Kubernetes.

Créer le réseau VPC personnalisé `kubernetes-the-hard-way` :

```sh
gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
```

Un [sous-réseau](https://cloud.google.com/compute/docs/vpc/#vpc_networks_and_subnets) doit être doté d'une plage d'adresses IP suffisamment large pour attribuer une adresse IP privée à chaque nœud de la grappe Kubernetes.

Créer le sous-réseau `kubernetes` dans le réseau VPC `kubernetes-the-hard-way` :

```sh
gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24
```

> La plage d'adresses IP `10.240.0.0/24` peut accueillir jusqu'à 254 instances de compute.

### Cloud NAT

Dans ce tutoriel, nous allons mettre en place des kubernetes avec des nœuds privés (c'est-à-dire que les nœuds n'ont pas d'IP public). Nous avons besoin d'un moyen pour les nœuds de se connecter à Internet (pour télécharger des images de conteneurs lorsque nous déployons une application par exemple), c'est à cela que sert une passerelle NAT, nous utiliserons [Google Cloud NAT](https://cloud.google.com/nat/docs/overview) qui est une passerelle NAT entièrement gérée ()

Créer un Google Cloud Router

```
gcloud compute routers create kube-nat-router --network kubernetes-the-hard-way
```

Créer une passerelle Google Cloud NAT

```
gcloud compute routers nats create kube-nat-gateway \
  --router=kube-nat-router \
  --auto-allocate-nat-external-ips \
  --nat-all-subnet-ip-ranges
```

### Règles relatives aux pare-feu

Créer une règle de pare-feu qui permet la communication interne à travers tous les protocoles :

```sh
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
```

Créer une règle de pare-feu qui autorise l'ICMP externe, et le HTTPS :

```sh
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0
```

Créer une règle de pare-feu qui autorise le netblock IAP ([Identity Aware Proxy](https://cloud.google.com/iap/docs/concepts-overview)), ceci est nécessaire pour configurer SSH avec [IAP Forwarding](https://cloud.google.com/iap/docs/using-tcp-forwarding) plus tard :

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-iap \
  --allow tcp \
  --network kubernetes-the-hard-way \
  --source-ranges 35.235.240.0/20
```


> An [external load balancer](https://cloud.google.com/compute/docs/load-balancing/network/) will be used to expose the Kubernetes API Servers to remote clients.

List the firewall rules in the `kubernetes-the-hard-way` VPC network:

```sh
gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"
```

> output

```sh
NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY
kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp
kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp
```

### Kubernetes Public IP Address

Allocate a static IP address that will be attached to the external load balancer fronting the Kubernetes API Servers:

```sh
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)
```

Verify the `kubernetes-the-hard-way` static IP address was created in your default compute region:

```sh
gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
```

> output

```sh
NAME  ADDRESS/RANGE  TYPE      PURPOSE  NETWORK  REGION    SUBNET  STATUS
kubernetes-the-hard-way  XX.XXX.XXX.XX  EXTERNAL                    us-west1          RESERVED
```

## Compute Instances

The compute instances in this lab will be provisioned using [Ubuntu Server](https://www.ubuntu.com/server) 18.04, which has good support for the [containerd container runtime](https://github.com/containerd/containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Kubernetes Controllers

Create three compute instances which will host the Kubernetes control plane:

```sh
for i in 0 1 2; do
  gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller \
    --no-address
done
```

### Kubernetes Workers

Each worker instance requires a pod subnet allocation from the Kubernetes cluster CIDR range. The pod subnet allocation will be used to configure container networking in a later exercise. The `pod-cidr` instance metadata will be used to expose pod subnet allocations to compute instances at runtime.

> The Kubernetes cluster CIDR range is later defined by the Controller Manager's `--cluster-cidr` flag. In this tutorial the cluster CIDR range will be set to `10.200.0.0/16`, which supports 254 `/24` pod subnets.

Create three compute instances which will host the Kubernetes worker nodes:

```sh
for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker \
    --no-address
done
```

### Verification

List the compute instances in your default compute zone:

```sh
gcloud compute instances list
```

> output

```sh
NAME          ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
controller-0  us-west1-c  n1-standard-1               10.240.0.10  XX.XXX.XXX.XXX  RUNNING
controller-1  us-west1-c  n1-standard-1               10.240.0.11  XX.XXX.X.XX     RUNNING
controller-2  us-west1-c  n1-standard-1               10.240.0.12  XX.XXX.XXX.XX   RUNNING
worker-0      us-west1-c  n1-standard-1               10.240.0.20  XXX.XXX.XXX.XX  RUNNING
worker-1      us-west1-c  n1-standard-1               10.240.0.21  XX.XXX.XX.XXX   RUNNING
worker-2      us-west1-c  n1-standard-1               10.240.0.22  XXX.XXX.XX.XX   RUNNING
```

## Configuring SSH Access

SSH will be used to configure the controller and worker instances. But because our nodes are private we cannot ssh directly into them from the internet, instead we will be using [Identity Aware Proxy](https://cloud.google.com/iap/docs/concepts-overview) with a feature called [TCP forwarding](https://cloud.google.com/iap/docs/using-tcp-forwarding).

Grant your current user the iap.tunnelResourceAccessor IAM role.

```sh
  export USER=$(gcloud auth list --format="value(account)")

  export PROJECT=$(gcloud config list --format="value(core.project)")

  gcloud projects add-iam-policy-binding $PROJECT \
    --member=user:$USER \
    --role=roles/iap.tunnelResourceAccessor
```

When connecting to compute instances for the first time SSH keys will be generated for you and stored in the project or instance metadata as described in the [connecting to instances](https://cloud.google.com/compute/docs/instances/connecting-to-instance) documentation.

Test SSH access to the `controller-0` compute instances:

```sh
gcloud compute ssh controller-0
```

If this is your first time connecting to a compute instance SSH keys will be generated for you. Enter a passphrase at the prompt to continue:

```sh
WARNING: The public SSH key file for gcloud does not exist.
WARNING: The private SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

At this point the generated SSH keys will be uploaded and stored in your project:

```sh
Your identification has been saved in /home/$USER/.ssh/google_compute_engine.
Your public key has been saved in /home/$USER/.ssh/google_compute_engine.pub.
The key fingerprint is:
SHA256:nz1i8jHmgQuGt+WscqP5SeIaSy5wyIJeL71MuV+QruE $USER@$HOSTNAME
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|                 |
|        .        |
|o.     oS        |
|=... .o .o o     |
|+.+ =+=.+.X o    |
|.+ ==O*B.B = .   |
| .+.=EB++ o      |
+----[SHA256]-----+
Updating project ssh metadata...-Updated [https://www.googleapis.com/compute/v1/projects/$PROJECT_ID].
Updating project ssh metadata...done.
Waiting for SSH key to propagate.
```

After the SSH keys have been updated you'll be logged into the `controller-0` instance:

```sh
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-1042-gcp x86_64)
...
Last login: Sun Sept 14 14:34:27 2019 from XX.XXX.XXX.XX
```

Type `exit` at the prompt to exit the `controller-0` compute instance:

```sh
$USER@controller-0:~$ exit
```

> output

```sh
logout
Connection to XX.XXX.XXX.XXX closed
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
