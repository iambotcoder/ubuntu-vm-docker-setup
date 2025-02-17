
# 🚀 Docker & Vagrant Virtual Machine Automation - Ubuntu Jammy

---

## 📖 Overview
This project demonstrates how to automate Docker containerization and provisioning of a virtual machine (VM) with Vagrant, using Ubuntu Jammy as the base image. The VM setup is tailored for running Docker and Apache web server on Ubuntu.

---

## 📑 Table of Contents
- [Prerequisites](-prerequisites) 🔑
- [Architecture](-architecture) 🗺️
- [Setup & Installation](-setup-and-installation) 🛠️
- [Vagrant Setup](-vagrant-setup) 🐳
- [Cleaning Up Resources](-cleaning-up-resources) 🧹
- [Conclusion](-conclusion) ✅

---

## 🔑 Prerequisites
Before starting, ensure you have the following installed:
- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Docker](https://docs.docker.com/get-docker/)
- A terminal or command prompt with administrative privileges

---

## 🗺️ Architecture
This project utilizes Vagrant to set up an Ubuntu virtual machine with Docker installed, and a custom Docker image that deploys an Apache web server with pre-packaged content. Here’s the architecture of the system:
- **Base OS:** Ubuntu Jammy 64-bit
- **Networking:**
  - Private network with a static IP (`192.168.56.82`)
  - Public network for external access
- **Resource Allocation:**
  - Memory: 2048MB
  - CPUs: 2
- **Synced Folder:** Optional, not configured here
- **Provisioning:**
  - Docker and Docker Compose installed
  - Apache server running a simple website

### Workflow:
1. **Initialize Vagrant** – Create the required configuration.
2. **Start the VM** – Boot the virtual machine.
3. **Run Docker Containers** – Deploy the Docker containers and images.
4. **Access the Application** – Test the web server running on the container.
5. **Destroy the VM** – Cleanup after testing is complete.

---

## 🛠️ Setup & Installation

### 1⃣ Initialize Vagrant:
```bash
vagrant init
```

### 2⃣ Vagrant Setup 🖥️

Below is the `Vagrantfile` configuration used for this project:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Use the Ubuntu Jammy box
  config.vm.box = "ubuntu/jammy64"

  # Private network for host-only access
  config.vm.network "private_network", ip: "192.168.56.82"

  # Public network for bridged access
  config.vm.network "public_network"

  # VirtualBox provider-specific configurations
  config.vm.provider "virtualbox" do |vb|
    # Allocate memory and CPUs
    vb.memory = "2048"
    vb.cpus = "2"
  end

  # Provision the VM with Docker and Apache installation
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg -y

    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
  SHELL
end
```

### 3⃣ Start the Virtual Machine:
```bash
vagrant up
```

### 4⃣ Check VM Status:
```bash
vagrant status
```

### 5⃣ SSH into the VM:
```bash
vagrant ssh
```

### 6⃣ Check Network Configuration:
```bash
ip addr show
```

### 7⃣ Build Docker Image & Run Containers

#### a. Build Image:
```bash
mkdir images
cd images/
vim Dockerfile
```

Paste the following content in the `Dockerfile`:

```Dockerfile
FROM ubuntu:latest AS BUILD_IMAGE
RUN apt update && apt install wget unzip -y
RUN wget https://www.tooplate.com/zip-templates/2128_tween_agency.zip
RUN unzip 2128_tween_agency.zip && cd 2128_tween_agency && tar -czf tween.tgz * && mv tween.tgz /root/tween.tgz

FROM ubuntu:latest
LABEL "project"="Marketing"
ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install apache2 git wget -y
COPY --from=BUILD_IMAGE /root/tween.tgz /var/www/html/
RUN cd /var/www/html/ && tar xzf tween.tgz
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
VOLUME /var/log/apache2
WORKDIR /var/www/html/
EXPOSE 80
```

Build the Docker image:
```bash
docker build -t tesimg .
```

#### b. Run the container:
```bash
docker run -P -d tesimg
```

### 8⃣ Access the Application:

After running the Docker container, use the following command to get the IP address and port mappings:

```bash
docker ps
ip addr show
```

Open a browser and navigate to `http://<VM-IP>:<Host-Port>`

---

## 🧹 Cleaning Up Resources

To remove the VM and free up system resources, run the following commands in order:

### 1⃣ Destroy the Virtual Machine:
```bash
vagrant destroy
```

### 2⃣ Remove Unused Vagrant Instances:
```bash
vagrant global-status --prune
```

### 3⃣ Clean Up Docker Containers and Images:

Stop and remove the running containers:
```bash
docker stop <container_name>
docker rm <container_name>
```

Remove unused images:
```bash
docker rmi <image_id>
```

---

## ✅ Conclusion

This project demonstrates how to automate VM provisioning with Vagrant and configure Docker for deploying web applications. It simplifies the process of building, running, and cleaning up Docker containers on an Ubuntu VM.

---

## 👨‍🏫 Instructor

This project was guided by ***Imran Teli*** , who provided valuable mentorship throughout the process.

---
