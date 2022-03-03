# VirtualBox images for development based on linux distributions

This project started as a side track of [WSL images](https://github.com/arundeep78/wsl_debian_dev). I did not think I would need it as I started with WSL, it seems it was the easier, light weight and happily integrated linux development environment when you main OS is windows. Ofcourse, if one is running primarily on Linux, then it is a different story all together.

## Motivation

I started with WSL images in order to learn Kubernetes, docker and development that happens using those environment. Mainly my target was
1. Apache Airflow
2. Python development
3. Flask/fastAPI based API development
4. Database setup 
5. Somewhat of Web development simply to expose data that is being processed using above. It could some JS in Flask e.g.

It actually did work 
1. For python development
2. Jupyter notebooks dev environment worked with and without VS code.
3. to have unix based utilities and interface available to connect to remote system e.g. kubectl, helm, Azure etc
4. VS code integration with WSL and remote containers worked nicely.
5. I could learn basics of devcontainers using Docker and VS Code.


However, it started to fail when I tried to follow steps to configure Kubernetes.
1. [Kind setup failed](https://github.com/arundeep78/wsl_debian_dev/blob/master/Kind_k8s_Readme.md) with docker.io connection issues
2. [Similary Minikube setup failed](https://github.com/arundeep78/wsl_debian_dev/blob/master/minikube_k8s_Readme.md)
3. [Microk8s failed right away](https://github.com/arundeep78/wsl_debian_dev/blob/master/microk8s_readme.md) as WSL does not support SNAP because it depends on systemd architecture.


So, I decided to try a pure virtual machine environment rather than WSL.

## Why Virtual Box 

Very simple, it is free and I knew it from before. Well, atleast I have heard about it. There can be another big debate about which Virtualization toolbox is better and many threads would point out to [VMware](https://www.vmware.com/) or [Parallels](https://www.parallels.com/eu/). But, I just selected [VirtualBox](https://www.virtualbox.org/) for familiarity, price and [this article that I came across while trying to find solutions for MicroK8s problem](https://praveenmak.medium.com/why-i-switched-from-wsl-to-virtualbox-6c464d51af3).


## Install Virtual Box on Windows 11.

This is a simple and straight forward procedure to follow.

1. Download and install latest [VirtualBox](https://www.virtualbox.org/) and install it.
2. If you like to read [documentation first then there is a huge page](https://www.virtualbox.org/manual/UserManual.html) for it with lots of details that I did not go through :) 


**NOTE** : VirtualBox6.2.1 does not support ClearLinux, even though [ClearLinux documentation says it does](https://docs.01.org/clearlinux/latest/get-started/virtual-machine-install/virtualbox-cl-installer.html). It gives and error about `Carryless-Multiplication` and as [per github issue](https://github.com/clearlinux/distribution/issues/2161), they won't fix it, but rather would like VirtualBox to change their setup.


## various VirtualBox images

Similar to WSL project, I would try to make layers of images to move horizontal or verticle for image selection.



1. [Base VirtualBox image with Debian 11](base_debian_readme.md).