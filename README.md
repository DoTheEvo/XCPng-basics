# XCP-ng

###### guide-by-example

![logo](https://i.imgur.com/FBH2aII.png)

# Purpose & Overview

A virtualization platform build around
Xen a type 1 **hypervisor**.<br>
An alternative to vmware esxi/vsphere, or proxmox, or hyper-v.

[Xen](https://en.wikipedia.org/wiki/Xen)
is an open source project, developed by **Linux Foundation** with
backing from intel, amd, arm, aws, google, alibaba cloud, huawei,...
Citrix owned Xen until 2013, when they made it open-source..
In 2022 Citrix was sold for $16 billion to a private equity group.

[XCP-ng](https://en.wikipedia.org/wiki/XCP-ng) itself started on kickstarter
as a fork of XenServer, which with version 7.0 closed-source some of it's components.
The first release was in 2018, but Vates - the company behind it
worked on Xen Orchestra since 2012. They are located in France
and have \~40 employees.

* **xen** - The hypervisor, developed by Linux Foundation.
* **xcpng** - A single purpose linux distro preconfigured with xen, centos userspace.
* **XO** - Xen Orchestra - a web interface for centralized management
  of xcpng hosts,<br>
  usually deployed as a container or a virtual machine.
* **XOA** - Xen Orchestra Appliance - a paid version of XO with full support
  and some extra features like [XOSTOR](https://vates.tech/xostor/)
  through webGUI.
* **xcpng center** - A windows desktop application for management of xcpng hosts,
  a community project, was abandonware but it has
  [a new maintainer](https://github.com/xcp-ng/xenadmin).

<details>
<summary><H1>Why XCP-ng</H1></summary>

![vs-proxmox](https://i.imgur.com/gLHHxOk.png)

In 2022 Broadcom announced the plan to buy VMware for $60 billion
and the search for ESXi replacement started.

* **Proxmox** - The absolute front runner, debian based, **huge active community**,
  dozens of tech youtubers, a proven solution being out there for **20 years**.
  Made in Austria.<br>
  Tried it and it looked good. Bit complicated, bit unpolished, but very powerful.
  The thing is that I never felt drawn to it. It felt like I would be spending
  a lot of time learning the ins and outs to feel confident about it
  and while that's expected, it's still a chore. A chore that's much easier
  when theres enthusiasm for a project, but for some reason I did not feel it.<br>
  This made me want to stick longer with esxi and let proxmox cook,
  get few more major releases and improvements as vmware refuges start to give
  input and money.
  
* **XCP-ng** - Seen it mentioned and had a spare Fujitsu P558 with i3-9100
  to test it on. After I wrapped my head around the need to deploy Xen Orchestra
  somewhere else, it felt like everything was simple and
  **it just worked with minimal effort**.
  That apparently is what makes me enthusiastic about stuff.

  * Tried to spin **win11 24H2** and it just worked without manually dealing
    with TPM. The VM creation did not feel overwhelming with 19 options
    and settings and choices which are all opportunities for a fuck up.
    Once RDPed in, it felt snappy and responsive without anything weird happening.
  * Tried to spin **arch linux**, no problem, zero pauses to read up on what to do
    to get uefi boot working, or dealing with secure boot or whatever.
    No complications.
  * Tried **igpu passthrough** in to that arch to test jellyfin in docker..
    and it was just pushing a slider next to the igpu and a restart of the host
    and then in the VM settings picking the igpu from a list.
  * Tried to deploy **opnsense** with wan side and lan side networks
    and some win VM that would be only connected to the opnsense LAN side..
    and it also just straight up worked, no complicated menus and options,
    no spending time reading or googling.
  * Tried **snaphots** and it was simple. Though if one makes snapshot with memory
    the VM is paused during snapshotting.
  * Tried **to backup a VM** to a network share and it seemed ok,
    also **rolling snapshots** just worked with simple scheduling.
    Though in backups there are more options and menus and terms that will
    require reading and testing...  but even this basic intuitive stuff beats
    the free esxi with the ghetto script.
  * Tried **migrating a VM from esxi** and it was also ridiculously easy.
    Just giving the ip and credentials, selecting which VM to migrate
    and what kind of system it is.
    Though it took \~5 hours for a 90GB vmkd, so that was not that ideal.

The **webUI** of XO has a bit of an [amateurish vibe](https://i.imgur.com/yuUfUhp.png)
compared to proxmox or esxi, but generally it's clean and simple.
I like that often the info you see can be clicked and edited right then and there.<br>
They work on [something new](https://i.imgur.com/dqOSDsi.png) with XO v6 that
reminds me of opnsense, which is good.

When googling proxmox vs xcpng there seems to be a repeating opinion that 
xcpng is **bit more stable, bit more reliable**. Which obviously sits well with me,
but I also know that guys like [Jeff from Craft Computing](https://www.youtube.com/@CraftComputing/videos)
are deploying proxmox commercially left and right for years, so it must be pretty
stable and I am just glad that I did not find bunch of complains about instability.
 
Now, since that first try I installed xcpng on a few other machines and
the experience there was **not as hurdle free as that first time**.
But still.. that first impression sold me on it pretty hard.

<details>
<summary>the issues encountered so far</summary>
* **A VM with "Generic Linux UEFI" preset 
  [failed to boot from arch ISO](https://i.imgur.com/cnnlBtJ.png)**<br>
  Weird issue. Seems the cause is that the ISO SR was created in `/media`
  on the boot drive which was a small OEM nvme ssd that came with that miniPC.
  The thing is that I had 3 miniPCs at that time and every single one of them
  failed at this task.<br>
  Any change to the setup **solved the problem**.
  Replacing the ssd with a different larger brand-name nvme ssd;
  creating ISO SR on a different drive; switch to a sata boot ssd;
  using nfs share for ISOs; using bios template instead of uefi;...<br>
  Probably some weird quirk with ext3 and a small nvme ssd or something.
* igpu **passthrough** of ryzen 4350GE is not working at all, ThinkCentre M75q Gen 2.
* igpu **passthrough** of i5-8400T had poor performance, ThinkCentre M720Q.<br>
  I am starting to wonder if my initial test with i3-9100 of the passthrough
  really worked as well as I remember it working.<br>
  Will keep testing when I get some more machines with intel igpu.

</details>

</details>

# Xen Orchestra

<!-- ![diagram](https://i.imgur.com/EuRpfe1.png) -->

![diagram](https://i.imgur.com/MumtDzU.png)

* **XO** - Xen Orchestra - free version
* **XOA** - Xen Orchestra Appliance - a paid version of XO
* **XO Lite** - preinstalled on every host, just informative and eases XOA
  deployment, work in progress

Unlike esxi or proxmox, going with a browser to the IP of an xcpng host
**shows just [XO-Lite,](https://docs.xcp-ng.org/management/manage-locally/xo-lite/)**
which is not very useful.

To get **full funcionality while keeping it free** one needs to deploy
XO - Xen Orchestra. Either as a VM or a docker container. And either on the xcpng
host or whatever other machine that can ping the host.
The complication is that **you need XO to deploy XO on to an xcpng host**.

If you have another hypervisor or a docker host it's trivial and quick.

* Spin up a **new debian virtual machine**,
  run [xcpng install script](https://github.com/ronivay/XenOrchestraInstallerUpdater).
  Done.<br>
  Some extra instructions [here](https://forums.lawrencesystems.com/t/how-to-build-xen-orchestra-from-sources-2024/19913).
* Spin up a **docker host** as VM, run xcpng container.<br>
  The details are in the docker section below.

But if the machine running xcpng is **your first and only server**,
you gotta go with a virtualbox or hyper-v on your desktop/notebook, deploy
XO there.. and then use that XO to deploy new XO on to the xcpng host itself.

It seems complicated and maybe it is, but if you ever move towards more servers
**it starts to make sense** -  not having your centralized management tool running
on the thing it manages.<br>
But yeah, it also means it is less friendly towards - *"my first home server"*
deployments. We will see what XO Lite brings in the future.

<details>
<summary><h4>XO running as a docker container</h4></summary>

* [ronivay github](https://github.com/ronivay/xen-orchestra-docker)

Note the port `80` is just exposed, not mapped on to the host.
Theres expectations of having a reverse proxy..
if not just change `expose` to `ports` and set some mapping, like `2280:80`

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

### Some aspects of XO

* XO is not needed for VMs to function, but you **lose some functionality**
  if you turn if off or disconnect it after the VMs are setup.
  * **Backups schedule and their execution.**<br>
    XO is what manages backups, even the data of the VMs that are being backed up
    flow through the XO during a backup job if it's going to a network share.
    Theres even [XO Proxy](https://xen-orchestra.com/blog/xen-orchestra-proxy/)
    to be there with the VMs on-site while main management XO is wherever...
  * **Metrics monitoring**.<br>
    Can't look up what was the cpu load last week if XO was not there to record it.
  * **HA - High Availability** - ...like duh
* XO is the free version, compiled from the source, **nagging notices**
  about not having subscription are something thats just there occasionally.
* \-

# XCP-ng Host Installation

![xcpng-console-menu](https://i.imgur.com/Iu0bdmh.png)

[Official docs.](https://docs.xcp-ng.org/installation/install-xcp-ng/)

[Download](https://docs.xcp-ng.org/releases/release-8-3/) the latest release ISO.
Boot the ISO, I use ventoy, click through the installation...<br>
All is pretty straight forward. The official docs have pretty hand holding
instructions too.

After reboot, we are shown a basic info menu, similar to esxi but better.
I really like the look with all the info and all the options.
This menu can be open even when SSH in, with `xsconsole` command.

# Basic Setup

![login-pic](https://i.imgur.com/EuFbbrn.png)
<!-- ![login-pic](https://i.imgur.com/VcmDmNE.png) -->

### The First login in to XO

* `admin@admin.net` // `admin`
* change login email and password<br>
  `Settings` > `Users`
* Turn off the filters for VMs, as by default only the *running VMs* are shown<br>
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

[Official docs.](https://xcp-ng.org/blog/2022/05/05/how-to-create-a-local-iso-repository-in-xcp-ng/)

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

If `/media` is seletected the storage repo is created on a 18GB root partition.<br>

<details>
<summary><h5>NFS share</h5></summary>

* Have an NFS share,
 [I use truenas scale](https://github.com/DoTheEvo/selfhosted-apps-docker/tree/master/trueNASscale#nfs-share)
* New > Storage
* Select your_host
* Set teh name and the description
* Select storage type: `NFS ISO`
* Server: ip_address_of_the_nfs_share
  * search icon
* select detected path
* Create

---
---

</details>

# Virtual Machines creation

![vms-250px](https://i.imgur.com/eD73qvp.png)

[Official docs.](https://docs.xcp-ng.org/vms/)

Spinning a new VM is easy and quick.<br>
Preconfigured templates take care of lots of settings. They are in json
format and can be browsed in `/usr/share/xapi/vm-templates`.<br>

* New > VM
* picking template
* vCPU, RAM, socket
* iso
* network default
* disks - change the name, set the size

### Guest Additions

#### Windows

[The official docs](https://xcp-ng.org/docs/guests.html#windows)

The above linked official docs tell well the details.

* citrix closed source<br>
  [Link to download from citrix site](https://www.xenserver.com/downloads)<br>
  `XenServer VM Tools for Windows 9.4.0` - was in december 2024
* github open source version<br>
  [https://github.com/xcp-ng/win-pv-drivers/releases](https://github.com/xcp-ng/win-pv-drivers/releases)<br>
  `XCP-ng Windows PV Tools 8.2.2.200-RC1` - as v9 seems still under development

Theres also a VM option, to get driver through windows updates,
but reading docs, it's just a driver and the VM still needs an agent,
so why mix stuff or bother.

#### Linux

[The official docs](https://docs.xcp-ng.org/vms/#%EF%B8%8F-guest-tools)

Again, the linked ocs tell well all the deails.<br>
For my go-to arch linux I just

* `yay xe-guest-utilities-xcp-ng`
* `sudo systemctl enable --now xe-linux-distribution.service`

<details>
<summary><h3>opnsense as a VM in xcpng</h3></summary>

![tx-checksumming-off](https://i.imgur.com/brpHNA5.png)

[Official docs.](https://docs.xcp-ng.org/guides/pfsense/)

The most important bit of info is to disable `TX Checksum Offload`

[Here](https://github.com/DoTheEvo/selfhosted-apps-docker/tree/master/opnsense#xcp-ng)
will be more detailed info on an example deployment.

---
---

</details>

## Passthrough

![passthrough-pic](https://i.imgur.com/nLNT9iH.gif)

When you want to give a virtual machine direct full hardware access to some device.<br>
Since v8.3 it is very easy to do through webGUI.

#### intel igpu passthrough

* On the server host
  * Home > Hosts > your_host > Advanced > PCI Devices<br>
    Enable slidder next to VGA compatible controller
  * Reboot the host, go check if the slider is on
* On the Virtual Machine
  * Home > VMs > your_VM > Advanced ><br>
    At the end a button - `Attach PCIs`, there pick the igpu listed.

In VM you can check with `lspci | grep -i vga`

Tested with jellyfin and enabled transcoding,
monitored with btop and intel_gpu_htop.

<details>
<summary><h5>The old way - cli passthrough</h5></summary>

[lawrance video](https://www.youtube.com/watch?v=KIhyGvuCDcc)

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

After reboot of the VM I had igpu in and successfully used it in jellyfin.

### amd igpu passthrough

`udevadm info --query=all --name=/dev/dri/renderD128`
`dmesg | grep -i amdgpu` - if loaded correctly

---
---

</details>

# Backups 

* [Official docs](https://docs.xen-orchestra.com/backup)
* [Official docs2](https://docs.xcp-ng.org/management/backup/)
* [Lawrence Systems video](https://youtu.be/weVoKm8kDb4)

Backups are important enough that the official docs should be the main
source of information. Stuff here are just some highlights, notes. 

### Backup Jobs Types

* VM Backup & Replication
  * **Rolling Snapshot**<br>
    Takes a snapshot at schedule. Retention is set in the schedule section.
  * **Backup**<br>
    Snapshots and exports VM to a remote location.
    Full size every time, so lot of space, bandwith and time is used.
  * **Delta Backup**<br>
    Incremental backups of only changes against the initial full back<br>
    [CBT](https://xen-orchestra.com/blog/xen-orchestra-5-96/)
    \- Changed Block Tracking - a new way to do incremental backups
  * **Disaster Recovery**<br>
    Full replication. Backup of the VM can be started immediately, no restoration.
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
Instead of just selecting manually.. it can be that all running VMs
on all hosts get rolling snapshots. Or the ones tagged as `production`,
or the ones that are on a specific host.

#### Veeam Support

[Seems](https://forums.veeam.com/veeam-backup-replication-f2/xcp-ng-support-t93030-60.html#p531802)
theres a prototype and a praise from veeam developers for quality of xen api.

### create a remote

* Settings > Remotes
* NFS or Local or SMB
* ...

### Create a backkup job

* Backup > New > VM Backup & Replication

# Pools

![pool-join-pic](https://i.imgur.com/jXaIqHb.png)

For easier management on larger scale.<br>
Removes duplicitous effort when setting up network shares or backups.
Allow for easier live migration of VMs or for automatic load balancing.
Safer updates of the hosts and easier scale up of the compute power by
adding more hosts.

* All hosts are masters in their own pool, pick one that will be the actual master<br>
  rename it's pool to something more specific
* for the machines that will be joining that pool
  * ssh in or get to the console of the host<br>
   Home > Hosts > your_host > Console 
  * `xsconsole` to get the core menu
    * Resource Pool Configuration > Join a Resource Pool
    * give hostname of the existing pool master
    * root and password


# Monitoring xcpng 

<details>
<summary><h3>Prometheus + Grafana monitoring</h3></summary>

![prometheus-monit](https://i.imgur.com/CMlikyw.png)

Details on general prometheus + grafana + loki deployment
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

# Notes on some concepts

## Storage

[Official docs.](https://docs.xcp-ng.org/storage/)

The above docs link gives good overview. To keep it simple 

* ext4 for local storage
* nfs fo remotes

# Virtualization Models

PV vs HVM vs PVH
