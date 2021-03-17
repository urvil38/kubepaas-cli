# KUBEPAAS
### **KUBEPAAS let you Focus on application rather than managing kubernetes.**

Deploy your web app on kubernetes with ease, no more YAML to write by hand, take advantage of automation with well defined API.

![kubepaas-flow-diagram](https://github.com/urvil38/kubepaas-cli/blob/master/doc/images/kubepaas-flow.png)

# Usage

```

██╗  ██╗██╗   ██╗██████╗ ███████╗██████╗  █████╗  █████╗ ███████╗
██║ ██╔╝██║   ██║██╔══██╗██╔════╝██╔══██╗██╔══██╗██╔══██╗██╔════╝
█████╔╝ ██║   ██║██████╔╝█████╗  ██████╔╝███████║███████║███████╗
██╔═██╗ ██║   ██║██╔══██╗██╔══╝  ██╔═══╝ ██╔══██║██╔══██║╚════██║
██║  ██╗╚██████╔╝██████╔╝███████╗██║     ██║  ██║██║  ██║███████║
╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚══════╝╚═╝     ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝

A tool for interacting with kubepaas platform
and used for all kind of command that This plateform will support

Usage:
  kubepaas [flags]
  kubepaas [command]

Available Commands:
  config      view and edit Kubepaas CLI properties
  deploy      Deploy the application to kubepaas platform
  help        Help about any command
  login       Log in to kubepaas platform
  logout      Log out from kubepaas platform
  logs        Print the logs of current running containers
  profile     A brief description of your command
  rollout     RollBack to Older or Newer Version
  signup      Sign up for kubepaas platform
  update      Used for changing values of diffrent configuration

Flags:
  -h, --help   help for kubepaas

Use "kubepaas [command] --help" for more information about a command.
```

# Download

- Download appropriate pre-compiled binary from the [release](https://github.com/urvil38/kubepaas/releases) page.

```
# download binary using cURL
$ curl -L https://github.com/urvil38/kubepaas/releases/download/0.0.1/kubepaas-darwin-amd64 -o kubepaas

# make binary executable
$ chmod +x ./kubepaas

# move it to bin dir (user need to has root privileges. run following command as root user using sudo.
$ sudo mv ./kubepaas /usr/local/bin
```


- Download using `go get`

```
$ go get -u github.com/urvil38/kubepaas
```

# Build

- If you want to build kubepaas right away, you need a working [Go environment](https://golang.org/doc/install). It requires Go version 1.12 and above.

```
$ git clone https://github.com/urvil38/kubepaas.git
$ cd kubepaas
$ make build
```

# Getting Started
Following tutorial walks you through setting up kubepaas kubernetes cluster and deploying a sample web application. If you already have kubepaas cluster set up then you can skip to [this](#deploying-web-app) section.

## Cluster setup

Kubepaas platform leverages the Google Cloud Platform to streamline provisioning of the kubernetes cluster using GKE and other resources like cloud storage and DNS.

### Prerequisites

- Active Google cloud account
- Domain name (You needs to have access to domain config panel to configure namespace servers)
- gcloud
- kubectl

I had tried writing shell script to bootstrap all the necessary resources on GCP, however, it becomes clear that I need to write a helper tool to make this process easy. The result is CLI tool called [kmanager](https://github.com/urvil38/kmanager). It's a simple CLI written in Go, which bootstraps the cluster and install kubernetes add-on like ingress controller, cert-manager, kubepaas-generator (config file generation service), etc.

1. Download latest kmanager CLI by following [this](https://github.com/urvil38/kmanager#download) installation guide.
2. Begin cluster creation process by running following command.

      ```sh
      kmanager create
      ```

      This will ask you the following questions:

      ```
      ? Enter Cluster name : [? for help]
      ```
      This name will be used as GKE cluster name. It should be unique across your current gcp account.

      ```
      ? Enter Domain name : [? for help]
      ```
      Provide a valid domain name which is owned by you. This will be a crucial step and this domain name will be used as a suffix. i.e. hello-world.domain.com, api.domain.com

      ```
      ? Choose region:  [Use arrows to move, type to filter]
      ? Choose zone:  [Use arrows to move, type to filter]
      ```
      Provide a region and zone in which you want to deploy your GKE cluster


      ```
      ? Choose google cloud project:  [Use arrows to move, type to filter]
      ```
      Provide GCP project name in which you want to create all the resources.

      Setting up the cluster will take approx. 10 minutes. Grab your favorite beverage and enjoy looking at the spaghetti of gcloud and kubectl command being run by this tool.

3. After the completion of this command verify that your cluster setup is succeeded, by doing the following cURL request:
    ```
    curl https://generator.[domain-name]
    ```

    this should print `Welcome to kubepaas generator service`.

## Deploying web app

This tutorial assumes you have access to kubepaas cluster.

1. Download latest kubepaas CLI by following [this](https://github.com/urvil38/kubePAAS#download) installation guide.
2. Configure kubepaas CLI to point it to kubepaas cluster.

    ```
    kubepaas config set generator-endpoint https://generator.[domain-name]
    ```

3. We are going to deploy a simple Go Application:

    - clone following sample repo:
      ```
      git clone https://github.com/urvil38/matrix.git
      cd matrix
      ```

    - This repo contains a web server that serves static files.
    - It also has a file called `app.yml` which is a manifest file. It provides additional information like runtime, version, port details to help configure this web app on kubepaas cluster.

4. Run `kubepaas deploy` from the root of your project(i.e. from the directory which have `app.yml` manifest).

    - This will start the deployment process. It will first create a Dockerfile if not exists from the manifest, after that it will start building docker image and push it to GCR. It then generates the kuberenets config and deploys it to Kubernetes.

5. That's it. You should be able to access the web app on https://[project-name].[domain-name] i.e. https://matrix.[domain-name].

6. Check logs using following command:

    ```
    kubepaas logs -f
    ```