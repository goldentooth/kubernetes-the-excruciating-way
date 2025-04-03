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

