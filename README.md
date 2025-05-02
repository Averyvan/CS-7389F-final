# CS 7389F Spring 2025 Preamble

This is the code for final project for CS 7389F based on the Gotham IoT Testbed paper (cited below). I ran into many issues with trying to get the original code repository to work, so most of my time was spent debugging and trying to reproduce the bare minimum functionality. I go into more detail on this topic in the [Reproducibility Issues](#reproducibility-issues) section.

## New version of the Gotham Scenario

As part of this project, I created a new version of the Gotham network with an additional layer of routers in front of each cluster of nodes within a neighborhood. This enables a study of the impact of additional firewall/indirection layers on the speed and transmissibility of various attacks. For example, Mirai bots are constantly doing wide scans on the network to find vulnerable devices, and this may be affected by these additional routers.

To use this alternative topology, use `./create_topology_gotham2.py` instead of `./create_topology_gotham.py` when creating the topology in [section 6](#6-topology-builder). 
```bash
$ python3 ./create_topology_gotham2.py
```

## Reproducibility Issues

System: Ubuntu 24.04.2 LTS

| Issue | Fix |
| --- | --- |
| Was unable to capture packets in the original [gotham-iot-testbed](https://github.com/xsaga/gotham-iot-testbed) repo | Switched to this downstream GothX repo | 
| Dependency aler9/rtsp-simple-server was deprecated on April 9, 2025, replaced by [bluenviron/mediamtx](https://hub.docker.com/r/bluenviron/mediamtx) | Changed stream_server's Dockerfile to use new image/repo |
| GothX: Gotham scenario missing links between main backbone routers | Re-added [the relevant code from the original repository](https://github.com/xsaga/gotham-iot-testbed/blob/master/src/create_topology_gotham.py#L166) |
| CICflowmeter ignores IPv6 packets, discarding all Gotham packets | Enabled reading IPv6 header in CICFlowMeter's [PacketReader.java](https://github.com/Averyvan/CS-7389F-final/blob/main/CICFlowMeter/src/main/java/cic/cs/unb/ca/ifm/Cmd.java) |

# GothX: a generator of customizable, legitimate and malicious IoT network traffic

This section and beyond are from the original GothX repository, with some sections that were irrelevant to the project removed, and a few clarifying details added.

This repository is a fork from [PekeDevil Gotham Testbed](https://github.com/PekeDevil/gotham-iot-testbed) (X. Sáez-de-Cámara, J. L. Flores, C. Arellano, A. Urbieta and U. Zurutuza, "Gotham Testbed: A Reproducible IoT Testbed for Security Experiments and Dataset Generation," in IEEE Transactions on Dependable and Secure Computing, doi: 10.1109/TDSC.2023.3247166)

It contains improved and extended source code of the testbed called Gotham to generate customizable, legitimate and malicious IoT network traffic.
Details about this traffic generator can be found in the paper [GothX: a generator of customizable, legitimate and malicious IoT network traffic](https://inria.hal.science/hal-04629350)

If you use or build upon this testbed, please consider citing the article.
> Manuel Poisson, Kensuke Fukuda, Rodrigo Carnier. GothX: a generator of customizable, legitimate and malicious IoT network traffic. CSET - 17th Cyber Security Experimentation and Test Workshop, Aug 2024, Philadelphia, United States. pp.1-9, ⟨10.1145/3675741.3675753⟩. ⟨hal-04629350v2⟩

# Table of contents

- [GothX: a generator of customizable, legitimate and malicious IoT network traffic](#gothx--a-generator-of-customizable--legitimate-and-malicious-iot-network-traffic)
- [Download the datasets generated with GothX](#download-the-datasets-generated-with-gothx)
- [Installation](#Installation)
  * [1 Install](#1-install)
    + [1.1 Install packages `make wget python3 konsole python3X-venv`](#11-install-packages--make-wget-python3-konsole-python3x-venv-)
    + [1.2  Install gns3 (server and gui)](#12--install-gns3--server-and-gui-)
    + [1.3 (Re)Install docker](#13--re-install-docker)
    + [1.4 Add user to the required groups `ubridge libvirt docker`](#14-add-user-to-the-required-groups--ubridge-libvirt-docker-)
    + [1.4 notes](#14-notes)
  * [2 Python virtual environment](#2-python-virtual-environment)
  * [3 Template creation](#3-template-creation)
    + [3.1 Build Docker images](#31-build-docker-images)
      - [3.1 Alternative B: Gotham topology](#31-gotham-topology)
    + [notes](#notes)
  * [4.1 start gns3](#41-start-gns3)
  * [5 Templates creation](#5-templates-creation)
    + [5.1 Docker templates creation](#51-docker-templates-creation)
    + [5.2 verify the created templates](#52-verify-the-created-templates)
    + [5.3 Create router template (manually with gui or automatically without gui)](#53-create-router-template--manually-with-gui-or-automatically-without-gui-)
      - [Option1: Create router template manually](#option1--create-router-template-manually)
      - [Option2: Create router template automatically without gui](#option2--create-router-template-automatically-without-gui)
- [GothX usage](#gothx-usage)
  * [6 Topology builder](#6-topology-builder)
      - [6.1 Gotham topology](#61-gotham-topology)
  * [7 Scenario execution](#7-scenario-execution)
      - [7.1 Gotham topology](#71-gotham-topology)
  * [8 pcap labelling](#8-pcap-labelling)


# Download the datasets generated with GothX

[You can download already generated datasets here.](https://files.inria.fr/aware/gothx-datasets.html)
More details about labels, settings and command line used during the traffic generation are available in the [dataset_details](./dataset_details) directory

----
# Installation

Tested on Ubuntu 20.04.4 LTS, 22.04 LTS and fedora 35

## 1 Install

Read [the install.sh script](./install.sh) to see commands for installation.
Some commands in `install.sh` only work with Linux distributions using `apt-get`.
Adapt accordingly if your distribution uses another package manager.

### 1.1 Install packages `make wget python3 konsole python3X-venv`
```bash
$ ./install.sh packages
```
### 1.2  Install gns3 (server and gui)
- Answer 'Yes' when the Wireshark installer asks 'Should non-superusers be able to capture packets?'
```bash
$ ./install.sh gns3
```
### 1.3 (Re)Install docker
```bash
$ ./install.sh docker
```
### 1.4 Add user to the required groups `ubridge libvirt docker`
```bash
$ ./install.sh groups
```
If the user is not correctly added to the groups, restart the machine.

### 1.4 notes

- For large topologies with many nodes (containers, VMs) you might need to increase the maximum number of open file descriptors (in the machine running GNS3) to start all the nodes simultaneously. 
  - Check the current limit with `ulimit -n`
  - Increase the current limit (to 2048 e.g.) with `ulimit -n 2048`

- KVM virtualization support. When running GNS3 in a virtual machine, enable nested virtualization.

---

## 2 Python virtual environment

Create a Python virtual environment to install the Python dependencies of the project (requires the `python3X-venv` package). To interact with the project, activate the virtual environment.

Inside the project's repository directory, run:

```
$ python3 -m venv venv
$ source venv/bin/activate
(venv) $ pip install -r requirements.txt
```

## 3 Template creation

### 3.1 Build Docker images

All the Dockerfiles and the dependencies that describe the emulated nodes (IoT devices, servers, attackers) are inside the `./Dockerfiles` directory. 
The build process of some Docker images depend on other images; instead of building them manually, the project includes a `Makefile` to automate the process.

#### 3.1 Gotham topology
Run `make` to automatically build all the Docker images in the correct order:
```
$ make -j
```

### notes
This `make` can take a long time (approx 30 min depending on the host and network speed); you can parallelize the build process running make with the `-j` flag.

If you modify any Dockerfile, configuration file, program or any other file inside the `./Dockerfiles` directory, run `make` again. It will rebuild the updated images and other images that depend on them.

## 4.1 start gns3
Open GNS3, the GNS3 server must be running.
- (Optional) If you want the graphical user interface (gui) run. 
This runs both gns3server and the gns3 client.
```bash
$ gns3
```
- If you don't need the graphical user interface (gui) run
```bash
$ gns3server
```

## 5 Templates creation

### 5.1 Docker templates creation
Inside the `./src` directory run:
```
(venv) $ python3 create_templates.py
```
### 5.2 verify the created templates
You can verify the created templates in GNS3 select: Edit > Preferences > Docker containers

![gns3 templates](img/gns3_templates.png)

### 5.3 Create router template (manually with gui or automatically without gui)

Inside the project's repository directory (`gotham-iot-testbed`), run:
```bash
make vyosiso
```

The artifacts (a .iso file and a .qcow2 file) will be downloaded into the directory`~/Downloads` directory. 

#### Option1: Create router template manually
Follow the instructions to import appliances in GNS3 https://docs.gns3.com/docs/using-gns3/beginners/import-gns3-appliance/. The router appliance file is located at `./router/iotsim-vyos.gns3a`.

![gns3 install vyos appliance](img/gns3_installvyosappliance.png)

#### Option2: Create router template automatically without gui

Inside the project's repository directory (`gotham-iot-testbed`), run
```bash
(venv) $ ./install_import_appliance.sh
```
Inside the `src` directory, run:
```bash
(venv) $ python3 create_templates.py vyos_template
```

----
# GothX usage

## 6 Topology builder

GNS3 must be running.

#### 6.1 Gotham topology

Inside the `./src` directory run:
```bash
(venv) $ python3 create_topology_gotham.py
```

Avery's note: Or use `create_topology_gotham2.py` for the more segmented version!

![gns3 topology](img/gns3_topology.png)

## 7 Scenario execution

GNS3 must be running.

#### 7.1 Gotham topology

Inside the `src/` directory, run the scenario

```
(venv) $ python3 run_scenario_gotham.py
```

Avery's note (I wish the original authors included this info...): If you want to run the original Gotham topology, you need to change the `PROJECT_NAME` variable in `src/run_scenario_gotham.py` from `gotham_scenario_mirai_new` (for gotham2) to `gotham_scenario_mirai` (for gotham).



## 8 pcap labelling

From a pcap file, extract network flows with [CICFlowMeter](https://github.com/GintsEngelen/CICFlowMeter), you get a csv file with features. 

Avery's note: I recommend using the docker instructions and use the included version of the program.


