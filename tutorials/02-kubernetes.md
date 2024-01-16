# Kubernetes

[Kubernetes](https://kubernetes.io/), also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.
In recent years, it has become the de-facto standard for cluster management in the cloud. We will be using kubernetes to manage our infrastructure and deploy
our applications. In this section of the tutorial, you will learn to deploy kubernetes to a create a cluster comprising of VMs that can be managed from a single point which is the master node.


## Installing Required Tools

Before installing Kubernetes, we need to install some tools that we will be using later on.

### Docker

You might need docker for local development of containers before deploying
them to Kubernetes. So, we recommend that you install Docker on the **master**
VM to have the tools available to you. **Notice** that seeing errors in this section (setting the permissions for Docker) is OK.

```sh
# install docker
curl -sSL https://get.docker.com | sh
# fix the user permissions for running docker seeing errors in this section is OK
sudo usermod -aG docker $USER
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "/home/$USER/.docker" -R
sudo chown "$USER":"$USER" /var/run/docker.sock
sudo chmod g+rwx /var/run/docker.sock -R
sudo systemctl enable docker
```

Test you docker installation by running the following:

```sh
(master) $ docker ps
(master) $ docker version
```

Which should yield a result like the following:

```console
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker version
Client: Docker Engine - Community
 Version:           24.0.7
 API version:       1.43
 Go version:        go1.20.10
 Git commit:        afdd53b
 Built:             Thu Oct 26 09:07:41 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.7
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.10
  Git commit:       311b9ff
  Built:            Thu Oct 26 09:07:41 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.27
  GitCommit:        a1496014c916f9e62104b33d1bb5bd03b0858e59
 runc:
  Version:          1.1.11
  GitCommit:        v1.1.11-0-g4bccb38
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

### Arkade and Other Tools

[Arkade](https://github.com/alexellis/arkade) Is a tool that can be used to simplify the
process of installing several applications and components necessary for interacting with
kubernetes clusters. You can learn more about Arkade by reading [this blog post](https://www.openfaas.com/blog/openfaas-arkade/).

Other tools we will be using for this installation are [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/), [helm](https://helm.sh/), and [k3sup](https://k3sup.dev/). `Kubectl` is the tool that we will
use to interact with our kubernetes cluster. `Helm` is the most used package manager for kubernetes, which
simplifies the process of installing and updating kubernetes applications. `K3sup` is a tool that simplifies
the process of setting up our kubernetes cluster with minimal effort.

To install the Arkade and some other tools that you might need for interacting with your cluster,
you can use the following installation scripts on the **master** VM:

```sh
# install arkade
curl -sLS https://get.arkade.dev | sudo sh
# Add tools bin directory to PATH
echo "export PATH=\$HOME/.arkade/bin:\$PATH" >> ~/.bashrc
# Copy bash completion script
arkade completion bash > ~/.arkade_bash_completion.sh
echo "source ~/.arkade_bash_completion.sh" >> ~/.bashrc
# Kubectl
arkade get kubectl
# Kubectl bash completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
source ~/.bashrc
# Helm
arkade get helm
# k3sup
arkade get k3sup
# move kubectl to /usr/bin to allow it to be used in python later on
sudo cp ~/.arkade/bin/kubectl /usr/bin/ && sudo chmod +x /usr/bin/kubectl
# Add helm and k3sup to your PATH variable
export PATH=$PATH:$HOME/.arkade/bin/

```

You can test your installation using following commands. **Notice** that the error shown for `kubectl version` is not important, as we have not yet installed our kubernetes cluster.

```console
$ arkade version
$ helm version
$ kubectl version
$ k3sup version


### Optional Tools

For special installations, you might need `skaffold` which can be installed using the following command. However, for most students this step is not required.

curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && sudo install skaffold /usr/local/bin/ && rm skaffold
```

## Installing Kubernetes

For this project, we will be using a lightweight distribution of kubernetes called [k3s](https://k3s.io/).
K3s is a lightweight distribution of Kubernetes created at [Rancher Labs](https://rancher.com/). Basically, it is a complete 
Kubernetes distribution, but they combined all processes into a single binary, added cross-compilation for ARM, dropped a lot of 
extra features that you don’t normally use and bundled some extra user-space tools. To install k3s on the master, run the
following commands on the master VM:

```sh
# run the following commands on your master VM
# create kubeconfig directory
mkdir ~/.kube
# master information
export MASTER_IP=192.168.0.102
# install k3s
k3sup install \
  --local \
  --k3s-channel stable \
  --local-path ~/.kube/config \
  --k3s-extra-args "--node-external-ip $MASTER_IP --node-ip $MASTER_IP --bind-address $MASTER_IP --disable traefik --write-kubeconfig-mode 644" \
  --print-command
```

If everything goes smoothly, you should see an output like the following:

```console
Running: k3sup install

[INFO]  Finding release for channel stable
[INFO]  Using v1.28.5+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.28.5+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.28.5+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, command exists in PATH at /home/eecs/.arkade/bin/kubectl
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, command exists in PATH at /usr/bin/ctr
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
stdout: "[INFO]  Finding release for channel stable\n[INFO]  Using v1.28.5+k3s1 as release\n[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.28.5+k3s1/sha256sum-amd64.txt\n[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.28.5+k3s1/k3s\n[INFO]  Verifying binary download\n[INFO]  Installing k3s to /usr/local/bin/k3s\n[INFO]  Skipping installation of SELinux RPM\n[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, command exists in PATH at /home/eecs/.arkade/bin/kubectl\n[INFO]  Creating /usr/local/bin/crictl symlink to k3s\n[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, command exists in PATH at /usr/bin/ctr\n[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh\n[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh\n[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env\n[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service\n[INFO]  systemd: Enabling k3s unit\n[INFO]  systemd: Starting k3s\n"Saving file to: /home/eecs/.kube/config

# Test your cluster with:
export KUBECONFIG=/home/eecs/.kube/config
kubectl config use-context default
kubectl get node -o wide
```

You can check out your installation using the following command:

```sh
kubectl get nodes -o wide
```

Which should yield an output like the following:

```console
$ kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE    VERSION        INTERNAL-IP     EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
server4   Ready    control-plane,master   100s   v1.28.5+k3s1   192.168.0.102   192.168.0.102   Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.11-k3s2
```

The status shown as `Ready` shows everything went smoothly and we have completely set up our master node.
Now, we need to have our other nodes (workers) join the kubernetes cluster. To do so, you need `k3sup` installed
on the cluster head.
<!-- To do so, download k3sup executable from its [release page](https://github.com/alexellis/k3sup/releases) 
for windows, or run the following for MacOS and Linux: -->

```sh
curl -sLS https://get.k3sup.dev | sudo sh
```

Next, you need to have the `worker node` join your kubernetes cluster. Run the following
commands on `Cluster head`.

```sh
# master information
export MASTER_IP=192.168.0.102
export MASTER_USER=eecs

# workers information
export WORKER_ONE_IP=192.168.0.101
export WORKER_TWO_IP=192.168.0.103
export WORKER_USER=eecs

# adding workers to the cluster
k3sup join --ip $WORKER_ONE_IP --user $WORKER_USER --server-ip $MASTER_IP --server-user $MASTER_USER --k3s-extra-args "--node-external-ip $WORKER_ONE_IP --node-ip $WORKER_ONE_IP" --k3s-channel stable --print-command
k3sup join --ip $WORKER_TWO_IP --user $WORKER_USER --server-ip $MASTER_IP --server-user $MASTER_USER --k3s-extra-args "--node-external-ip $WORKER_TWO_IP --node-ip $WORKER_TWO_IP" --k3s-channel stable --print-command
```

In case you did not include your private and public keys in the default path (`~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`), you need to specify them whenever you want to connect with your remote VM. In order to do so, you need to perform the **join** command in the following way, setting the `KEY_LOCATION` to the path to your private key:

```sh
# set your key location here
export WORKER_ONE_KEY_LOCATION="/PATH/TO/YOUR/PRIVATE_KEY"
export WORKER_TWO_KEY_LOCATION="/PATH/TO/YOUR/PRIVATE_KEY"

k3sup join --ip $WORKER_ONE_IP \
    --user $WORKER_USER \
    --server-ip $MASTER_IP \
    --server-user $MASTER_USER \
    --k3s-extra-args "--node-external-ip $WORKER_ONE_IP --node-ip $WORKER_ONE_IP" \
    --k3s-channel stable \
    --print-command \
    --ssh-key $WORKER_ONE_KEY_LOCATION

k3sup join --ip $WORKER_TWO_IP \
    --user $WORKER_USER \
    --server-ip $MASTER_IP \
    --server-user $MASTER_USER \
    --k3s-extra-args "--node-external-ip $WORKER_TWO_IP --node-ip $WORKER_TWO_IP" \
    --k3s-channel stable \
    --print-command \
    --ssh-key $WORKER_TWO_KEY_LOCATION
```

If everything goes as planned, you should see an output like the following:

```sh
$ k3sup join --ip $WORKER_TWO_IP --user $WORKER_USER --server-ip $MASTER_IP --server-user $MASTER_USER --k3s-extra-args "--node-external-ip $WORKER_TWO_IP --node-ip $WORKER_TWO_IP" --k3s-channel stable --print-command
Running: k3sup join
Joining 192.168.0.103 => 192.168.0.102
ssh: sudo cat /var/lib/rancher/k3s/server/node-token

Received node-token from 192.168.0.102.. ok.
ssh: curl -sfL https://get.k3s.io | K3S_URL='https://192.168.0.102:6443' K3S_TOKEN='K10b459aa5ddc586e8dee44fd36c7db845221a039d910dd8a78e95c7675ece216d7::server:fa481b833a500a586179ecfaa8009dd0' INSTALL_K3S_CHANNEL='stable' sh -s - --node-external-ip 192.168.0.103 --node-ip 192.168.0.103
[INFO]  Finding release for channel stable
[INFO]  Using v1.28.5+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.28.5+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.28.5+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent
Logs: Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
Output: [INFO]  Finding release for channel stable
[INFO]  Using v1.28.5+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.28.5+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.28.5+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
[INFO]  systemd: Starting k3s-agent
```

We should also be able to see the added worker nodes in the cluster by running the following on the `master`:

```sh
(master) $ kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE     VERSION        INTERNAL-IP     EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
server4   Ready    control-plane,master   8m39s   v1.28.5+k3s1   192.168.0.102   192.168.0.102   Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.11-k3s2
server3   Ready    <none>                 65s     v1.28.5+k3s1   192.168.0.101   192.168.0.101   Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.11-k3s2
server2   Ready    <none>                 3s      v1.28.5+k3s1   192.168.0.103   192.168.0.103   Ubuntu 22.04.3 LTS   5.15.0-91-generic   containerd://1.7.11-k3s2
```

Notice the `Ready` status for both VMs.

Now that our cluster is set up and ready to use, we can proceed to the [next step](03-microservice.md). You can check
out some of the most used `kubectl` commands on [their documentations](https://kubernetes.io/docs/reference/kubectl/overview/).
You can also take a look at the [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).

[Next Step](03-microservice.md) -->

## References

- [Medium: The Ultimate Guide to Building Your Personal K3S Cluster](https://itnext.io/the-ultimate-guide-to-building-your-personal-k3s-cluster-bf2643f31dd3)
- [Kubernetes basics tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)
- [kubectl documentation](https://kubernetes.io/docs/reference/kubectl/overview/)
- [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
