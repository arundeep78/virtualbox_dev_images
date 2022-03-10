# VirtualBox Debian image with Microk8s installation

This documentation is to have configure a Debian 11 image in VirtualBox with microk8s installed on the image. This is based on the [standard Debian 11 image configured with basic tools like Oh-my-ZSH etc](base_debian_readme.md).

I decided to create a separate image simply not to have any conflict with other flavours of K8s. Microk8s uses snap package and I read somewhere that one of the flavours(I think it was Kind) does not like snap. Maybe those issues are solved, but maybe it is better to keep them isolated lets continue to have a isolation. Afterall all this containerization is about isolation, right?


## Why MicroK8s?

The documentation says that it has a very low memory footprint. This is cool as it should help keep some resources free on my laptop. 

It was also the image mentioned in one of the [Microsoft Tutorails](https://docs.microsoft.com/en-us/learn/modules/intro-to-kubernetes/) that I followed to learn K8s. 

Additionally, it does not seems to need docker to run it. At the moment, I have [issues with Kind and Minikube clusters in WSL](https://github.com/arundeep78/wsl_debian_dev/blob/master/Kind_k8s_Readme.md#check-internet-access-from-nodes) as clusters nodes cannot reach docker repository, even though all other internet sites are accessible! So, it might help me to get going with actual cluster tasks rather than figuring out why I cannot reach docker repository.

## Create a new Virtual Box machine

Once can prepare a new VM from scratch. But in this case, I will simply clone the base image using VirtualBox tools. This image has all the basic tools needed for MicroK8s.

## Install and configure MicroK8s - attempt 1

### install mikrok8s

Follow [official documentation](https://microk8s.io/?ref=thechiefio) for basic installtion steps.

```zsh
$ sudo snap install microk8s --classic

microk8s (1.23/stable) v1.23.3 from Canonical✓ installed
```

### Check microK8s status

```zsh
$ microk8s status --wait-ready

zsh: command not found: microk8s
```

Now now, what a surprise. It might feel like that I am somehow finding all the wrong configuration to get even the basics working!! This SNAP package does say it is installed!!

- I logged out and logged in again, same result!
- All snap commands wwere hanging e.g. `snap list`, `snap version`
- Tried to shutdown VM. Did not work.
- Power Off the VM worked
- `snap version` gives below information
  ```zsh
  $ snap version
  snap    2.54.3.2
  snapd   unavailable
  series  -
  ```
- For some reason snapd does not start anymore
  ```zsh
  systemctl status snapd
  snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; enabled; vendor preset: enabled)
     Active: activating (start) since Mon 2022-03-07 12:08:22 CET; 53s ago
    TriggeredBy: ● snapd.socket
    Main PID: 4002 (snapd)
      Tasks: 7 (limit: 2341)
     Memory: 15.6M
        CPU: 49ms
     CGroup: /system.slice/snapd.service
             └─4002 /usr/lib/snapd/snapd
  ```
- A restart of the service also fails
  ```zsh
  $ sudo systemctl restart snapd
  Job for snapd.service failed because a timeout was exceeded.
  See "systemctl status snapd.service" and "journalctl -xe" for details.
  ```
- Not sure what went wrong. The clone that I created from the base image has snapd running.
- maybe easier to restart rather than investigating

## Install and configure microk8s - attempt 2

### Create new VB image from the base image

Create a clone, of if you are familiar with snapshots and have used those, should work as well.

### check status of snapd first

```zsh
$ sudo systemctl status snapd
  snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-03-07 14:01:57 CET; 4min 1s ago
  TriggeredBy: ● snapd.socket
   Main PID: 361 (snapd)
      Tasks: 8 (limit: 2341)
     Memory: 81.9M
        CPU: 1.259s
     CGroup: /system.slice/snapd.service
             └─361 /usr/lib/snapd/snapd
  ```

### Install microk8s

[Same procedure as attempt 1](#install-microk8s)

```zsh
$ sudo snap install microk8s --classic

microk8s (1.23/stable) v1.23.3 from Canonical✓ installed
```

### Same issue

However, noticed that immediately after snapd installation completed ( took about 2 minutes), `snapd list` command worked however very slow (took 30s). I tried it again after 10 seconds and it failed as before
```zsh
$ sudo snap list

error: cannot list snaps: cannot communicate with server: timeout exceeded while waiting for response
```

Not sure, if some microk8s services changes some network profiles as they start and thus causing snapd to fail. But, even bigger problem is that even when snapd gives a successfull installation message, `microk8s` command is not found in the system, even when snapd seems to have downloaded all files.

## Install and configure microk8s - attempt 3

### Create new VB image from the base image

Create a clone, of if you are familiar with snapshots and have used those, should work as well.

I realized that I have VM with 2GB of RAM and 8GB VHDD and 1 vcpu.
I changed the configuration to
- 6GB RAM
- 2 vcpu


### check status of snapd first

```zsh
$ sudo systemctl status snapd
  snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-03-07 14:01:57 CET; 4min 1s ago
  TriggeredBy: ● snapd.socket
   Main PID: 361 (snapd)
      Tasks: 8 (limit: 2341)
     Memory: 81.9M
        CPU: 1.259s
     CGroup: /system.slice/snapd.service
             └─361 /usr/lib/snapd/snapd
```

```zsh
  $ sudo snap list
Name  Version    Rev    Tracking       Publisher   Notes
core  16-2.54.3  12725  latest/stable  canonical✓  core
```

### Install microk8s

[Same procedure as attempt 1](#install-microk8s)

 Changed the version to old ones as [some of the github issues](https://github.com/canonical/microk8s/issues/2912) mentioned that old version worked. But not in this case.
```zsh
$ sudo snap install microk8s --classic --channel=1.21/stable

microk8s (1.23/stable) v1.23.3 from Canonical✓ installed
```

```zsh
$ snap list
Name      Version    Rev    Tracking       Publisher   Notes
core      16-2.54.3  12725  latest/stable  canonical✓  core
core18    20211215   2284   latest/stable  canonical✓  base
microk8s  v1.21.10   3005   1.21/stable    canonical✓  classic
```

**New error**

message from systemlogd : kernel watchdog: BUG: soft lockup - CPU#0 stuck for 22s! [kubectl:3057]

This seems to be related to high load on VM. Might be virtual box specific. I restarted the VM and checked again

- I changed shell from ZSH to BASH, `microk8s` exits.
- Came back to ZSH, `microk8s` exists now here too. pure magic!
- `microk8s stop` worked.
- `microk8s start` worked and started trace. Found errors related to networking
  ```zsh
  $ sudo journalctl -f -u snap.microk8s.daemon-kubelite

  Mar 08 00:29:37 deb11mk8s microk8s.daemon-kubelite[24749]: I0308 00:29:37.491571   24749 eviction_manager.go:390] "Eviction manager: unable to evict any pods from the node"
  Mar 08 00:29:40 deb11mk8s microk8s.daemon-kubelite[24749]: E0308 00:29:40.068702   24749 pod_workers.go:918] "Error syncing pod, skipping" err="failed to \"StartContainer\" for \"install-cni\" with CrashLoopBackOff: \"back-off 2m40s restarting failed container=install-cni pod=calico-node-29x5m_kube-system(144b3044-4f15-46ba-93bf-305be42d810e)\"" pod="kube-system/calico-node-29x5m" podUID=144b3044-4f15-46ba-93bf-305be42d810e
  Mar 08 00:29:42 deb11mk8s microk8s.daemon-kubelite[24749]: E0308 00:29:42.150703   24749 kubelet.go:2347] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
  Mar 08 00:29:47 deb11mk8s microk8s.daemon-kubelite[24749]: E0308 00:29:47.157809   24749 kubelet.go:2347] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
  Mar 08 00:29:47 deb11mk8s microk8s.daemon-kubelite[24749]: I0308 00:29:47.496864   24749 eviction_manager.go:338] "Eviction manager: attempting to reclaim" resourceName="ephemeral-storage"
  Mar 08 00:29:47 deb11mk8s microk8s.daemon-kubelite[24749]: I0308 00:29:47.496964   24749 container_gc.go:85] "Attempting to delete unused containers"
  Mar 08 00:29:47 deb11mk8s microk8s.daemon-kubelite[24749]: I0308 00:29:47.503240   24749 image_gc_manager.go:327] "Attempting to delete unused images"
  Mar 08 00:29:47 deb11mk8s microk8s.daemon-kubelite[24749]: I0308 00:29:47.506265   24749 eviction_manager.go:349] "Eviction manager: must evict pod(s) to reclaim" resourceName="ephemeral-storage"
  Mar 08 00:29:47 deb11mk8s microk8s.daemon-kubelite[24749]: I0308 00:29:47.508884   24749 eviction_manager.go:367] "Eviction manager: pods ranked for eviction" pods=[kube-system/calico-node-29x5m]
  Mar 08 00:29:47 deb11mk8s microk8s.daemon-kubelite[24749]: E0308 00:29:47.510555   24749 eviction_manager.go:560] "Eviction manager: cannot evict a critical pod" pod="kube-system/calico-node-29x5m"
  ```
- Remove microk8s
  ```zsh
  $ sudo snap remove microk8s
  error: cannot communicate with server: timeout exceeded while waiting for response
  ```
- seems like `snap` is again stuck
  ```zsh
  sudo systemctl status snapd
  snapd.service - Snap Daemon
     Loaded: loaded (/lib/systemd/system/snapd.service; enabled; vendor preset: enabled)
     Active: deactivating (stop-sigterm) (Result: timeout) since Tue 2022-03-08 00:37:36 CET; 5min ago
  TriggeredBy: ● snapd.socket
    Main PID: 26366 (snapd)
        Tasks: 9 (limit: 4679)
      Memory: 23.6M
          CPU: 113ms
      CGroup: /system.slice/snapd.service
              └─26366 /usr/lib/snapd/snapd

  Mar 08 00:40:37 deb11mk8s systemd[1]: Starting Snap Daemon...
  Mar 08 00:40:37 deb11mk8s snapd[26366]: AppArmor status: apparmor is enabled but some kernel features are missing: dbus, network
  Mar 08 00:40:37 deb11mk8s snapd[26366]: AppArmor status: apparmor is enabled but some kernel features are missing: dbus, network
  Mar 08 00:42:07 deb11mk8s systemd[1]: snapd.service: start operation timed out. Terminating.
  ```

## Microk8s installation fails on Debian 11 on Virtual BOX

As of march 8th, 2022, I cannot install microk8s on Debian11 in VirtualBox VM on Windows 11 host.

- Base VM works fine.
- snap install microk8s finishes successfully
- But `microk8s` command does not added to path, even if the program is downloaded
- snap fails to start working. Restarting VM or service does not help.

# To be picked up other time - research and reading continues.

maybe someone has an idea and can leave a comment for solution!!!
