# ByteBunker Cluster Setup

This repository contains automation for configuring my homelab Kubernetes cluster using [Talos Linux](https://www.talos.dev/) as the operating system, following the approach described in [The Best OS for Kubernetes](https://mirceanton.com/posts/2023-11-28-the-best-os-for-kubernetes/).

## Prerequisites

- Access to a Proxmox server.
- `make`, `curl`, `scp`, `ssh`, and `brew` installed locally.
- [Talosctl](https://www.talos.dev/) installed (`make install-talosctl`).
- Proxmox credentials and network details configured in the `Makefile` as needed.

## Configuration Steps

1. **Set the Talos version:**
   ```sh
   export version=<talos_version>
   ```

2. **Download Talos ISO:**
   ```sh
   make download version=$version
   ```

3. **Upload ISO to Proxmox:**
   ```sh
   make upload version=$version
   ```

4. **Create VMs for the cluster:**
   ```sh
   make create-cluster version=$version
   ```

5. **Generate cluster settings and secrets:**
   ```sh
   make create-cluster-settings version=$version
   ```

6. **Bootstrap the cluster:**
   ```sh
   make bootstrap-cluster
   ```

7. **Start/Stop/Destroy cluster VMs:**
   - Start: `make start-cluster`
   - Stop: `make stop-cluster`
   - Destroy: `make destroy-cluster`

## Notes

- The nodes need their ips handled by reservations for now in DHCP - this will be fixed in future. 
- The cluster configuration and patches are managed in the `patches/` directory.
- Talos and Kubernetes configuration files are rendered in the `rendered/` directory.
- After bootstrapping, set environment variables for Talos and Kubernetes:
  ```sh
  export TALOSCONFIG=.talos/config
  export KUBECONFIG=.talos/kubeconfig
  ```

## Reference

This setup is based on the guide: [The Best OS for Kubernetes](https://mirceanton.com/posts/2023-11-28-the-best-os-for-kubernetes/).
