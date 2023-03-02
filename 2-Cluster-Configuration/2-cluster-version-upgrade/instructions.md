## Upgrading the Control Plane Node

1. Shell into the control plane node with `ssh kube-control-plane`.
2. Upgrade kubeadm to `1.23.4-00` via `apt-get`. Verify that the correct version has been set.
3. Drain the control plane node.
4. Use the appropriate kubeadm command to upgrade the control plane to `1.23.4`.
5. Uncordon the control plane node.
6. Upgrade kubelet and kubectl to `1.23.4-00`.
7. Restart the kubelet.
8. Exit out of the node.

## Upgrading the Worker Node

1. Shell into the worker node with `ssh kube-worker-1`.
2. Upgrade kubeadm to `1.23.4-00` via `apt-get`. Verify that the correct version has been set.
3. Drain the node.
4. Upgrade the kubelet configuration.
5. Upgrade kubelet and kubectl to `1.23.4-00`.
6. Restart the kubelet.
7. Uncordon the node.
8. Exit out of the node.
