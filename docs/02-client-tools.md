# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial: [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).

## Install CFSSL

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

Download and install `cfssl` and `cfssljson`:

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

Some macOS users may experience problems using the pre-built binaries in which case [Homebrew](https://brew.sh) might be a better option:

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

### cfssl Verification

Verify `cfssl` and `cfssljson` version 1.3.4 or higher is installed:

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

## Install kubectl

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

### kubectl on macOS

```sh
curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.16.2/bin/darwin/amd64/kubectl
```

```sh
chmod +x kubectl
```

```sh
sudo mv kubectl /usr/local/bin/
```

### kubectl on Linux

```sh
wget https://storage.googleapis.com/kubernetes-release/release/v1.16.2/bin/linux/amd64/kubectl
```

```sh
chmod +x kubectl
```

```sh
sudo mv kubectl /usr/local/bin/
```

### kubectl Verification

Verify `kubectl` version 1.16.2 or higher is installed:

```sh
kubectl version --client
```

> output

```sh
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:18:23Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"darwin/amd64"}
```

Next: [Provisioning Compute Resources](03-compute-resources.md)
