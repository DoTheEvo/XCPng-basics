# XCP-ng

###### guide-by-example

![logo](https://i.imgur.com/FBH2aII.png)

1. [Purpose & Overview](#Purpose--Overview)
2. [Why XCP-ng](#Why-XCP-ng)
3. [Xen Orchestra](#Xen-Orchestra)
4. [The Basics](#The-Basics)
5. [Backups](#Backups)
6. [Advanced Concepts](#Advanced-Concepts)
7. [Issues encountered](#Issues-encountered)
8. [Videos](#Videos)

# Purpose & Overview

A virtualization platform build around
**Xen a type 1 hypervisor**.<br>
An alternative to Proxmox or ESXi/vCenter.

[Xen](https://en.wikipedia.org/wiki/Xen)
is an open source project, developed under **the Linux Foundation** with
support from major industry players - AWS, intel, amd, arm, google,
alibaba cloud,...<br>
2007-2013 Citrix directed Xen development, but gave up the control
to attract more collaboration from the industry giants, of which AWS(amazon)
is the biggest xen user. 

[XCPng](https://en.wikipedia.org/wiki/XCP-ng) itself started on kickstarter
as a fork of XenServer, which with version 7.0 closed-source some of it's components.
The first release came out in 2018, but Vates - the company behind it,
worked on Xen Orchestra since 2012. They are located in France
and have \~90 employees.

* **Xen** - The hypervisor.
* **XCPng** - A single purpose linux distro preconfigured with xen,
  uses centos user space.<br>
  This is what you install on the metal.
* **XO** - Xen Orchestra - a web interface for centralized management
  of xcpng hosts.<br>
  Deployed as a container or a virtual machine.
* **XOA** - Xen Orchestra Appliance - a paid version of XO with full support
  and some extra features like [XOSTOR](https://vates.tech/xostor/)
  through webGUI.
* **XCPng Center** - A windows desktop application for management of xcpng hosts,
  just a community project.

<details open>
<summary><H1>Why XCP-ng</H1></summary>

![vs-proxmox](https://i.imgur.com/gLHHxOk.png)

In 2022 Broadcom announced the plan to buy VMware for $60 billion
and the search for ESXi replacement started.

### Proxmox

The absolute front runner candidate.
Debian based, uses KVM for VMs, LXC for containers, native ZFS, native CEPH,
**a huge active community** with dozens of tech youtubers covering it.
A proven solution being out there for **20 years**. Made in Austria.<br>
Tried it and it looked good. Bit complicated, bit unpolished, but very powerful.
The thing is that I never felt drawn to all that power and the very hands-on
phylosophy.
So many options, so much reliance on SSHing into a host to tweak configs directly.
And I do love terminal, but keeping track of all those edits and tweaks...<br>
Generally, it felt like I would be spending **a lot of time learning**
the ins and outs to get the confidence I had with esxi.
And while that's expected, it's still a chore, still an annoyance
and PVE feels fragile and convoluted without that effort put in.<br>
This all made me want to stick longer with esxi and let proxmox cook,
get few more major releases and improvements as VMware refuges start to give
feedback and money. 
  
### XCPng 

Seen it mentioned and had a spare Fujitsu P558 with i3-9100
to test it on. After I wrapped my head around the need to deploy Xen Orchestra
somewhere, it felt like everything was simple and
**it just worked with minimal effort**.
And that apparently is what makes me enthusiastic about stuff.

* Tried to spin **win11 24H2** and it just worked without manually dealing
  with TPM. The VM creation did not feel overwhelming with 19 options per tab,
  which all feel like opportunities for a fuck up.
  Once RDPed in, it felt fast and responsive, without anything weird
  happening and without being send to read 5 pages on how to tweak it to
  improve performance.
* Tried to spin **arch linux**, no problem, zero pauses to read up on what to do
  to get uefi boot working, or some dealings with secure boot. No issues, 
  no complications, no refusal to turn off when the VM was told to turn off.
* Tried **igpu passthrough** in to that arch to test jellyfin in docker..
  and it was just pushing a slider next to the igpu and a restart of the host
  and then in the VM's settings picking the igpu from a list.
* Tried to deploy **opnsense** with a wan side and a lan side networks
  and some windows VM that would be only connected to the opnsense LAN side...
  and it also just straight up worked, no crowded complicated menus and settings,
  no spending lot of time investigating.
* Tried **snapshots** and it was simple. Though they are simple in all
  hypervisors I guess.
* Tried **to backup a VM** to a network share and it seemed ok,
  also **rolling snapshots** just worked with simple scheduling.
  Though in backups there are more options and menus and terms that will
  require reading and testing...  but even this basic intuitive stuff beats
  the free esxi with the ghetto script.
* Tried **migrating a VM from esxi** and it was also ridiculously easy.
  Just giving the ip and credentials, selecting which VM to migrate
  and what kind of system it is.
  Though it took \~5 hours for a 90GB vmkd.

The **webUI** of XO has a bit of an [amateurish vibe](https://i.imgur.com/yuUfUhp.png)
compared to proxmox or esxi, but generally it's clean and simple.
I like that often the info you see can be clicked and edited right then and there.
That's how you change names, the number of cores, the ram, or enlarge disks.<br>
They do work on [a new redesign](https://i.imgur.com/dqOSDsi.png)
that reminds me of opnsense, which is good.

When googling proxmox vs xcpng there seems to be a repeating opinion that 
xcpng is **bit more stable, bit more reliable**. Which obviously sits well with me,
but I also know that guys like [Jeff from Craft Computing](https://www.youtube.com/@CraftComputing/videos)
are deploying proxmox commercially left and right for years, so it must be pretty
stable and I am just glad that I did not find bunch of complains about instability.
 
Now, since that first try I installed xcpng on a few more machines and
the experience there was **not as hurdle-free as that first time**.
There's now [a chapter](#Issues-encountered) where I note issues
I encounter. But still.. that first impression sold me on it pretty hard.

<details open>
<summary><h2>Hypervisors Benchmarks</h2></summary>

![benchmark-symbols](https://i.imgur.com/cPmsLR8.png)

For me the performance is not a deciding factor and I expect it to be
adequate with all of modern hyperviros.
But since I am playing with these I can as well run some benchmarks.

Test machine - ThinkCentre M75q Gen2; ryzen 4350GE; 16+4GB ram;
128GB nvme oem ssd for OS, 500GB sata ssd for VMs<br>
VMs are win10 x64, 8 cores 16GB ram<br>
Tests are run 3+ times, the highest value is noted.

* metal - nothing of note, though it had 20GB or ram compared to VMs 16GB.
* xcpng - storage is ext thin; the guest drivers installed.
* proxmox - cpu - host; storage - [thin LVM](https://i.imgur.com/ZZvQjjm.png);
  virtio drivers installed; followed [this video](https://youtu.be/DnPUkqjvVPw)
  for general setup.
* hyperv - ram was not dynamic

| Win10 WM test     |   metal  | xcpng  | proxmox  | hyperv |
|-------------------|----------|--------|----------|--------|
| cinebench         |  [866](https://i.imgur.com/IEhqZ80.png) | [839](https://i.imgur.com/MT7fPcB.png) | [788](https://i.imgur.com/rpAEkht.png) | [776](https://i.imgur.com/r0FkoRS.png) |
| geekbench         |  [1380 & 4780](https://browser.geekbench.com/v6/cpu/10098173) | [1306 & 4636](https://browser.geekbench.com/v6/cpu/10117642) | [1292 & 4213](https://browser.geekbench.com/v6/cpu/10112746) | [1283 & 4455](https://browser.geekbench.com/v6/cpu/10108799) |
| cristal disk mark |  [41](https://i.imgur.com/3Zniq7s.png) | [22](https://i.imgur.com/g3RbjPQ.png) | [17](https://i.imgur.com/W7qcUf1.png) | [18](https://i.imgur.com/oIWXGMh.png) |
| hdtune            |  [177](https://i.imgur.com/gJFvYsD.png) | [144](https://i.imgur.com/eT9IJVN.png) | [87](https://i.imgur.com/IUooK1t.png) | [113](https://i.imgur.com/70rHbiE.png) |
| iperf             |  [pass](https://i.imgur.com/Clj2PWF.png) | [pass](https://i.imgur.com/c0c51CR.png) | [pass](https://i.imgur.com/sxvxWAU.png) | [pass](https://i.imgur.com/XiM3zVh.png) |
| latency           |  [pass](https://i.imgur.com/V1eExDx.png) | [pass](https://i.imgur.com/iZePi1s.png) | [pass](https://i.imgur.com/ireTL7T.png) | [pass](https://i.imgur.com/9J2VRqP.png) |
| setup overview    |  [info](https://i.imgur.com/qxF8QdT.png) | [info](https://i.imgur.com/aFlVPyP.png) | [info](https://i.imgur.com/t2QBxaj.png) | [info](https://i.imgur.com/2Ip3Blv.png) |

**Cinebench R15** is pretty clear cut. Was run several times on each, even with restarts.<br>
**Geekbench** is nice that it gives a link to detailed results,
has some note about an issue with timers with proxmox.<br>
**Cristal disk mark** shows pretty big differences, picked random read
as the important value for the table.
**HDtune** Burst speed was picked for the table. There also was some cashing going
on hyperv and xcpng, hence after a while sequentials higher than metal.<br>
**iperf** would be more interesting if 2.5gbit or 10gbit nic, maybe there be some
difference.<br>
**DPC latency** test is probably worthless but it's maybe a check if there is
not some weird slowness going on, but all passed. Of note is that the very first
measurment on proxmox had [better values](https://i.imgur.com/43Rw6YH.png)
than metal, but I could not replicate it with later runs.<br>

Likely the performance can be tweaked and improved on some, but I am fine with
xcpng performance so its nice to not needing to bother.<br>
Also of note - the performance of a windows VM,
**is not indicative of a performance of a linux VM**, but I dont feel like doing linux, it would be
probably geekbench + fio and I hate dealing with fio test configs and results.

* *Later experience* - Had MSI cubi miniPC and I planed to install opnsense
straight on metal, but there were issues and I thought because of realtek NICs
and freeBSD of opnsense combo... so virtualization time...<br>
XCPng had terrible throughput of \~50Mbit, Proxmox with virtio managed full \~300Mbit.

</details>

---
---

</details>

# Xen Orchestra

<!-- ![diagram](https://i.imgur.com/EuRpfe1.png) -->

<!-- ![diagram](https://i.imgur.com/cFF1D5p.png) -->

![diagram](https://i.imgur.com/6Q9VJd1.png)

[The official docs](https://docs.xen-orchestra.com/) and 
[the official docs2.](https://docs.xcp-ng.org/management/manage-at-scale/xo-web-ui/)<br>
[Github.](https://github.com/vatesfr/xen-orchestra)<br>
[Ronivay's install script.](https://github.com/ronivay/XenOrchestraInstallerUpdater)

An open source web-based centralized management platform for xcpng servers.<br>
Something like vCenter in the VMware world.

* **XO** - Xen Orchestra - Free version compiled from the source.
* **XOA** - Xen Orchestra Appliance - Paid version. Functional in free mode
  but with limitations.
* **XO Lite** - Xen Orchestra Lite - Running on every host.
  Only provides basic info and simplifies XOA deployment. Under development.

Most non commercial users want to deploy XO which provides
**full functionality while being free.**
It can run either as a VM or a docker container. And either on the xcpng host
or any other machine that can ping the host. The complication is that
**you need XO to deploy XO on to an xcpng host**.

* **docker container** - the most trivial and quick deployment.
* **virtual machine** - there's extra work of spinning up a new debian VM.
* **VM on the xcpng itself** - there's additional extra work of using XOA first.

<details>
<summary><h3>XO in Docker</h3></summary>

![docker-logo](https://i.imgur.com/x25dYmF.png)

[Ronivay's github.](https://github.com/ronivay/xen-orchestra-docker)

The compose here uses ronivay's image and is a variation of their compose.

The changes made - switching **from volumes to bind mounts** and not mapping 
port `80` to docker host port `80`, but **just using expose** to document the port
webGUI is using. Reason is that theres an expectation of running a reverse proxy.
If no reverse proxy then go with
[ronivay's port mapping](https://github.com/ronivay/xen-orchestra-docker/blob/master/docker-compose.yml).

`compose.yml`
```yml
services:

  xen-orchestra:
    image: ronivay/xen-orchestra:latest
    container_name: xen-orchestra
    hostname: xen-orchestra
    restart: unless-stopped
    env_file: .env
    stop_grace_period: 1m
    expose:
        - "80"         # webGUI
    cap_add:           # capabilities are needed for NFS/SMB mount
      - SYS_ADMIN
      - DAC_READ_SEARCH
    # additional setting required for apparmor enabled systems. also needed for NFS mount
    security_opt:
      - apparmor:unconfined
    volumes:
      - ./xo_data:/var/lib/xo-server
      - ./redis_data:/var/lib/redis
    # these are needed for file restore.
    # allows one backup to be mounted at once which will be umounted after some minutes if not used (prevents other backups to be mounted during that)
    # add loop devices (loop1, loop2 etc) if multiple simultaneous mounts needed.
    devices:
     - "/dev/fuse:/dev/fuse"
     - "/dev/loop-control:/dev/loop-control"
     # - "/dev/loop0:/dev/loop0"

networks:
  default:
    name: $DOCKER_MY_NETWORK
    external: true
```

`.env`
```bash
# GENERAL
DOCKER_MY_NETWORK=caddy_net
TZ=Europe/Bratislava

# XO
HTTP_PORT=80
```

Caddy is used for reverse proxy, details
[here](https://github.com/DoTheEvo/selfhosted-apps-docker/tree/master/caddy_v2).</br>

`Caddyfile`
```php
xo.{$MY_DOMAIN} {
    reverse_proxy xen-orchestra:80
}
```

---
---

</details>

<details>
<summary><h3>XO in a VM</h3></summary>

![debian-logo](https://i.imgur.com/d732Me7.png)

If you got an another server with a hypervisor, or if you on your desktop
run a virtualbox or a hyperv, or for the final VM deployment of XO on xcpng...

* Spin up a **new debian virtual machine**, click through the regular install.
* **clone the github repo** with the install script<br>
  `git clone https://github.com/ronivay/XenOrchestraInstallerUpdater.git`
* go inside<br>
  `cd XenOrchestraInstallerUpdater`
* make a copy of the **sample config**.<br>
  `cp sample.xo-install.cfg xo-install.cfg`
* **run the install script**<br>
  `sudo ./xo-install.sh`

More discussion about the process
[here.](https://forums.lawrencesystems.com/t/how-to-build-xen-orchestra-from-sources-2024/19913)

---
---

</details>   

<details>
<summary><h3>XO on XCPng itself</h3></summary>

![web-install](https://i.imgur.com/HFwezjG.png)

*Note:* **the videos** showcasing the process are in [the last chapter](#Videos).

The easiest way is to **first deploy the paid XOA** and use that to deploy XO.


  * In a browser, go to the xcpng host **IP address**,
    top right corner you see **Deploy XOA**<br>
    * btw, one can also initialize this deployment by creating an account
      on [xen-orchestra.com](https://xen-orchestra.com/)
      and under the account find *"XOA quick deploy - Deploy now".*
  * Click through the setup.
  * Login to XOA at the ip address this new VM got.
  * Follow [The Basics](#The-Basics) section to:
    * create **iso storage** and upload iso
    * spin up a **new VM** with debian or ubuntu or centos stream
    * git clone XO install script repo, rename the config file,
      **execute the install script**<br>
      or alternatively setup the new VM **as a docker host** and deploy XO
      as a container there
    * **add xcpng host** as a server in to the XO
    * delete XOA virtual machine

---
---

</details> 

XO deployment is an extra step compared to other hypervisors, but if you ever
get more servers this approach **starts to make sense** -  not thinking
about running the management tool on the thing it manages. That the hosts are
thought of as replaceable cogs in a bigger machine...<br>
But yeah, it also means it is an extra work for - *"my first home server"*
types of deployments.

#### Some aspects of XO

* Once VMs are up and running, XO is not required for them to function. But you
  **lose some functionality** if you would turn it off or disconnect.
  * **Backups schedule and their execution.**<br>
    XO is what manages backups, even the data of the VMs that are being backed up
    flow through the XO during a backup job if it's going to a network share.
    Theres even [XO Proxy](https://xen-orchestra.com/blog/xen-orchestra-proxy/)
    to be there with the VMs on-site while the main management XO is wherever...
  * **Metrics monitoring**.<br>
    Can't look up cpu load from the last week if XO was not there to record it.
  * **HA - High Availability** - ...like duh, something needs to orchestrate it...
* XO is the free version, compiled from the source, **nagging notices**
  about not having subscription are something thats just there occasionally.

# The Basics

### XCPng Host Installation

[The official docs.](https://docs.xcp-ng.org/installation/install-xcp-ng/)

[Download](https://docs.xcp-ng.org/releases/) the latest release ISO.
Boot the ISO, I use ventoy, click through the installation...<br>
All is pretty straight forward. The official docs have pretty hand holding
instructions too.

After reboot, we are shown the basic info menu, similar to esxi but better.
I really like the look with all the info and all the functionality.
This menu can be open even when SSH in, with `xsconsole` command.

Maybe now it's a good time to set **static IP** address for this xcpng host.

![xcpng-console-menu](https://i.imgur.com/Iu0bdmh.png)


<!-- ![login-pic](https://i.imgur.com/EuFbbrn.png) -->
<!-- ![login-pic](https://i.imgur.com/VcmDmNE.png) -->


### The First login in to XO

* `admin@admin.net` // `admin`
* change login email and password<br>
  `Settings` > `Users`
* **Turn off the filters** for VMs, as by default only the *running VMs* are shown<br>
  user icon in the left bottom corner > 
  `Customize filters` > VM - Default filter = None

### Add Server

<!-- ![iso-sr-250px](https://i.imgur.com/x3NM9Cj.png) -->

* `New` > `Server`
* label whatever
* ip address 
* `root` / password set during the xcpng installation
* slider `Allow Unauthorized Certificates` - True

### Updates

* Home > Hosts > your_host > Patches

### Create DVD ISO storage

![iso-sr-250px](https://i.imgur.com/3USpQpq.png)

[The official docs.](https://xcp-ng.org/blog/2022/05/05/how-to-create-a-local-iso-repository-in-xcp-ng/)

##### Local Storage Repo

* New > Storage
* Select your_host
* Set the name and the description
* Select storage type: `ISO SR: Local`
* Path: `/media` if you are ok putting ISOs on the boot disk
  * Alternative is to ssh in and look around for a path to another drive,
    usually it's somewhere in `run/sr-mount/`
* Create
* To upload an ISO<br>
  Import > Disk > To SR: `whatever_named`<br>
  It knows the type of the storage repo and allows upload of ISOs

If `/media` is selected the storage repo is created on a 18GB root partition.<br>

<details>
<summary><h5>NFS share</h5></summary>

* Have an NFS share,
 [I use truenas scale](https://github.com/DoTheEvo/selfhosted-apps-docker/tree/master/trueNASscale#nfs-share)
* New > Storage
* Select your_host
* Set the name and the description
* Select storage type: `NFS ISO`
* Server: ip_address_of_the_nfs_share
  * search icon
* select detected path
* Create

---
---

</details>

### Virtual Machines creation

![vms-250px](https://i.imgur.com/eD73qvp.png)

[The official docs.](https://docs.xcp-ng.org/vms/)

Spinning a new VM is easy and quick.<br>
Preconfigured templates take care of lots of settings. They are in json
format and can be browsed in `/usr/share/xapi/vm-templates`.<br>

* New > VM
* Select template
* vCPU, RAM, socket
* ISO
* Network default
* Disk - change the name, set the size

### Guest Tools

![agent-detect](https://i.imgur.com/DxP0iFx.gif)

[The official docs.](https://docs.xcp-ng.org/vms/#%EF%B8%8F-guest-tools)

Consist of two components and 
you **absolutely** want to make sure you got both working properly.<br>
The **info is in the General tab** of every virtual machine.

* Kernel Paravirtualization **Drivers** - improve performance, usually I/O.
  * HVM - no drivers
  * PVHVM - drivers present
* Management **Agent** - better guest management and metrics reporting.
  * Management agent not detected
  * Management agent detected

<details>
<summary><h4>Windows</h4></summary>

[The official docs.](https://xcp-ng.org/docs/guests.html#windows)

The above linked official docs tell well the details.

* **citrix closed source VM windows tools**<br>
  [Link to download from citrix site](https://www.xenserver.com/downloads)<br>
  `XenServer VM Tools for Windows 9.4.0` - was in January 2025<br>
  The **recommended go-to** way to get drivers and agent for windows VMs.
  During install it asks if **automatic updates** for agent and drivers.
* **xcpng open source VM windows tools**<br>
  [https://github.com/xcp-ng/win-pv-drivers/releases](https://github.com/xcp-ng/win-pv-drivers/releases)<br>
  `XCP-ng Windows PV Tools 8.2.2.200-RC1` - as v9 seems still under slow development.

Theres also a VM option, to get **the drivers** through **windows updates**,
but reading the docs, it's just a driver and the VM **still needs the agent**,
so you would still be installing agent... so it's not worth the bother.

---
---

</details>

<details>
<summary><h4>Linux</h4></summary>

[The official docs.](https://docs.xcp-ng.org/vms/#%EF%B8%8F-guest-tools)

Again, the linked docs tell well all the details.<br>
**The drivers** are in linux kernel, so one only needs the agent.

For my go-to archlinux I just

* `yay xe-guest-utilities-xcp-ng`
* `sudo systemctl enable --now xe-linux-distribution.service`

For occasianal debian install it's just as the docs say

* XO comes with `XCPng-Tools` iso, mount that in to virtual dvd in General tab
  of a VM
* restart the VM
* `sudo mount /dev/cdrom /mnt`
* `sudo bash /mnt/Linux/install.sh`
* reboot and unmount 

---
---

</details>

# Backups

![backup-diagram](https://i.imgur.com/cFF1D5p.png)

* [The official docs](https://docs.xen-orchestra.com/backup) and
  [the official docs2.](https://docs.xcp-ng.org/management/backup/)
* [Lawrence Systems video.](https://youtu.be/weVoKm8kDb4)

Backups are important enough that the official docs should be the main
source of information. Stuff here are just some highlights, notes. 

Be aware - **Xen Orchestra is what schedules and executes backups.**<br>
XO must be running with xcpng hosts, its not a fire and forget deployment.

### Backup Jobs Types

At the moment I just played with rolling snapshots, backups, and delta backups.

* VM Backup & Replication
  * **Rolling Snapshot**<br>
    Takes a snapshot at schedule. Retention is set in the schedule section.
  * **Backup**<br>
    Snapshot of a VM and then exports to a remote location.
    Full size every time, so lot of space, bandwidth and time is used.
  * **Delta Backup**<br>
    Incremental backups of only changes against the initial full back<br>
    * [CBT](https://xen-orchestra.com/blog/xen-orchestra-5-96/)
    \- Changed Block Tracking<br>
    a new and a better way to do incremental backups,
    but its not yet battle tested.
  * **Disaster Recovery**<br>
    Full replication. The backup of the VM can be started immediately,
    no restoration.
  * **Continuous Replication**<br>
    Replication but through incremental changes.
* VM Mirror Backup
  * **Mirror full backup**<br>
    backup of a backup repo
  * **Mirror incremental backup**<br>
    backup of a backup repo through incremental changes.
* XO config & Pool metadata Backup
* Sequence

#### Smart mode

Gives ability to more broadly target VMs for backup jobs.<br>
Instead of just selecting VMs manually.. it can be that all running VMs
on all hosts get rolling snapshots. Or the ones tagged as `production`
will have nightly full backup,...  

#### Health check

[Here's](https://youtu.be/A0HTRF3dhQE) Lawrence Systems video on backups
that are automatically tested. A VM is restored at a host of choice,
booted without network and theres a check that guest tools agent starts.
If all that happens, the backup is marked as healthy and the VM is destroyed.

#### Veeam Support

[Seems](https://forums.veeam.com/veeam-backup-replication-f2/xcp-ng-support-t93030-60.html#p531802)
theres a prototype and a praise of xen api from veeam devs.<br>
Though that does not mean the management will decide to create and support
xen veeam edition.

### Remotes

Kinda weird how for backups you are creating remotes and not storage,
like its some type of different category even when I am doing same nfs..

`showmount -e 192.168.1.150`  - a handy command showing nfs shares and paths

* Settings > Remotes
* Local or NFS or SMB or S3
* IP address
* port - can be left empty
* path of the share
* custom options - can be empty 

### Create a backup job

* Backup > New > VM Backup & Replication

![backup-job-report](https://i.imgur.com/AudRSUP.png)

### Backup reports

First setup email server for notifications

* Settings > Plugins > `transport-email`<br>
* I use a free Brevo account for an smtp server - 300 emails a day.

Then in backup job settings 

* Report when - always | skipped or failure
* Report recipient - set an email and you have to **press the plus** sign
* Save 

# Advanced Concepts

## Passthrough

![passthrough-pic](https://i.imgur.com/nLNT9iH.gif)

When you want to give a virtual machine **direct full hardware access**
to some device.<br>
Be aware that once passthrough is setup it's tight to hardware addresses and
hardware changes first **require disabling the passthrough**, or your xpcng
might not boot or devices might be stuck hidden.

#### intel igpu passthrough

* On the server host
  * Home > Hosts > your_host > Advanced > PCI Devices<br>
    Enable slider next to VGA compatible controller
  * Reboot the host, go check if the slider is on
* On the Virtual Machine
  * Home > VMs > your_VM > Advanced ><br>
    At the end a button - `Attach PCIs`, there pick the igpu listed.

In VM you can check with `lspci | grep -i vga`

Tested with jellyfin and enabled transcoding,
monitored with btop and intel_gpu_htop.

<details>
<summary><h5>The old way - cli passthrough</h5></summary>

[Lawrence video.](https://www.youtube.com/watch?v=KIhyGvuCDcc)

* ssh in on to xcpng host
* `lspci -D` list the devices that can be passthrough
* pick the device you want, note the HW address at the begining,
  in this case it was `0000:00:02.0`
* hide the device from the system<br>
  `/opt/xensource/libexec/xen-cmdline --set-dom0 "xen-pciback.hide=(0000:00:02.0)"`
  * be aware, the command is overrwriting the current blacklist,
    so for multiple devices it would be<br>
    `/opt/xensource/libexec/xen-cmdline --set-dom0 "xen-pciback.hide=(0000:00:02.0)(0000:00:01.0)"`
* reboot the hypervisor
* can use command `xl pci-assignable-list` to check device that can be passthrough    

---
---

</details>

#### amd igpu passthrough

No luck so far.

`udevadm info --query=all --name=/dev/dri/renderD128`<br>
`lspci -vvv -s 00:08.0` - "00:08.0" being physical address shown in the udevadm
`dmesg | grep -i amdgpu` - if loaded correctly


## Pools

![pool-join-pic](https://i.imgur.com/jXaIqHb.png)

For easier management on larger scale.<br>
Pools remove some duplicitous effort when setting up shared storage, or networks,
or backups.
Allow for easier/faster live migration of VMs or for automatic load balancing.
Safer updates of the hosts and easier scale up of the compute power by
adding more hosts.<br>
To join a pool, the hosts must have similar CPU,
for examle you can no mix amd and intel, but not sure how similar till you
get message - *"Failed: The hosts in this pool are not homogeneous. CPUs differ."*

* All hosts are masters in their own pool, **pick one** that will be the master<br>
  **rename it's pool** to something more specific
* for the machines that will be **joining** that pool
  * ssh in or get to the **console** of the host<br>
   Home > Hosts > your_host > Console 
  * `xsconsole` to get the core menu
    * Resource Pool Configuration > **Join a Resource Pool**
    * give the hostname of the pool master
    * root and password

I only had few machines in a pool to check it out, do some testing.<br> 
Might add more info in the future.

## Monitoring

![prometheus-monit](https://i.imgur.com/CMlikyw.png)

<details>
<summary><h3>Prometheus + Grafana monitoring</h3></summary>

To get metrics and setup alerts.<br>
Details on general prometheus + grafana deployment
[here.](https://github.com/DoTheEvo/selfhosted-apps-docker/tree/master/prometheus_grafana_loki)

* [MikeDombo Prometheus exporter](https://github.com/MikeDombo/xen-exporter)
* [Grafana dashboard - id - 16588](https://grafana.com/grafana/dashboards/16588-xen/)

`compose.yml`
```yml
services:

  xen01:
    image: ghcr.io/mikedombo/xen-exporter:latest
    container_name: xen01
    hostname: xen01
    restart: unless-stopped
    environment:
      - XEN_HOST=10.0.19.62
      - XEN_USER=root
      - XEN_PASSWORD=aaaaaa
      - XEN_SSL_VERIFY=false

networks:
  default:
    name: caddy_net
    external: true
```

`prometheus.yml`
```php
scrape_configs:
  - job_name: 'xenserver'
    static_configs:
      - targets: ['xen01:9100']
```

---
---

</details>

## Networking and VLANs in XO

* [The official docs](https://docs.xcp-ng.org/networking/)
* [VLAN trunking](https://docs.xcp-ng.org/guides/VLAN-trunking-vm/)
* Lawrence Systems - [video1](https://youtu.be/z-cSXuQgDrk); [video2](https://youtu.be/H5PJ_tHQlZk)

Reading the official docs is a must.<br>
But some highlights

* A network is basicly a virtual switch.
* You want to set up networking in the `pool` not on the `host`
* For simple vlans you create a new network named whatever, add it vlan tag.<br>
  Thats it, now you can change the network in a VM's network 








## opnsense or pfsense as a VM in xcpng

![tx-checksumming-off](https://i.imgur.com/brpHNA5.png)

[The official docs.](https://docs.xcp-ng.org/guides/pfsense/)

Read the above docs.<br>

The hihglight is to use UEFI boot to avoid potentially longer boot times.

**The most important** bit of info is to **disable TX Checksum Offload**<br>
[Here](https://github.com/DoTheEvo/selfhosted-apps-docker/tree/master/opnsense#xcp-ng)
will be more detailed example of opnsense deployment.

## Notes on some concepts

### Storage

[The official docs.](https://docs.xcp-ng.org/storage/)

The above docs link gives good overview.
I plan to keep it simple.

* ext4 for local storage
* nfs for network shares

Various file types encountered.

* `VDI` - Virtual Disk Storage - a concept, not an actual file type
* `.vhd` - A file representing a virtual disks and snapshots.
* `.xva` - An archive of a VM, used for backups.
* `.iso` - Bootable dvd image, usually for OS installation.

### Virtualization Models

![xen-virt-modes](https://i.imgur.com/FagP99J.png)

[Xenproject wiki](https://wiki.xenproject.org/wiki/Understanding_the_Virtualization_Spectrum)
has a good article on these, especially with bit of history.

* **PV** - Paravirtualization<br>
  The oldest way, bypassing need for emulation
  of hardware by having the guest OS aware of being in a VM, running with
  a modified kernel and using a specific hypervisor API.
* **HVM** - Hardware Virtual Machine<br>
  Full emulation of hardware using hardware support - intel VT-x | AMD-V
* **PVHVM** -   Hardware virtualization with paravirtualization drivers enabled<br>
  HVM performed worse in some aspects, especially I/O, installing the PV
  emulated device drivers bypasses some qemu overhead, improving the performance.
* **PVH** - Paravirtualization-on-HVM<br>
  Further performance improvements and reduced complexity. Completely drops
  the need for qemu for the emulation of hardware. Not yet really used.

# Issues encountered

* **A VM with "Generic Linux UEFI" preset 
  [failed to boot from arch ISO](https://i.imgur.com/cnnlBtJ.png)**<br>
  Weird issue. Seems the cause is that the ISO SR was created in `/media`
  on a small OEM nvme ssd that came with that miniPC.
  The thing is that I had 3 lenovo miniPCs at that time and every single
  one of them had this issue. Debian 12 ISO and template also had that issue.<br>
  Any change to the setup **solved the problem**.
  Replacing the ssd with a larger brand-name nvme ssd;
  creating ISO SR on a different drive; switch to a sata ssd;
  using nfs share for ISOs; switching to bios;...<br>
  Probably some weird quirk with uefi and ext3 and a small nvme ssd or something.
* igpu **passthrough**
  * ryzen **4350GE** is not working at all, ThinkCentre M75q Gen 2.
  * **i5-8400T** had poor performance, ThinkCentre M720q.<br>
  I am starting to wonder if my initial test with the i3-9100 really worked
  as well as I remember it.<br>
  Will keep testing as I get hands on some other machines.
  * **i3-10105T** in a Dell OptiPlex 3080.<br>
    Passthrough worked straight up without a hitch transcoding 8 streams.
  * **i3-7100** in a Fujitsu D556.<br>
    Passthrough worked transcoding 8 streams, cpu usage weirdly high.
* **OPNsense** had poor network performance under xcpng caused by virtual
  xen NICs having poor drivers on freeBSD. Would need intel NIC passthrough,
  or its better to use proxmox with its virtio NICs which perform much better.

<details>
<summary><h1>Videos</h1></summary>

How to deploy XOA on a freshly installed xcpng host and then using XOA
to deploy debian VM and run script that installs XO.

[01-XO-lite-Deploy-XOA.mp4](https://github.com/user-attachments/assets/66fdb443-4218-4eb2-b07c-23a1e95d2830)

[02-XOA-firstlogin-new-vm.mp4](https://github.com/user-attachments/assets/b3d5083c-be5b-4cad-bae9-834626a15346)

[03-debian-install-speed.mp4](https://github.com/user-attachments/assets/2195474a-3ad5-493b-a42f-9b852155ba0f)

[04-XO-install-script.mp4](https://github.com/user-attachments/assets/2ccc6824-b7a5-4ebb-a57e-64527c931e96)

[05-XO-firstlogin-adding-host-remove-XOA.mp4](https://github.com/user-attachments/assets/a88efff1-4547-4140-8664-196292e6b800)

---
---

</details>

