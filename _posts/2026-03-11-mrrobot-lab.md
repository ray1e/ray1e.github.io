---
layout: post
title: MrRobot Lab
author: 0xray1e
image: /assets/img/uploads/Mr__Robot - banner.png
date: 2026-03-11T17:19:00.000+03:00
description: This is a walkthrough of Mr Robot digital forensics lab in
  cyberdefenders, as well as my own notes for later reference.
categories:
  - cyberdefenders
  - notes
tags:
  - DigitalForensics
  - vmware
---
## Download the artefacts

You are provided with some dumps that will be useful for solving the challenges. Download and unzip them.

![](/assets/img/uploads/mr robot - downloaded files.png)

The files have `vmss` and `vmsd` extensions.

*VMSS File* - this file stores the memory content of the VM and is used to start the machine from a suspended state. It takes up roughly the same space as the assigned RAM.

*VMSD File* - This has information about the snapshots created. Every time a [snapshot](https://vmblog.com/bylines/understanding-vmware-virtual-machine-snapshots-what-you-need-to-know/) is created or deleted, this file is updated. It contains the name of the VMDK file and the VMSN file that each snapshot uses.

Other [VMware files](https://www.techtarget.com/searchvmware/tip/Understanding-the-files-that-make-up-a-VMware-virtual-machine).

`vmsd` is a text file.

```
encoding = "UTF-8"
snapshot.lastUID = "1"
snapshot.current = "1"
snapshot0.uid = "1"
snapshot0.filename = "POS-01-Snapshot1.vmsn"
snapshot0.displayName = "before-firstrun"
snapshot0.type = "1"
snapshot0.createTimeHigh = "336295"
snapshot0.createTimeLow = "1080846290"
snapshot0.numDisks = "1"
snapshot0.disk0.fileName = "POS-01.vmdk"
snapshot0.disk0.node = "scsi0:0"
snapshot.numSnapshots = "1"
```

Meaning of each entry in `vmsd`. Read more [here](https://knowledge.broadcom.com/external/article/337269/understanding-the-relationship-between-s.html).

```
snapshot.numSnapshots: The number of snapshots in the hierarchy
snapshot.lastUID: The most recently created snapshot UID = used to ensure there is no duplicate and is incremented by 1, every time a snapshot is created
snapshot.current: The current snapshot UID = the snapshot from which the virtual machine inherits.
snapshot.needConsolidate = TRUE: If a consolidation is needed
snapshotX: where X is the internal snapshot number.
snapshotX.uid: The unique Snapshot UID
snapshotX.displayName: The Snapshot name: displayed is Snapshot Manager
snapshotX.description: The optional description
snapshotX.filename: The VMSN file
snapshotX.numDisks: The number of virtual disks
snapshotX.diskY.node: The SCSI device ID
snapshotX.diskY.fileName: The virtual disk filename associated to the SCSI device ID snapshotX.diskY.node
```
