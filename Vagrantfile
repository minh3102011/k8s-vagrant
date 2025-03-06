=begin
Vagrant.configure("2") do |config|
  # Cấu hình Workernode
  config.vm.define "Workernode" do |worker|
    worker.vm.box = "bento/ubuntu-22.04"
    worker.vm.hostname = "Workernode"
    worker.vm.network "public_network", bridge: "VMware Bridge Adapter 8"
    worker.vm.provider "vmware_desktop" do |vmware|
      vmware.memory = 1024
      vmware.cpus = 2
      vmware.vmx["displayName"] = "Workernode"
    end

    worker.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt upgrade -y
      # Thêm cả Masternode và Workernode vào /etc/hosts
      IPWorkernode=$(hostname -I | awk '{print $1}')
      IFS='.' read -r i1 i2 i3 i4 <<< "$IPWorkernode"
      IPMasternode="$i1.$i2.$i3.$((i4+1))"
      sudo swapoff -a
      sudo sed -i 's|^\(/swap.img.*\)|#\1|' /etc/fstab
      sudo mount -a
      free -h
      # Cập nhật hostname
      sudo hostnamectl set-hostname "k8s-worker.ndbien.local"
      sudo sed -i '2s/.*/127.0.1.1 k8s-worker/' /etc/hosts
      echo "$IPMasternode k8s-master.ndbien.local k8s-master" | sudo tee -a /etc/hosts
      echo "$IPWorkernode k8s-worker.ndbien.local k8s-worker" | sudo tee -a /etc/hosts
      sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
      sudo modprobe overlay
      sudo modprobe br_netfilter
      sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
      sudo sysctl --system
      sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates docker.io
      sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
      sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
      echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      # sudo apt install -y docker-ce docker-ce-cli containerd.io
      sudo apt-get install -y docker.io
      wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb
      sudo dpkg -i cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb
      sudo systemctl enable --now cri-docker
      sudo systemctl restart docker
      sudo systemctl restart cri-docker
      sudo apt update
      containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
      sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
      # sudo systemctl restart containerd
      # sudo systemctl enable containerd
      #helm
      curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
      chmod 700 get_helm.sh
      ./get_helm.sh
      # Cài đặt Kubernetes & Docker
      sudo apt-get update && sudo apt-get upgrade -y
      sudo apt-get install -y apt-transport-https ca-certificates curl gpg
      sudo mkdir -p -m 755 /etc/apt/keyrings
      curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
      echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
      sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
      sudo apt-mark hold kubelet kubeadm kubectl
    SHELL
  end

  # Cấu hình Masternode
  config.vm.define "Masternode" do |master|
    master.vm.box = "bento/ubuntu-22.04"
    master.vm.hostname = "Masternode"
    master.vm.network "public_network", bridge: "VMware Bridge Adapter 8"
    master.vm.provider "vmware_desktop" do |vmware|
      vmware.memory = 3072
      vmware.cpus = 2
      vmware.vmx["displayName"] = "Masternode"
    end

    master.vm.provision "shell", inline: <<-SHELL
      # Lấy địa chỉ IP của Masternode
      IPMasternode=$(hostname -I | awk '{print $1}')
      IFS='.' read -r i1 i2 i3 i4 <<< "$IPMasternode"
      IPWorkernode="$i1.$i2.$i3.$((i4-1))"

      # Cập nhật hostname
      sudo hostnamectl set-hostname "k8s-master.ndbien.local"
      sudo sed -i '2s/.*/127.0.1.1 k8s-master/' /etc/hosts
      echo "$IPMasternode k8s-master.ndbien.local k8s-master" | sudo tee -a /etc/hosts
      echo "$IPWorkernode k8s-worker.ndbien.local k8s-worker" | sudo tee -a /etc/hosts
            sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
      # Cài đặt Kubernetes & Docker
      sudo apt-get update && sudo apt-get upgrade -y
      sudo apt-get install -y docker.io apt-transport-https ca-certificates curl gpg
      #helm
      curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
      chmod 700 get_helm.sh
      ./get_helm.sh
      
      sudo swapoff -a
      sudo sed -i 's|^\(/swap.img.*\)|#\1|' /etc/fstab
      sudo mount -a
      free -h
      sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
      sudo modprobe overlay
      sudo modprobe br_netfilter
      sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
      sudo sysctl --system
      sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
      sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
      sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
      echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      sudo apt install -y docker-ce docker-ce-cli containerd.io

      wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb
      sudo dpkg -i cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb
      sudo systemctl enable --now cri-docker
      sudo systemctl restart docker
      sudo systemctl restart cri-docker
      sudo apt update
      # sudo apt install -y containerd.io
      containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
      sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
      # sudo systemctl restart containerd
      # sudo systemctl enable containerd
      #kubectl
      sudo mkdir -p -m 755 /etc/apt/keyrings
      curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
      echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
      sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
      sudo apt-mark hold kubelet kubeadm kubectl
      # Khởi tạo Kubernetes
      sudo kubeadm init --cri-socket unix:///var/run/cri-dockerd.sock --pod-network-cidr=192.168.0.0/16   --control-plane-endpoint=k8s-master.ndbien.local| tee /home/vagrant/kubeadm-init.log
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      sudo curl https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml -O
      sudo curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O
      # sudo sed -i 's/"Network": "[0-9.\/]*"/"Network": "192.168.0.0\/16"/' kube-flannel.yml
    SHELL
  end
