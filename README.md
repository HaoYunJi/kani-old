## Kani - Rapid deploy Kubernetes v1.22.1 and elements by Ansible

### Content:

- Purpose

- Environment

- Files to be changed

- Issues

- Usage

- References

### Purpose:

- Rapid deploy kubernetes cluster with single master node.  

- Use `containerd` runtime and `Calico` CNI on Kubernetes cluster.

- (Optional) Deploy `Red Hat Quay v3 registry` as container images registry.

### Environment:

- OS version: `CentOS Linux release 7.9.2009` (Core)

- kernel version: `5.13.12`

> 📌 Linux kernel updated by elrepo repository.

- containerd version: `1.5.5`

- kubernetes version: `v1.22.1`

- Prepare one node as ansible control node and install `ansible`, `rhel-system-roles` rpm packages.

- Download the project directory and store it by regular user like `godev` user in vars/all.yml.

### Files to be changed:

- Change arguements in following files according to your environment:
  
  - `./ansible.cfg` (Ansible config file):
    
    - remote_user
  
  - `./files/kubeadm-conf.yml` (kubeadm config file):
    
    - localAPIEndpoint.advertiseAddress
    
    - nodeRegistration.criSocket
    
    - nodeRegistration.taints
    
    - `kube-proxy mode`
    
    - imageRepository
    
    - KubernetesVersion
    
    - `networking.podSubnet`
    
    - cgroupDriver
  
  - `./kubecluster` (Ansible inventory file):
    
    - change group name and short hostname
  
  - `./vars/all.yml` (variables file):
    
    > Arguments need to be edited has been noted by 'NEED TO EDIT'. According to your environment to change them.
    
    - gateway
    
    - nic
    
    - operator_user
    
    > 👉 operator_user same as remote_user of ansible.cfg
    
    - kube_master_node
    
    > 💥 Must change the arguement with the same as the master node short hostname in your inventory.

### Issues:

- Reported errors during deploy kubernetes cluster like followings:
  
  kubeamd init master node failure, and report `node not found`. Because all `pause` infra images haven't been tagged `k8s.io/pause:3.5`, `kubelet` on master node can't create `sandbox`.
  
  ![](https://github.com/Alberthua-Perl/tech-docs/blob/master/images/rapid-kube-deploy/kubeadm-init-master-error-1.jpg)
  
  ![](https://github.com/Alberthua-Perl/tech-docs/blob/master/images/rapid-kube-deploy/kubeadm-init-master-error-2.jpg)

- `register` variable in playbook can't be use between different plays.
  
  ![](https://github.com/Alberthua-Perl/tech-docs/blob/master/images/rapid-kube-deploy/register-var-used-between-two-plays-error.jpg)

- As to use `containerd` to run container, so `ctr`, `crictl` or `nerdctl` can be used to get container images or containers itself status. But just `crictl` use containerd config file `/etc/containerd/config.toml` directly. crictl load the config file inserted quay registry endpoint, user and password.

- If use ca certification file, crictl can't pull image from quay registry,
  
  and always report `x509 cert file error`:
  
  ![](https://github.com/Alberthua-Perl/tech-docs/blob/master/images/rapid-kube-deploy/crictl-ssl-ca-request-quay-error.jpg)

- Finally config `insecure_skip_verify = true` in config file and skip tls verify to pull images.

### Usage:

- Kani directory tree structure as following:
  
  ![](https://github.com/Alberthua-Perl/tech-docs/blob/master/images/rapid-kube-deploy/kani-tree.jpg)
  
  Because of GitHub uploading size limit, `cri-containerd-cni-1.5.5-linux-amd64.tar.gz` can be downloaded use this [link](https://pan.baidu.com/s/1ytxDjSN0u5Tewy5rcEGWNQ), password is `apdl`.

- Deploy kubernetes cluster after changing all files:

```bash
$ cd kani
$ chmod +x ./kani
$ ./kani deploy-kube
```

- kubernetes cluster status in the project as following:
  
  ![](https://github.com/Alberthua-Perl/tech-docs/blob/master/images/rapid-kube-deploy/kubernetes-cluster-status.jpg)

- Destroy kubernetes cluster and reinstall cluster:
  
  ```bash
  $ ./kani destroy-kube
  ```

- (Optional) `Red Hat Quay v3 registry`: 
  
  If use Red Hat Quay v3 as container image registry, run commands:
  
  ```bash
  $ ./kani config-quay
  # one step
  $ firefox
  # two step: config Quay on web UI 
  $ ./kani deploy-quay
  # three step
  ```
  
  You can read doc from [link](https://github.com/Alberthua-Perl/tech-docs/blob/master/Red%20Hat%20Quay%20v3%20registry%E5%8E%9F%E7%90%86%E4%B8%8E%E5%AE%9E%E7%8E%B0.md) to know how to config quay by Web UI.
  
  In our scenario containerd runtime use `godev` user to connect quay and pull images, so after deploy quay registry we should create godev user. You can create another user in quay registry according your environment.

### References:

- [Kubernetes Docs - Bootstrapping clusters with kubeadm](https://v1-22.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/)

- [Install Calico](https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises)
