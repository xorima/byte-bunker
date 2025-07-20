PROXMOX_USER ?= root
PROXMOX_HOST ?= 192.168.100.5
VM_IDS ?= 1 2 3


validate:
	@if [ -z "$(version)" ]; then \
		echo "Error: version is not set."; \
		echo "Usage: make <command> version=<talos_version>"; \
		exit 1; \
	fi


download: validate
	curl -L https://github.com/siderolabs/talos/releases/download/$(version)/metal-amd64.iso -o talos-$(version)-amd64.iso

upload: validate
	scp talos-$(version)-amd64.iso root@192.168.100.5:/var/lib/vz/template/iso/

create-cluster: validate
	for i in $(VM_IDS); do \
		ssh $(PROXMOX_USER)@$(PROXMOX_HOST) "qm create 900$$i --name bb-talos-hk$$i --memory 8192 --net0 e1000=BC:24:11:D1:DF:5$$i,bridge=vmbr0,firewall=1 --ide2 local:iso/talos-$(version)-amd64.iso,media=cdrom --boot order=scsi0\;ide2 --cores 2 --sockets 1 --cpu x86-64-v4 --scsi0 vms:50 --scsihw virtio-scsi-pci"; \
	done

stop-cluster: 
	for i in $(VM_IDS); do \
		ssh $(PROXMOX_USER)@$(PROXMOX_HOST) "qm stop 900$$i"; \
	done

start-cluster:
	for i in $(VM_IDS); do \
		ssh $(PROXMOX_USER)@$(PROXMOX_HOST) "qm start 900$$i"; \
	done


destroy-cluster: stop-cluster
	for i in $(VM_IDS); do \
		ssh $(PROXMOX_USER)@$(PROXMOX_HOST) "qm destroy 900$$i --purge --destroy-unreferenced-disks"; \
	done

install-talosctl:
	brew install siderolabs/tap/talosctl

create-cluster-settings: install-talosctl
	@if [ ! -f secrets.yaml ]; then \
		talosctl gen secrets; \
	fi
	talosctl gen config bb-talos-hk-cluster https://192.168.100.210:6443 \
	--with-secrets secrets.yaml \
	--config-patch @patches/allow-controlplane-workloads.yaml \
	--config-patch @patches/cni.yaml \
	--config-patch @patches/dhcp.yaml \
	--config-patch @patches/install-disk.yaml \
	--config-patch @patches/interface-names.yaml \
	--config-patch @patches/kubelet-certificates.yaml \
	--config-patch-control-plane @patches/vip.yaml \
	--output rendered/

bootstrap-cluster:
	for i in $(VM_IDS); do \
		talosctl apply -f rendered/controlplane.yaml -n 192.168.100.21$$i --insecure; \
	done
    @mkdir -p .talos
	cp rendered/talosconfig .talos/config
	echo "Run: export TALOSCONFIG=.talos/config"
	echo "Run: export KUBECONFIG=.talos/kubeconfig"
	talosctl config --talosconfig=.talos/config endpoint 192.168.100.211 192.168.100.212 192.168.100.213
	talosctl config --talosconfig=.talos/config node 192.168.100.211
	talosctl --talosconfig=.talos/config bootstrap
	talosctl --talosconfig=.talos/config kubeconfig .talos/kubeconfig
