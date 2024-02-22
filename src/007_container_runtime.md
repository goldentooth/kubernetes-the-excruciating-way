# Container Runtime

Kubernetes is a container orchestration platform and therefore requires some container runtime to be installed.

This is a simple step; [`containerd`](https://containerd.io) is well-supported, well-regarded, and I don't have any reason _not_ to use it.

I used Jeff Geerling's [Ansible role](https://github.com/geerlingguy/ansible-role-containerd) to install and configure `containerd` on my cluster; this is really the point at which some kind of IaC/configuration management system becomes something more than a polite suggestion 🙂

That said, the actual steps are not very demanding (aside from the fact that they will need to be executed once on each Kubernetes host). They intersect largely with [Docker Engine's installation instructions](https://docs.docker.com/engine/install/debian/) (since Docker, not the Containerd project, maintains the package repository), which I won't repeat here.

The container runtime installation is handled in my [`deploy_kubernetes.yaml` playbook](https://github.com/goldentooth/cluster/blob/main/playbooks/deploy_kubernetes.yaml), which is where we'll be spending a lot of time in subsequent sections.