end
=end

  
Vagrant.configure("2") do |config|
  # Hàm setup chung cho cả MasterNode & WorkerNode
  def setup_common(vm)
    vm.vm.box = "bento/ubuntu-22.04"
    vm.vm.network "public_network", bridge: "VMware Bridge Adapter 8"

    vm.vm.provision "shell", inline: <<-SHELL
      sudo apt update && sudo apt upgrade -y
      sudo swapoff -a
      sudo sed -i 's|^\(/swap.img.*\)|#\1|' /etc/fstab
      echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
      sudo mount -a
      sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
      sudo modprobe overlay
      sudo modprobe br_netfilter
      sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
      sudo sysctl --system
      sudo apt-get install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
      sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
      sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
      sudo apt-get update && sudo apt-get install -y docker.io

      # Cài cri-dockerd (chạy Docker với Kubernetes)
      wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb
      sudo dpkg -i cri-dockerd_0.3.1.3-0.ubuntu-jammy_amd64.deb
      sudo systemctl enable --now cri-docker
      sudo systemctl restart docker
      sudo systemctl restart cri-docker
      # Cấu hình containerd
      containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
      sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
      sudo usermod -aG docker vagrant
      newgrp docker
      # Cài đặt Helm
      curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
      chmod 700 get_helm.sh
      ./get_helm.sh

      # Cài đặt Kubernetes
      sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gpg
      sudo mkdir -p -m 755 /etc/apt/keyrings
      curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
      echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
      sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
      sudo apt-mark hold kubelet kubeadm kubectl
      #cài đặt kubectl
      curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
      sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
      chmod +x kubectl
      mkdir -p ~/.local/bin
      mv ./kubectl ~/.local/bin/kubectl
      
    SHELL
  end

  # Cấu hình Workernode
  config.vm.define "Workernode" do |worker|
    setup_common(worker)
    worker.vm.hostname = "Workernode"
    worker.vm.provider "vmware_desktop" do |vmware|
      vmware.memory = 1024
      vmware.cpus = 2
      vmware.vmx["displayName"] = "Workernode"
    end

    worker.vm.provision "shell", inline: <<-SHELL
      # Cấu hình hostname & hosts file
      IPWorkernode=$(hostname -I | awk '{print $1}')
      IFS='.' read -r i1 i2 i3 i4 <<< "$IPWorkernode"
      IPMasternode="$i1.$i2.$i3.$((i4+1))"

      sudo hostnamectl set-hostname "k8s-worker.ndbien.local"
      sudo sed -i '2s/.*/127.0.1.1 k8s-worker/' /etc/hosts
      echo "$IPMasternode k8s-master.ndbien.local k8s-master" | sudo tee -a /etc/hosts
      echo "$IPWorkernode k8s-worker.ndbien.local k8s-worker" | sudo tee -a /etc/hosts
    SHELL
  end

  # Cấu hình Masternode
  config.vm.define "Masternode" do |master|
    setup_common(master)
    master.vm.hostname = "Masternode"
    master.vm.provider "vmware_desktop" do |vmware|
      vmware.memory = 3072
      vmware.cpus = 2
      vmware.vmx["displayName"] = "Masternode"
    end

    master.vm.provision "shell", inline: <<-SHELL
      # Cấu hình hostname & hosts file
      IPMasternode=$(hostname -I | awk '{print $1}')
      IFS='.' read -r i1 i2 i3 i4 <<< "$IPMasternode"
      IPWorkernode="$i1.$i2.$i3.$((i4-1))"

      sudo hostnamectl set-hostname "k8s-master.ndbien.local"
      sudo sed -i '2s/.*/127.0.1.1 k8s-master/' /etc/hosts
      echo "$IPMasternode k8s-master.ndbien.local k8s-master" | sudo tee -a /etc/hosts
      echo "$IPWorkernode k8s-worker.ndbien.local k8s-worker" | sudo tee -a /etc/hosts

      # Khởi tạo Kubernetes
      sudo kubeadm init --cri-socket unix:///var/run/cri-dockerd.sock --pod-network-cidr=192.168.0.0/16 --control-plane-endpoint=k8s-master.ndbien.local | tee /home/vagrant/kubeadm-init.log
      
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      sudo scp /etc/kubernetes/admin.conf vagrant@k8s-worker:/home/vagrant/.kube/config
      # Cài đặt CNI (Calico hoặc Flannel)
      curl https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml -O
      curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O
    SHELL
  end
end

