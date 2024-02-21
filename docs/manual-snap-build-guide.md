## Microk8s Snap

#### Instructions:
For MicroK8s inspect the snapcraft.yaml and go over all parts and build them manually one by one. This way when the time comes to patch any of them you will know how to do that. Understanding what is in the snapcraft.yaml will also help you understand how the k8s cluster is setup and what its commands and daemons are. Spend some time there to get familiar with the commands and the code each one wraps

## What happens when the snap is built

First have a look at the snapcraft file.
You can have a look at the [CI snap build](https://github.com/canonical/microk8s/actions/runs/7972601834/job/21764687592). I would recommend searching for some build step keywords.
## Set Up
Let's set up our dev environment.
#### Create VM
```
multipass launch -c 4 -d 30G -m 9G --name KU-357-microk8s 22.04
multipass shell KU-357-microk8s 
```
#### Create Microk8s repo
```
git clone https://github.com/canonical/microk8s.git
```

#### Install Snapcraft

```
sudo snap install snapcraft --classic
```

## Build deps
Install Dependencies:
```
sudo apt-get update
sudo apt-get install autoconf automake autopoint autotools-dev bison btrfs-progs libbtrfs-dev build-essential curl flex git libjansson-dev liblz4-dev libnetfilter-conntrack-dev libnetfilter-conntrack3 libnfnetlink-dev libseccomp-dev libtool libuv1-dev pkg-config rsync tcl
```
Install go 
```
sudo snap install --classic --channel=1.21/stable go
```


## Build Components:
We have a handful of components:
- [K8s-dqlite](https://github.com/canonical/k8s-dqlite)
This database can be used to replace etcd in the microk8s cluster.
- etcd
- ...


#### Build K8s-dqlite:
These are the instructions from the `snapcraft.yaml`:
```
  k8s-dqlite:
    after: [build-deps]
    source: build-scripts/components/k8s-dqlite
    plugin: nil
    override-build: $SNAPCRAFT_PROJECT_DIR/build-scripts/build-component.sh k8s-dqlite

```
Let's break it apart:
- k8s-dqlite: name of the component
- after: we can only build the component after we've installed all the build-deps
- source: points to the source code or necessary files (but not used)
- plugin: this is no standard Snapcraft plugin
- override-build: The script to build the specified component.

So we have to go into the microk8s dir and build the component:
```
cd microk8s/
./build-scripts/build-component.sh k8s-dqlite
```

What does the `build-component.sh` script do?

Go to build-scripts/components/k8s-dqlite. You should see (at least 3 files)
- repository: the git repo
- version: the version of our component
- build.sh: a build script for this specific component.

The build script grabs the components git repo and downloads our specific component's version.
Then it makes sure everything that got installed gets placed in the right location.

Where does k8s-dqlite get installed?
```
ls /home/ubuntu/microk8s/build-scripts/.install/bin/
```