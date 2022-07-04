# k3s

Setting up a new cluster in an OpenStack-based cloud such as ElastX or SafeSpring.

## Setup

### Master node

The master node requires less resources than the worker nodes. A 2 core setup with 4 GB RAM is sufficient for a small cluster.

Example specs:

> 2 VCPU
> 4 GB RAM
> 100 GB disk
> Ubuntu 22.04 (LTS)

1. Create the master node using the specs above
2. Assign a security group having ports 22, 80, 443 and 6443 open (TCP)
3. Make sure you give it access to a public network so you can SSH into it later
4. SSH into the node you just created `ssh ubuntu@13.37.0.1`
5. Setup k3s: `curl -sfL https://get.k3s.io | sh -`
6. Verify that you have a cluster `sudo kubectl get node`

**NOTE:** On SafeSpring you should only assign one network to each node. So pick _public_.

### Worker node(s)

Example specs:

> 4 VCPU
> 8 GB RAM
> 100 GB disk
> Ubuntu 22.04 (LTS)

1. Create one or more worker nodes using the specs above
2. Enter the customization script from the chapter below to have the worker(s) automatically join the cluster
3. Run `ssh ubuntu@13.37.0.1 sudo kubectl get node` to verify that the node(s) join the cluster

#### Customization script

OpenStack supports running a script inside each node node that is created. This is called the customization script (or --user-data when running the CLI). According to the docs the script can be up to 16 kB so there is really room for doing a lot of stuff. We will scratch the surface a little by having our node(s) install k3s and join the kubernetes cluster.

- **K3S_URL** is the URL to the kubernetes API on the master node
- **K3S_TOKEN** is found by running `ssh ubuntu@13.37.0.1 sudo cat /var/lib/rancher/k3s/server/node-token`

Modify the variables and then enter the whole script as your customization script when creating the worker node(s).

```shell
#!/bin/bash

K3S_URL=https://13.37.0.1:6443
K3S_TOKEN="K109bd91178d4de34b1b2cd8171e10344752b1be594af3772503a2cd8b4c4d11142::server:31baaa142225fd9330083e7121bd0119"

curl -sfL https://get.k3s.io | sh -
systemctl enable --now k3s-agent
```

## Maintaining the cluster

- Run updates regularly
