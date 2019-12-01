# Kubernetes on (vanilla) Raspbian Lite

Yes - you can create a Kubernetes cluster with Raspberry Pis with the default operating system called Raspbian. This means you can carry on using all the tools and packages you're used to with the officially-supported OS.

This is part of a blog post [Serverless Kubernetes home-lab with your Raspberry Pis
](https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/) written by [Alex Ellis](https://twitter.com/alexellisuk).

> Copyright disclaimer: Please provide a link to the post and give attribution to the author if you plan to use this content in your own materials.

## Update - k3s and `docker`

My current thinking is that [k3s](https://github.com/teamserverless/k8s-on-raspbian#pick-k3s) from Rancher Labs is a better option than `kubeadm` to bootstrap a cluster. Whilst both create a compliant Kubernetes cluster, k3s uses fewer resources, is faster and doesn't run into some of the timing issues we've seen in the community with `kubeadm`.

You should also see my note on [installing Docker on Raspbian Buster](https://github.com/teamserverless/k8s-on-raspbian#fix-docker-for-raspbian-buster-optional)

## Pre-reqs:

* To install and operate Kubernetes, you use only Raspberry Pi 3B, 3B+, or 4B
* I'm assuming you're using wired ethernet (Wi-Fi also works, but it's not recommended)

## Master node setup

You can either follow the steps below, or use my flashing script which automates the below. The automated flashing script must be run on a Linux computer with an SD card writer or an RPi.

### Flash with a Linux host

[Provision a Raspberry Pi SD card](https://gist.github.com/alexellis/a7b6c8499d9e598a285669596e9cdfa2)

Then run: 

```
curl -sLSf https://gist.githubusercontent.com/alexellis/fdbc90de7691a1b9edb545c17da2d975/raw/125ad6eae27e40a235412c2b623285a089a08721/prep.sh | sudo sh
```

### Continue to flash manually

* Flash Raspbian to a fresh SD card.

You can use [Etcher.io](https://etcher.io) to burn the SD card.

Before booting set up an empty file called `ssh` in /boot/ on the SD card.

Use Raspbian Stretch Lite

> Update: I previously recommended downloading Raspbian Jessie instead of Stretch. At time of writing (3 Jan 2018) Stretch is now fully compatible.

https://www.raspberrypi.org/downloads/raspbian/

* Change hostname

Use the `raspi-config` utility to change the hostname to k8s-master-1 or similar and then reboot.

* Set a static IP address

It's not fun when your cluster breaks because the IP of your master changed. The master's certificates will be bound to the IP address, so let's fix that problem ahead of time:

```
cat >> /etc/dhcpcd.conf
```

Paste this block:

```
profile static_eth0
static ip_address=192.168.0.100/24
static routers=192.168.0.1
static domain_name_servers=8.8.8.8
```

Hit Control + D.

Change 100 for 101, 102, 103 etc.

You may also need to make a reservation on your router's DHCP table so these addresses don't get given out to other devices on your network.

* Install Docker

This installs 17.12 or newer.

```
curl -sSL get.docker.com | sh

# Add current user to docker group:

sudo usermod pi -aG docker
# Refresh groups

newgrp docker
```

* Disable swap

For Kubernetes 1.7 and onwards you will get an error if swap space is enabled.

Turn off swap:

```
$ sudo dphys-swapfile swapoff && \
  sudo dphys-swapfile uninstall && \
  sudo update-rc.d dphys-swapfile remove
```

For Debian, also run:

```sh
sudo systemctl disable dphys-swapfile
```

This should now show no entries:

```
$ sudo swapon --summary
```

* Edit `/boot/cmdline.txt`

Add this text at the end of the line, but don't create any new lines:

```
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

Now reboot - do not skip this step.

* Add repo lists & install `kubeadm`

```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubeadm
```

> I realise this says 'xenial' in the apt listing, don't worry. It still works.

### Initialize your master node

* You now have two new commands installed:
 * kubeadm - used to create new clusters or join an existing one
 * kubectl - the CLI administration tool for Kubernetes
 
* Pre-pull images

`kubeadm` now has a command to pre-pull the requisites Docker images needed to run a Kubernetes master, type in:

```
$ sudo kubeadm config images pull -v3
```

If using Weave Net

* Initialize your master node:

```
$ sudo kubeadm init --token-ttl=0
```

If using Flannel:

* Initialize your master node with a Pod network CIDR:

```
$ sudo kubeadm init --token-ttl=0 --pod-network-cidr=10.244.0.0/16
```

We pass in `--token-ttl=0` so that the token never expires - do not use this setting in production. The UX for `kubeadm` means it's currently very hard to get a join token later on after the initial token has expired. 

> Optionally also pass `--apiserver-advertise-address=192.168.0.27` with the IP of the Pi as found by typing `ifconfig`.

Note: This step can take a long time, even up to 15 minutes.

Sometimes this stage can fail, if it does then you should patch the API Server to allow for a higher failure threshold during initialization around the time you see `[controlplane] wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"`

```
sudo sed -i 's/failureThreshold: 8/failureThreshold: 20/g' /etc/kubernetes/manifests/kube-apiserver.yaml && \
sudo sed -i 's/initialDelaySeconds: [0-9]\+/initialDelaySeconds: 360/' /etc/kubernetes/manifests/kube-apiserver.yaml
```

After the `init` is complete run the snippet given to you on the command-line:

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

This step takes the key generated for cluster administration and makes it available in a default location for use with `kubectl`.

* Now save your join-token

Your join token is valid for 24 hours, so save it into a text file. Here's an example of mine:

```
$ kubeadm join --token 9e700f.7dc97f5e3a45c9e5 192.168.0.27:6443 --discovery-token-ca-cert-hash sha256:95cbb9ee5536aa61ec0239d6edd8598af68758308d0a0425848ae1af28859bea
```

* Check everything worked:

```
$ kubectl get pods --namespace=kube-system
NAME                           READY     STATUS    RESTARTS   AGE                
etcd-of-2                      1/1       Running   0          12m                
kube-apiserver-of-2            1/1       Running   2          12m                
kube-controller-manager-of-2   1/1       Running   1          11m                
kube-dns-66ffd5c588-d8292      3/3       Running   0          11m                
kube-proxy-xcj5h               1/1       Running   0          11m                
kube-scheduler-of-2            1/1       Running   0          11m                
weave-net-zz9rz                2/2       Running   0          5m 
```

You should see the "READY" count showing as 1/1 for all services as above. DNS uses three pods, so you'll see 3/3 for that.

### Setup networking with Weave Net or Flannel

Some users have reported stability issues with Weave Net on ARMHF. These issues do not appear to affect x86_64 (regular PCs/VMs). You may want to try Flannel instead of Weave Net for your RPi cluster.

#### Weave Net

Install [Weave Net](https://www.weave.works/oss/net/) network driver

```
$ kubectl apply -f \
 "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

If you run into any issues with Weaveworks' networking then [flannel](https://github.com/coreos/flannel) is also a popular choice for the ARM platform.

#### Flannel (alternative)

Apply the Flannel driver on the master:

```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

On each node that joins including the master:

```
$ sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

### Join other nodes

On the other RPis, repeat everything apart from `kubeadm init`.

* Change hostname

Use the `raspi-config` utility to change the hostname to `k8s-worker-1` or similar and then reboot.

* Join the cluster

Replace the token / IP for the output you got from the master node, for example:

```
$ sudo kubeadm join --token 1fd0d8.67e7083ed7ec08f3 192.168.0.27:6443
```

You can now run this on the master:

```
$ kubectl get nodes
NAME      STATUS     AGE       VERSION
k8s-1     Ready      5m        v1.7.4
k8s-2     Ready      10m       v1.7.4
```

## Deploy a container

This container will expose a HTTP port and convert Markdown to HTML. Just post a body to it via `curl` - follow the instructions below. 

*function.yml*

```yaml
apiVersion: v1
kind: Service
metadata:
  name: markdownrender
  labels:
    app: markdownrender
spec:
  type: NodePort
  ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
      nodePort: 31118
  selector:
    app: markdownrender
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: markdownrender
  labels:
   app: markdownrender
spec:
  replicas: 1
  selector:
    matchLabels:
      app: markdownrender
  template:
    metadata:
      labels:
        app: markdownrender
    spec:
      containers:
      - name: markdownrender
        image: functions/markdownrender:latest-armhf
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          protocol: TCP
```

Deploy and test:

```
$ kubectl create -f function.yml
```

Once the Docker image has been pulled from the hub and the Pod is running you can access it via `curl`: 

```
$ curl -4 http://127.0.0.1:31118 -d "# test"
<p><h1>test</h1></p>
```

If you want to call the service from a remote machine such as your laptop then use the IP address of your Kubernetes master node and try the same again.

## Start up the Kubernetes dashboard

The dashboard can be useful for visualising the state and health of your system, but it does require the equivalent of "root" in the cluster. If you want to proceed you should first run in a [ClusterRole from the docs](https://github.com/kubernetes/dashboard/wiki/Access-control#admin-privileges).

```
echo -n 'apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-head
  labels:
    k8s-app: kubernetes-dashboard-head
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard-head
  namespace: kube-system' | kubectl apply -f -
```

This is the development/alternative dashboard which has TLS disabled and is easier to use.

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/alternative/kubernetes-dashboard-arm-head.yaml
```

You can then find the IP and port via `kubectl get svc -n kube-system`. To access this from your laptop you will need to use `kubectl proxy` and navigate to `http://localhost:8001/` on the master, or tunnel to this address with `ssh`.

See also: [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) docs.

## Remove the test deployment

Now on the Kubernetes master remove the test deployment:

```
$ kubectl delete -f function.yml
```

### Wrapping up

You should now have an operational Kubernetes master and several worker nodes ready to accept workloads.

Now let's head back [over to the tutorial and deploy OpenFaaS](https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/) to put the cluster through its paces with Serverless functions.

See also: [Kubernetes documentation](https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational)
