## Kubernetes on Raspbian

This repository holds the original tutorial for "Kubernetes on Raspbian" / Raspberry Pi by Alex Ellis

![](https://pbs.twimg.com/media/DKGfQ7bWkAAkGb9.jpg)

This guide is part of a larger blog post: [Build your own bare-metal ARM cluster](https://blog.alexellis.io/build-your-own-bare-metal-arm-cluster/).

### Start the guide

Pick between `k3s` or `kubeadm`.

#### Pick `k3s`

My current recommendation is to use [k3s](https://k3s.io) from Rancher Labs.

It's:

* faster, uses less resources
* well-maintained and ARMHF / ARM64 just works
* still upstream / compliant Kubernetes
* doesn't appear to run the complicated issues seen with `kubeadm`

[Will it cluster? k3s on your Raspberry Pi](https://blog.alexellis.io/test-drive-k3s-on-raspberry-pi/)

#### Pick `kubeadm`

[Kubernetes on (vanilla) Raspbian Lite](./GUIDE.md)

Once you're up and running please share your clusters on Twitter with [@alexellisuk](https://twitter.com/alexellisuk).

You can also join the OpenFaaS Slack community's dedicated channel for ARM and Raspberry Pi #arm-and-pi. Just email alex at openfaas dot com for your invitation.

### Attribution

You're welcome to make use of this guide and to refer to it, but please do not copy it or pass it off as your own without giving attribution to the author(s). If you have suggestions or have found that some of the instructions have fallen out of date, then please see the Contributions section below on how to contribute.

#### Reader's clusters

* [roncrivera's 4-node home-lab running OpenFaaS in a gun-case](https://twitter.com/roncrivera/status/1078552483029381121)
* [Scott Hanselman's 6-node cluster running Kubernetes, OpenFaaS with the Pimoroni blinkt!](https://twitter.com/shanselman/status/953716434458247168) [Scott's cluster-selfie](https://twitter.com/alexellisuk/status/955568790061936640)
* [Ken Fukuyama's Kubernetes cluster running OpenFaaS](https://twitter.com/kenfdev/status/954748775678976000) from Japan
* [Karol Stępniewski's Asus Tinkerboard cluster running OpenFaaS and K8s](https://twitter.com/kars7e/status/948122096969818113)
* [Bart Plasmeijer's K8s cluster](https://twitter.com/bartplasmeijer/status/933778520500731904)
* [Burton Rheutan's cluster stashed away in a closet](https://twitter.com/_burtonr/status/1033745565379641344)
* [Kevin Turcios' OpenFaaS cluster](https://twitter.com/kjturcios/status/1071253482441715713)
* [Estelle Auberix' OpenFaaS and K8s cluster for ServerlessConf Paris](https://twitter.com/chussenot/status/960849791776419840)
* [Davy's Kubernetes and OpenFaaS cluster](https://twitter.com/realDavyHua/status/1028862482259931137)
* [Jaigouk Kim's OpenFaaS and K8s cluster with the Asus Tinkerboard](https://twitter.com/jaigouk/status/964529214564298756)
* [Ram's 7-node homelab with OpenFaaS and Kubernetes](https://twitter.com/rprakashg/status/947347563912470528)
* [David Muckle's OpenFaaS cluster](https://twitter.com/dvdmuckle/status/977297461210484737)
* [Brian Moelk's battery-powered OpenFaaS cluster](https://twitter.com/brianmoelk/status/954459005149175809)
* [Marcus Smallman' DIY Raspberry Pi Kubernetes Cluster](https://marcussmallman.io/2018/02/18/diy-rasberry-pi-kubernetes-cluster/)
* [Mathias Deremer-Accettone's Serverless sur Raspberry PI avec Docker Swarm et OpenFaas](https://blog.ineat-conseil.fr/2019/01/serverless-sur-raspberry-pi-avec-docker-swarm-et-openfaas-partie-1-installation-dopenfaas/)
* [Daniel Llewellyn's three node Raspberry Pi Swarm](https://twitter.com/diddledan/status/1088759711745351682)
* [Gareth Bradley's 6 node Raspberry Pi Kubernetes Cluster](https://garfbradaz.github.io/blog/2019/02/12/RaspberryPi-Cluster-Kubernetes.html)

Submit your cluster and description by creating a GitHub issue.

#### Adaptations / derived works

* ["Kubernetes on Raspbian" with Ansible](https://rak8s.io) by Chris Short
* ["Kubernetes on Raspbian" for .NET Core / Windows developers](https://www.hanselman.com/blog/HowToBuildAKubernetesClusterWithARMRaspberryPiThenRunNETCoreOnOpenFaas.aspx) by Scott Hanselman
* ["Kubernetes on Raspbian" broken-up into bash scripts with a custom laser-cut design](https://kubecloud.io/setup-a-kubernetes-1-9-0-raspberry-pi-cluster-on-raspbian-using-kubeadm-f8b3b85bc2d1) by Kasper Nissen
* ["Kubernetes on Raspbian" with Packer](https://blog.codybunch.com/2018/01/05/OpenFaaS-on-Kubernetes-on-Raspberry-Pi/) by Cody Bunch
* ["Kubernetes on Raspbian" adapted for HypriotOS](https://gist.github.com/elafargue/a822458ab1fe7849eff0a47bb512546f) by  Edouard Lafargue

### Contributions

Contributions are welcome, but must be tested and justified.

Please make sure each commit is signed off with `git commit -s` (this means don't edit in the GitHub UI). 

See below for more information.

```
Developer Certificate of Origin
Version 1.1

Copyright (C) 2004, 2006 The Linux Foundation and its contributors.
1 Letterman Drive
Suite D4700
San Francisco, CA, 94129

Everyone is permitted to copy and distribute verbatim copies of this
license document, but changing it is not allowed.


Developer's Certificate of Origin 1.1

By making a contribution to this project, I certify that:

(a) The contribution was created in whole or in part by me and I
    have the right to submit it under the open source license
    indicated in the file; or

(b) The contribution is based upon previous work that, to the best
    of my knowledge, is covered under an appropriate open source
    license and I have the right under that license to submit that
    work with modifications, whether created in whole or in part
    by me, under the same open source license (unless I am
    permitted to submit under a different license), as indicated
    in the file; or

(c) The contribution was provided directly to me by some other
    person who certified (a), (b) or (c) and I have not modified
    it.

(d) I understand and agree that this project and the contribution
    are public and that a record of the contribution (including all
    personal information I submit with it, including my sign-off) is
    maintained indefinitely and may be redistributed consistent with
    this project or the open source license(s) involved.
```
