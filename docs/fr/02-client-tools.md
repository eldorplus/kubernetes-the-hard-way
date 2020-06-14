# Installation des outils du client

Dans ce laboratoire, vous installerez les utilitaires en ligne de commande nécessaires à la réalisation de ce tutoriel : [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), et [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).

## Installer le CFSSL

Les utilitaires en ligne de commande `cfssl` et `cfssljson` seront utilisés pour fournir une [infrastructure PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure) et générer des certificats TLS.

Télécharger et installer  `cfssl` et `cfssljson`:

### macOS

```sh
curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/darwin/cfssl
curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/darwin/cfssljson
```

```sh
chmod +x cfssl cfssljson
```

```sh
sudo mv cfssl cfssljson /usr/local/bin/
```

Certains utilisateurs de macOS peuvent rencontrer des problèmes en utilisant les binaires pré-construits, auquel cas le [Homebrew](https://brew.sh) pourrait être une meilleure option :

```sh
brew install cfssl
```

### Linux

```sh
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssljson
```

```sh
chmod +x cfssl cfssljson
```

```sh
sudo mv cfssl cfssljson /usr/local/bin/
```

### vérification du cfssl

Vérifiez que `cfssl` et `cfssljson` version 1.3.4 ou supérieure est installée :

```sh
cfssl version
```

> output

```sh
Version: 1.3.4
Revision: dev
Runtime: go1.13
```

```sh
cfssljson --version
```

```sh
Version: 1.3.4
Revision: dev
Runtime: go1.13
```

## Installer kubectl

L'utilitaire en ligne de commande `kubectl` est utilisé pour interagir avec le serveur API Kubernetes. Téléchargez et installez `kubectl` à partir des binaires de la version officielle :

### kubectl sur macOS

```sh
curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/darwin/amd64/kubectl
```

```sh
chmod +x kubectl
```

```sh
sudo mv kubectl /usr/local/bin/
```

### kubectl sur Linux

```sh
wget https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl
```

```sh
chmod +x kubectl
```

```sh
sudo mv kubectl /usr/local/bin/
```

### Vérification de kubectl

Vérifiez que la version 1.18.0 ou supérieure de `kubectl` est installée :

```sh
kubectl version --client
```

> output

```sh
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:18:23Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"darwin/amd64"}
```

Suivant : [Provisionnement des ressources Compute](03-compute-resources.md)
