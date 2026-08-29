# Containerlab Topology Setup Guide

This guide provides step-by-step instructions for deploying a virtual network topology using [Containerlab](https://containerlab.dev/). The environment includes Nokia SR Linux routers, Arista cEOS multi-layer switches, lightweight Alpine Linux hosts, and an Nginx web server, all integrated with a dedicated Out-of-Band (OOB) management network.

## Prerequisites
* A host machine running a Linux distribution (e.g., Ubuntu/Debian).
* Internet access to download required packages and Docker images.
* An active [Arista.com](https://www.arista.com/) account (required for downloading the proprietary cEOS image).

---

## Step 1: Install Docker Engine

Containerlab relies on a container runtime to spin up the network nodes. We will install Docker Engine using the official convenience script, which supports most standard Linux distributions.

```bash
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sudo sh get-docker.sh
```
> **Note:** Verify the installation by running `docker --version`. It is recommended to add your user to the `docker` group to run commands without `sudo`, though this guide assumes standard root/sudo privileges for network operations.

## Step 2: Install Containerlab

Execute the official installation script to download and install the latest Containerlab binary.

```bash
bash -c "$(curl -sL [https://get.containerlab.dev](https://get.containerlab.dev))"
```
> **Note:** Verify the installation by running `containerlab version`.

## Step 3: Retrieve Public Docker Images

Our topology utilizes several public images that can be pulled directly from standard container registries.

```bash
# Pull Nokia SR Linux for the routing nodes
docker pull ghcr.io/nokia/srlinux:latest

# Pull Alpine Linux for the lightweight end-hosts
docker pull alpine:latest

# Pull Nginx for the web server
docker pull nginx:alpine
```

## Step 4: Import the Arista cEOS Image

The Arista cEOS (Containerized EOS) image is proprietary and must be manually downloaded and imported into Docker.

1. Navigate to the Arista Software Download Portal.
2. Log in with your registered account.
3. Go to EOS > cEOS-lab and locate the desired release (this guide uses the 4.36.2F train).
4. Download the 64-bit `.tar.xz` file (e.g., `cEOS64-lab-4.36.2F.tar.xz`) to your Linux machine.
5. Import the archive into Docker using the following command:

```bash
docker import cEOS64-lab-4.36.2F.tar.xz ceosimage:4.36.2F
```
> **Note:** Run `docker images` to confirm that `alpine:latest`, `nginx:alpine`, `ghcr.io/nokia/srlinux:latest`, and `ceosimage:4.36.2F` are all successfully loaded and present on your system.

## Step 5: Define the Containerlab Topology

Create a file named `ana-lab-topo.clab.yml`. This YAML file defines the nodes, the container images they use, and the explicit point-to-point links (virtual cables) connecting them.

> **Note on Naming & SSH:** By default, Containerlab prefixes node names with the lab name (e.g., `clab-ana-lab-topo-R1`). We use `prefix: ""` in this configuration to force strict naming. This allows you to SSH into the devices using their exact, short hostnames (like `R1` or `S1`).

> **Note on Management Architecture:** This topology includes a dedicated `mgmt` cEOS switch. It acts as an Out-of-Band (OOB) bridge connecting port 10 on every device directly to the Linux host VM via a virtual interface (`ubuntu-vm`). This prepares the environment for a cloud-based NMAS connection (e.g., via Tailscale) without relying on Containerlab's default management bridge.

```yaml
name: ana-lab-topo
prefix: ""

topology:
  nodes:
    # Dedicated Management Switch
    mgmt:
      kind: ceos
      image: ceosimage:4.36.2F

    # Routers (Nokia SR Linux)
    R1:
      kind: srl
      image: ghcr.io/nokia/srlinux:latest
    R2:
      kind: srl
      image: ghcr.io/nokia/srlinux:latest
    R3:
      kind: srl
      image: ghcr.io/nokia/srlinux:latest
    R4:
      kind: srl
      image: ghcr.io/nokia/srlinux:latest
    R5:
      kind: srl
      image: ghcr.io/nokia/srlinux:latest

    # Switches (Arista cEOS)
    S1:
      kind: ceos
      image: ceosimage:4.36.2F
    S2:
      kind: ceos
      image: ceosimage:4.36.2F
    S3:
      kind: ceos
      image: ceosimage:4.36.2F
    S4:
      kind: ceos
      image: ceosimage:4.36.2F

    # End Hosts (Alpine Linux)
    # The 'cmd: sleep infinity' argument prevents the container from immediately exiting.
    H1:
      kind: linux
      image: alpine:latest
      cmd: sleep infinity
    H2:
      kind: linux
      image: alpine:latest
      cmd: sleep infinity
    H3:
      kind: linux
      image: alpine:latest
      cmd: sleep infinity
    H4:
      kind: linux
      image: alpine:latest
      cmd: sleep infinity

    # Server
    webserver:
      kind: linux
      image: nginx:alpine

  links:
    # --- Data Plane Connections ---
    - endpoints: ["R5:e1-1", "webserver:eth1"]
    - endpoints: ["R5:e1-2", "R3:e1-2"]
    - endpoints: ["R5:e1-3", "R4:e1-3"]
    - endpoints: ["R3:e1-1", "R4:e1-1"]
    - endpoints: ["R3:e1-3", "S3:eth3"]
    - endpoints: ["R4:e1-2", "S4:eth2"]
    - endpoints: ["S3:eth1", "S4:eth1"]
    - endpoints: ["S3:eth2", "R1:e1-2"]
    - endpoints: ["S4:eth3", "R2:e1-3"]
    - endpoints: ["R1:e1-1", "S1:eth1"]
    - endpoints: ["R2:e1-1", "S2:eth1"]
    - endpoints: ["S1:eth2", "S2:eth2"]
    - endpoints: ["S1:eth3", "H1:eth3"]
    - endpoints: ["S1:eth4", "H2:eth4"]
    - endpoints: ["S2:eth3", "H3:eth3"]
    - endpoints: ["S2:eth4", "H4:eth4"]

    # --- Out-of-Band Management Connections ---
    - endpoints: ["mgmt:eth1", "H1:eth10"]
    - endpoints: ["mgmt:eth2", "H2:eth10"]
    - endpoints: ["mgmt:eth3", "H3:eth10"]
    - endpoints: ["mgmt:eth4", "H4:eth10"]
    - endpoints: ["mgmt:eth5", "R1:e1-10"]
    - endpoints: ["mgmt:eth6", "R2:e1-10"]
    - endpoints: ["mgmt:eth7", "R3:e1-10"]
    - endpoints: ["mgmt:eth8", "R4:e1-10"]
    - endpoints: ["mgmt:eth9", "R5:e1-10"]
    - endpoints: ["mgmt:eth10", "S1:eth10"]
    - endpoints: ["mgmt:eth11", "S2:eth10"]
    - endpoints: ["mgmt:eth12", "S3:eth10"]
    - endpoints: ["mgmt:eth13", "S4:eth10"]
    - endpoints: ["mgmt:eth14", "webserver:eth10"]
    
    # --- Host VM Gateway Connection ---
    - endpoints: ["mgmt:eth20", "host:ubuntu-vm"]
```

## Step 6: Deploy the Topology

Execute the deployment command from the directory containing your `.clab.yml` file.

```bash
sudo containerlab deploy -t ana-lab-topo.clab.yml
```
Containerlab will instantiate all containers, configure the SSH keys locally, and wire the veth interfaces between the nodes. A summary table will be printed upon completion.

## Step 7: Verify Access

Because we removed the standard naming prefix, Containerlab updates the local `/etc/ssh/ssh_config.d/` directory with strict host aliases. You can instantly access the management CLI of any network device by using standard SSH syntax with the default administrative user (`admin`).

```bash
# Example: SSH into Router 1
ssh admin@R1

# Example: SSH into Switch 1
ssh admin@S1
```
> **Note:** The Layer 1 lab is now successfully running. You may proceed to manually configure the data-plane elements (OSPF, RIPv2, VLANs) and the OOB subnets (e.g., `192.168.10.0/24`) inside the respective network operating systems.
