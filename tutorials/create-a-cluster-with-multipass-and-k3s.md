---
title: How to create a local Kubernetes cluster with Multipass and K3s
description: This tutorial is designed to show how you can create a local Kubernetes cluster with Multipass and K3s.
layout: tutorial
collection: tutorials
---

This tutorial is designed to show how you can create a local Kubernetes cluster with Multipass and K3s.

## Prerequisites

* {% include prerequisites-linux-macos.md %}
* `kubectl`. Refer to the [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) page for details about how you can install `kubectl`.

## Create three virtual machines

1. Install Multipass. Point your browser to the [Mulitpass website](https://multipass.run/), and then download the binary for your platform.

2. Display the list of available images:

    ```Bash
    multipass find
    ```

    ```
    Image                   Aliases           Version          Description
    snapcraft:core          core16            20200318         Snapcraft builder for Core 16
    snapcraft:core18                          20200320         Snapcraft builder for Core 18
    16.04                   xenial            20200318         Ubuntu 16.04 LTS
    18.04                   bionic,lts        20200317         Ubuntu 18.04 LTS
    ```

3. To create a node named `master`, run the following `multipass launch` command and pass it the following flags:

     * `-c` with the number of CPUs to allocate (`1`)
     * `-m` with the amount of memory to allocate (`1G`). Note that you can use the following suffixes: `K`, `M`, and `G`
     * `-d` with the disk space to allocate (`4G`). Similarly, note that you can use the following suffixes: `K`, `M`, and `G`
     * `-n` with the name of the node (`master`)
     * The name of the image (`18.04`)

    ```Bash
    multipass launch -c 1 -m 1G -d 4G -n k3s-master 18.04
    ```

    ```
    Launched: k3s-master
    ```

4. Retrieve details about the `master node with:

    ```Bash
    multipass info k3s-master
    ```

    ```
    Name:           k3s-master
    State:          Running
    IPv4:           192.168.64.16
    Release:        Ubuntu 18.04.4 LTS
    Image hash:     fe3030939822 (Ubuntu 18.04 LTS)
    Load:           1.12 1.35 0.61
    Disk usage:     1003.0M out of 3.7G
    Memory usage:   72.4M out of 985.7M
    ```

5. Enter the following Bash script to create two worker nodes named `node2` and `node3`:


    ```Bash
    for f in 1 2; do
        multipass launch -c 1 -m 1G -d 4G -n k3s-worker-$f 18.04
    done
    ```

    ```
    Launched: k3s-worker-1
    Launched: k3s-worker-2
    ```

6. Use the `multipass list` command to make sure everything went well. The following example output shows that your nodes are in `Running` state:

    ```Bash
    multipass list
    ```

    ```
    Name                    State             IPv4             Image
    k3s-master              Running           192.168.64.16    Ubuntu 18.04 LTS
    k3s-worker-1            Running           192.168.64.17    Ubuntu 18.04 LTS
    k3s-worker-2            Running           192.168.64.18    Ubuntu 18.04 LTS
    ```

## Deploy K3s

K3s is a lightweight Kubernetes distribution geared towards resource-constrained environments. Its easy installation process h makes it perfect for rapid prototyping

1. Use the `multipass exec` command to deploy K3s to the `k3s-master` node:

    ```Bash
    multipass exec k3s-master -- bash -c "curl -sfL https://get.k3s.io | sh -"
    ```

    ```
    [INFO]  Finding latest release
    [INFO]  Using v1.17.4+k3s1 as release
    [INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.4+k3s1/sha256sum-amd64.txt
    [INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.4+k3s1/k3s
    [INFO]  Verifying binary download
    [INFO]  Installing k3s to /usr/local/bin/k3s
    [INFO]  Creating /usr/local/bin/kubectl symlink to k3s
    [INFO]  Creating /usr/local/bin/crictl symlink to k3s
    [INFO]  Creating /usr/local/bin/ctr symlink to k3s
    [INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
    [INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
    [INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
    [INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
    [INFO]  systemd: Enabling k3s unit
    Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
    [INFO]  systemd: Starting k3s
    ```

    Note that, in the above command, everything after `--` is passed as an argument to the virtual machine.

2. The K3s installer created a Kubernetes token on the `k3s-master` node and saved it into the `/var/lib/rancher/k3s/server/node-token` file. Execute the following command to store the content of this file into an environment variable called `TOKEN`:

    ```Bash
    TOKEN=$(multipass exec k3s-master sudo cat /var/lib/rancher/k3s/server/node-token)
    ```

3. You can inspect your Kubernetes token as follows:

    ```Bash
    K10a822199e28e2c11a0eb19fe70f483a1a204cf3ee243cc2d12b75fe54c0de66ac::server:3e440ae3b6f4720c6e9c0438c76adc18
    ```

    ```
    K10db5d436cafc40df4e69e9eaec0df4a35ad982e600e3acecda5264d73859c2618::server:beed2c91b9c38560fc6f82d3649dc94a
    ```

4. Save the IP of your `master` node into an environment variable named `IP`

    ```Bash
    IP=$(multipass info k3s-master | grep IPv4 | awk '{print $2}')
    ```

5. The following `multipass exec` command uses the `TOKEN` and `IP` environment variables to add the worker nodes to your cluster:

    ```Bash
    for f in 1 2; do
        multipass exec k3s-worker-$f -- bash -c "curl -sfL https://get.k3s.io | K3S_URL=\"https://$IP:6443\" K3S_TOKEN=\"$TOKEN\" sh -"
    done
    ```

    ```
    [INFO]  Finding latest release
    [INFO]  Using v1.17.4+k3s1 as release
    [INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.4+k3s1/sha256sum-amd64.txt
    [INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.4+k3s1/k3s
    [INFO]  Verifying binary download
    [INFO]  Installing k3s to /usr/local/bin/k3s
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
    [INFO]  Finding latest release
    [INFO]  Using v1.17.4+k3s1 as release
    [INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.4+k3s1/sha256sum-amd64.txt
    [INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.4+k3s1/k3s
    [INFO]  Verifying binary download
    [INFO]  Installing k3s to /usr/local/bin/k3s
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
    ```

## Verify your cluster

1. Use the following `multipass exec` command to open a shell session into the `k3s-master` and then list the nodes in your cluster:

    ```Bash
    multipass exec k3s-master -- bash
    ```

    ```
    ubuntu@k3s-master:~$
    ```

2. Now you can list the nodes in your cluster as follows:

    ```Bash
    sudo kubectl get nodes
    ```

    ```
    NAME           STATUS   ROLES    AGE     VERSION
    k3s-master     Ready    master   11m     v1.17.4+k3s1
    k3s-worker-1   Ready    <none>   4m47s   v1.17.4+k3s1
    k3s-worker-2   Ready    <none>   4m27s   v1.17.4+k3s1
    ```


## Clean up your cluster

1. Open a new terminal window (on the host), and then use the `multipass stop` command to stop your nodes:

    ```Bash
    multipass stop k3s-master k3s-worker-1 k3s-worker-2
    ```

2. Delete your instances with:

    ```Bash
    multipass delete k3s-master k3s-worker-1 k3s-worker-2
    ````

3. Permanently delete your instances by entering:

    ```
    multipass purge
    ```

4. Use the `multipass list` command to verify that the instances were deleted:

    ```Bash
    multipass list
    ```

    ```
    No instances found.
    ```

---

Congratulations on completing this tutorial, where you learned how to deploy Kubernetes cluster with Multipath and K3s.

Thanks for reading!
<!-- To learn even more, continue with the following tutorials. -->
