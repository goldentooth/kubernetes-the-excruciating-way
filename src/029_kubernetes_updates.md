# Kubernetes Updates

Because I'm not a particularly smart man, I've allowed my cluster to fall behind. The current version, as of today, is 1.32.3, and my cluster is on 1.29.something.

So that means I need to upgrade 1.29 -> 1.30, 1.30 -> 1.31, and 1.31 -> 1.32.

## 1.29 -> 1.30

First, I update the repo URL in `/etc/apt/sources.list.d/kubernetes.sources` and run:

```bash
$ sudo apt update
Hit:1 http://deb.debian.org/debian bookworm InRelease
Hit:2 http://deb.debian.org/debian-security bookworm-security InRelease
Hit:3 http://deb.debian.org/debian bookworm-updates InRelease
Hit:4 https://download.docker.com/linux/debian bookworm InRelease
Hit:6 http://archive.raspberrypi.com/debian bookworm InRelease
Hit:7 https://baltocdn.com/helm/stable/debian all InRelease
Get:5 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease [1,189 B]
Err:5 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
  The following signatures were invalid: EXPKEYSIG 234654DA9A296436 isv:kubernetes OBS Project <isv:kubernetes@build.opensuse.org>
Reading package lists... Done
W: https://download.docker.com/linux/debian/dists/bookworm/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
W: GPG error: https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease: The following signatures were invalid: EXPKEYSIG 234654DA9A296436 isv:kubernetes OBS Project <isv:kubernetes@build.opensuse.org>
E: The repository 'https://pkgs.k8s.io/core:/stable:/v1.30/deb  InRelease' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```

Well, shit. Looks like I need to do some surgery elsewhere.

Fortunately, I had some code for setting up the Kubernetes package repositories in `install_k8s_packages`. Of course, I don't want to install new versions of the packages – the upgrade process is a little more delicate than that – so I factored it out into a new role called `setup_k8s_apt`. Running that role against my cluster with `goldentooth setup_k8s_apt` made the necessary changes.

```bash
$ sudo apt-cache madison kubeadm
   kubeadm | 1.30.11-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.10-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.9-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.8-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.7-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.6-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.5-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
   kubeadm | 1.30.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
```

There we go. That wasn't that bad.

Now, the next steps are things I'm going to do repeatedly, and I don't want to type a bunch of commands, so I'm going to do it in Ansible. I need to do that advisedly, though.

I created a new role, `goldentooth.upgrade_k8s`. I'm working through the [upgrade documentation](https://v1-30.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/), Ansible-izing it as I go.

So I added some tasks to update the Apt cache, unhold kubeadm, upgrade it, and then hold it again (via a handler). I tagged these with `first_control_plane` and invoke the role dynamically (because that is the only context in which you can limit execution of a role to the specified tags).

```bash
$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"30", GitVersion:"v1.30.11", GitCommit:"6a074997c960757de911780f250ecd9931917366", GitTreeState:"clean", BuildDate:"2025-03-11T19:56:25Z", GoVersion:"go1.23.6", Compiler:"gc", Platform:"linux/arm64"}
```

It worked!

The plan operation similarly looks fine.

```bash
$ sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.29.6
[upgrade/versions] kubeadm version: v1.30.11
I0403 11:18:34.338987  564280 version.go:256] remote version is much newer: v1.32.3; falling back to: stable-1.30
[upgrade/versions] Target version: v1.30.11
[upgrade/versions] Latest version in the v1.29 series: v1.29.15

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE        CURRENT   TARGET
kubelet     bettley     v1.29.2   v1.29.15
kubelet     cargyll     v1.29.2   v1.29.15
kubelet     dalt        v1.29.2   v1.29.15
kubelet     erenford    v1.29.2   v1.29.15
kubelet     fenn        v1.29.2   v1.29.15
kubelet     gardener    v1.29.2   v1.29.15
kubelet     harlton     v1.29.2   v1.29.15
kubelet     inchfield   v1.29.2   v1.29.15
kubelet     jast        v1.29.2   v1.29.15

Upgrade to the latest version in the v1.29 series:

COMPONENT                 NODE      CURRENT    TARGET
kube-apiserver            bettley   v1.29.6    v1.29.15
kube-apiserver            cargyll   v1.29.6    v1.29.15
kube-apiserver            dalt      v1.29.6    v1.29.15
kube-controller-manager   bettley   v1.29.6    v1.29.15
kube-controller-manager   cargyll   v1.29.6    v1.29.15
kube-controller-manager   dalt      v1.29.6    v1.29.15
kube-scheduler            bettley   v1.29.6    v1.29.15
kube-scheduler            cargyll   v1.29.6    v1.29.15
kube-scheduler            dalt      v1.29.6    v1.29.15
kube-proxy                          1.29.6     v1.29.15
CoreDNS                             v1.11.1    v1.11.3
etcd                      bettley   3.5.10-0   3.5.15-0
etcd                      cargyll   3.5.10-0   3.5.15-0
etcd                      dalt      3.5.10-0   3.5.15-0

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.29.15

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE        CURRENT   TARGET
kubelet     bettley     v1.29.2   v1.30.11
kubelet     cargyll     v1.29.2   v1.30.11
kubelet     dalt        v1.29.2   v1.30.11
kubelet     erenford    v1.29.2   v1.30.11
kubelet     fenn        v1.29.2   v1.30.11
kubelet     gardener    v1.29.2   v1.30.11
kubelet     harlton     v1.29.2   v1.30.11
kubelet     inchfield   v1.29.2   v1.30.11
kubelet     jast        v1.29.2   v1.30.11

Upgrade to the latest stable version:

COMPONENT                 NODE      CURRENT    TARGET
kube-apiserver            bettley   v1.29.6    v1.30.11
kube-apiserver            cargyll   v1.29.6    v1.30.11
kube-apiserver            dalt      v1.29.6    v1.30.11
kube-controller-manager   bettley   v1.29.6    v1.30.11
kube-controller-manager   cargyll   v1.29.6    v1.30.11
kube-controller-manager   dalt      v1.29.6    v1.30.11
kube-scheduler            bettley   v1.29.6    v1.30.11
kube-scheduler            cargyll   v1.29.6    v1.30.11
kube-scheduler            dalt      v1.29.6    v1.30.11
kube-proxy                          1.29.6     v1.30.11
CoreDNS                             v1.11.1    v1.11.3
etcd                      bettley   3.5.10-0   3.5.15-0
etcd                      cargyll   3.5.10-0   3.5.15-0
etcd                      dalt      3.5.10-0   3.5.15-0

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.30.11

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```


