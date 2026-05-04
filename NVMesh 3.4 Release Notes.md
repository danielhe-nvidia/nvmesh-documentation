# NVMesh 3.4 Release Notes <img src="./rn-media/NVIDIA_logo.png" width="220" alt="The NVIDIA logo." />

<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# Change log

| Version |          Date           | Release | Soul                           |
| :-----: | :---------------------: | :-----: | ------------------------------ |
|   1.0   | <nobr>2026-03-12</nobr> |  3.4.0  | First release of NVMesh 3.4.0. |
|   1.1   | <nobr>2026-04-29</nobr> | 3.4.0-HF1 | Release of NVMesh 3.4.0 HF1. |

# General

The main objective of the NVMesh 3.4.0 release is to provide some key resilience-related improvements. The release of the first functionally complete version of managed non-disruptive upgrade (mNDU) is the main enhancement.

See [Release Index](https://nvidia.atlassian.net/wiki/spaces/NSV/pages/2831793511/Release+index) for package details.

# Functionality

## Managed NDU Completion and Optimizations

[NVMESH-2557](https://jirasw.nvidia.com/browse/NVMESH-2557) \- The scope of mNDU now includes upgrading management itself, the nvmesh-upgrader agents and the interopDB. The end-to-end upgrade is initiated by upgrading and restarting a single management. Then this upgraded management can be instructed to upgrade the rest of the cluster.

- For previous upgrade agents, i.e., prior to NVMesh 3.4.0, these will not be auto-upgraded, so the new version of the upgrader will have to be installed and restarted manually on all nodes running NVMesh.

[NVMESH-6594](https://jirasw.nvidia.com/browse/NVMESH-6594) \- Multiple clients can now be upgraded concurrently, i.e., in parallel instead of one by one. In addition, mNDU does not stop on a single upgrade failure. Instead, it stops after some user-set number of failures.

## TOMA Field Resilience Improvements

A few options have been added to the `toma_rpc` application to facilitate overcoming field issues. These options were added as a means to overcome reversion of specific TOMA issues that have been fixed in the interim. Nevertheless, these options may be useful in certain unexpected scenarios instead of restarting TOMAs and thus provide SREs with additional optionality.

[NVMESH-7068](https://jirasw.nvidia.com/browse/NVMESH-7068) \- Addition of a `toma_rpc` command to instruct the local TOMA to stop being the leader. This is done as follows:

- `toma_rpc simulate reelect`

[NVMESH-7072](https://jirasw.nvidia.com/browse/NVMESH-7072) \- Addition of a `toma_rpc` command to instruct the TOMAs to resend all volume statuses to management to resynchronize them. The command can also be limited to a specific volume. This is done as follows:

- `toma_rpc simulate resend-praids-report all/<volume_name>`

## NVMesh Observability Enhancements

Multiple enhancements have been made to the NVMesh exporter as described in the EPIC, [NVMESH-5849](https://jirasw.nvidia.com/browse/NVMESH-5849) providing enhanced observability for memory usage, VPG consumption, SERJIO usage and TOMA RAFT information.

Some `/proc` additions and changes have been implemented as part of these enhancements.

## REST API for Metadata Management

Prior to NVMesh 3.4.0, it was possible to insert arbitrary fields in volume definitions through REST CRUD operations. This was not limited, so it was possible to collide with fields used by management itself. To make this more robust, only management fields are allowed in the base volume object hereon. User-defined fields can only be set within the volume's metadata section, [NVMESH-5320](https://jirasw.nvidia.com/browse/NVMESH-5320).

Volume metadata can also be managed via the CLI, [NVMESH-7010](https://jirasw.nvidia.com/browse/NVMESH-7010).

## Integrated OTEL Support

Integrated management support for OpenTelemetry (OTEL) traces has been added. It runs auto-instrumentation for several adjunct components, i.e., MongoDB, Kafka and NodeJS, which generate a significant amount of traces. NVMesh management itself generates a small amount of traces reporting on the the internal management queue length.

Integrated OTEL support is disabled by default and can be enabled via `management.js.conf` via the options in `config.openTelemetry`.

## CPU Pinning and "Noisy Neighbor" Reduction

NVMesh 3.4.0 introduces new options for pinning IO of specific volumes to specific CPU cores. This is useful for machines running multiple applications requiring different volumes that are CPU-core separated, e.g., when a volume is used only by a specific container or VM and it is pinned to specific CPU cores. In that case, it makes sense to pin the IO to that volume the same cores, [NVMESH-6156](https://jirasw.nvidia.com/browse/NVMESH-6156).

**<u>Note:</u>** This is alpha level functionality and has not undergone any QA.

## Performance Improvements for Ethernet Multi-Rail Environments

In Ethernet multi-rail environments, many connection attempts will fail. Improvements were made to reduce the affect this has during error situations so that reconnection and IO resumption is significantly faster, [NVMESH-7778](https://jirasw.nvidia.com/browse/NVMESH-7778).

## Performance Improvement to Drive Formatting

Drive formatting time on multi-drive servers is improved by performing multiple formats in parallel, [NVMESH-7337](https://jirasw.nvidia.com/browse/NVMESH-7337).

## Monitoring Improvements

[NVMESH-6578](https://jirasw.nvidia.com/browse/NVMESH-6578) \- Add /proc file for SERJIO status in JSON format.

[NVMESH-7067](https://jirasw.nvidia.com/browse/NVMESH-7067) \- Add /proc file for active TOMA mTLS certificate inspection.

[NVMESH-7696](https://jirasw.nvidia.com/browse/NVMESH-7696) \- Add /proc file for TOMA client subscribers count.

## NVMesh CSI Driver Update, version 1.9.1

This CSI Driver supports both this version of NVMesh and the following earlier versions: NVMesh 2.7.2-HF16+, NVMesh 3.3.1-HF7+ and NVMesh 3.3.2-HF3+.

### New Features

None.

### Bug Fixes

[NVMESH-6750](https://jirasw.nvidia.com/browse/NVMESH-6750) \- Reorganize shutdown process with configurable pre-shutdown delay.

[NVMESH-6872](https://jirasw.nvidia.com/browse/NVMESH-6872) \- Update system packages and urllib3 for security fixes.

[NVMESH-7590](https://jirasw.nvidia.com/browse/NVMESH-7590) \- CSI v1.9.0 incorrectly addresses clients by their K8s node name.

[NVMESH-7605](https://jirasw.nvidia.com/browse/NVMESH-7605) \- Parsing of the client's status.json output failed due to a change in a field's format leading to incorrect assertion of state.

Unticketed \- Fix a typo in a config field, `allowIncompatibleVersions`.

### Compatibility

- NVMesh 2.7.2, 3.3.1, 3.3.2, 3.4.0
- Kubernetes 1.22 \- 1.34

# CLI Changes

## Command Changes

- **–description** added to **user** **create**/**update** and to **config-profile create**/**update** commands

## Alpha Features

The following features are considered Alpha and not intended for use by end-users:

- An alpha feature has been added enabling mass or bulk operations on objects using shell-style ranges such as “**volume create –name test-volumes-{prod,stage}-{0..100} …**”. The use case for this is primarily testing related. This should not be used for production operations.

- **upgrade-agent, upgrade, upgrade-step, component, release, platform** \- these are new commands related to NDU. These are partial implementations that are not qualified for production operations.

# Resolved Issues

<!--
Template for new table entries
| [NVMESH-](https://jirasw.nvidia.com/browse/NVMESH-) | | |
-->

| Ticket | Description | Comments |
| :-: | :-- | :-- |
| <nobr>[NVMESH‑5138](https://jirasw.nvidia.com/browse/NVMESH-5138)</nobr> | Bug fix for Grace CPU when the IOMMU is enabled. |  |
| <nobr>[NVMESH‑5412](https://jirasw.nvidia.com/browse/NVMESH-5412)</nobr> | Improve performance for local drive operations. |  |
| <nobr>[NVMESH‑5956](https://jirasw.nvidia.com/browse/NVMESH-5956)</nobr> | Improve cold recovery to handle additional error cases of media errors. |  |
| <nobr>[NVMESH‑6061](https://jirasw.nvidia.com/browse/NVMESH-6061)</nobr> | Correct nvmesh-utils installation issue. |  |
| <nobr>[NVMESH‑6336](https://jirasw.nvidia.com/browse/NVMESH-6336)</nobr> | Fixed a bug that caused failures in drive formatting. | Affected Micron 6500 drives. |
| <nobr>[NVMESH‑6337](https://jirasw.nvidia.com/browse/NVMESH-6337)</nobr> | Reload systemd daemon as part of RPM post-install. |  |
| <nobr>[NVMESH‑6338](https://jirasw.nvidia.com/browse/NVMESH-6338)</nobr> | Error handling improvements to client service startup. |  |
| <nobr>[NVMESH‑6395](https://jirasw.nvidia.com/browse/NVMESH-6395)</nobr> | Old traces deleted on upgrade. |  |
| <nobr>[NVMESH‑6554](https://jirasw.nvidia.com/browse/NVMESH-6554)</nobr> | TOMA networking did not handle an EWOULDBLOCK return from a call to sendto. |  |
| <nobr>[NVMESH‑6574](https://jirasw.nvidia.com/browse/NVMESH-6574)</nobr> | Correct nvmesh_update kernel parsing. |  |
| <nobr>[NVMESH‑6712](https://jirasw.nvidia.com/browse/NVMESH-6712)</nobr> <br> <nobr>[NVMESH‑6802](https://jirasw.nvidia.com/browse/NVMESH-6802)</nobr> <br> <nobr>[NVMESH‑7253](https://jirasw.nvidia.com/browse/NVMESH-7253)</nobr> | Improve handling of detaching of deleted volumes during restarts. |  |
| <nobr>[NVMESH‑6726](https://jirasw.nvidia.com/browse/NVMESH-6726)</nobr> | Fix incorrect iostats latency units, off by 10x. |  |
| <nobr>[NVMESH‑6786](https://jirasw.nvidia.com/browse/NVMESH-6786)</nobr> | Fix SoftiWarp race condition that causes a kernel crash. |  |
| <nobr>[NVMESH‑6788](https://jirasw.nvidia.com/browse/NVMESH-6788)</nobr> | Fix a client crash when the IOMMU is enabled. |  |
| <nobr>[NVMESH‑6837](https://jirasw.nvidia.com/browse/NVMESH-6837)</nobr> | Improve connectivity times upon IP address change. |  |
| <nobr>[NVMESH‑7022](https://jirasw.nvidia.com/browse/NVMESH-7022)</nobr> | Avoid soft lockups and reduce the time to IO enabled when the IOMMU is enabled. |  |
| <nobr>[NVMESH‑7038](https://jirasw.nvidia.com/browse/NVMESH-7038)</nobr> | Fix regression in volume encryption robustness under load. |  |
| <nobr>[NVMESH‑7054](https://jirasw.nvidia.com/browse/NVMESH-7054)</nobr> | Prevent kernel crash in SoftiWarp upon a multi-disaster scenario. |  |
| <nobr>[NVMESH‑7071](https://jirasw.nvidia.com/browse/NVMESH-7071)</nobr> | Improve recovery from Kafka certificate expiry and other Kafka issues to avoid Kafka message loss. |  |
| <nobr>[NVMESH‑7198](https://jirasw.nvidia.com/browse/NVMESH-7198)</nobr> | Avoid having to do a TOMA leader restart to start some rebuilds after double drive eviction. |  |
| <nobr>[NVMESH‑7226](https://jirasw.nvidia.com/browse/NVMESH-7226)</nobr> | Avoid TOMA crash on missing crt file. |  |
| <nobr>[NVMESH‑7288](https://jirasw.nvidia.com/browse/NVMESH-7288)</nobr> | Revert changes made that increased mNDU IO-disabled time. |  |
| <nobr>[NVMESH‑7313](https://jirasw.nvidia.com/browse/NVMESH-7313)</nobr> | Reduce redundant SIW trace message, "Nothing to receive". |  |
| <nobr>[NVMESH‑7673](https://jirasw.nvidia.com/browse/NVMESH-7673)</nobr> | Fix rare crash in SIW kernel module. |  |
| <nobr>[NVMESH‑7772](https://jirasw.nvidia.com/browse/NVMESH-7772)</nobr> | Fix crash due to race condition in the target. | The bug may have been introduced in the development of 3.4.0, so may be redundant to note it. |
| <nobr>[NVMESH‑7778](https://jirasw.nvidia.com/browse/NVMESH-7778)</nobr> | Improving handling of TCP_CLOSE in the SoftiWarp stack. | This improves error handling performance and IO disabled times when using SoftiWarp. |
| <nobr>[NVMESH‑7797](https://jirasw.nvidia.com/browse/NVMESH-7797)</nobr> | Improve TOMA network path selection for RAFT messages to increase robustness. |  |

# Known Issues

| Ticket | Description | Workaround & Comments |
| --- | :-- | :-- |
| <nobr>[NVMESH‑7214](https://jirasw.nvidia.com/browse/NVMESH-7214)</nobr> | When performing mNDU from 3.3.2 to 3.4.0, the 3.3.2 management may still be accessible for a few minutes. | This could be considered as working as designed in practice. However, set the "Disable Old Management when in Upgrade Mode" to true before upgrading to avoid this. |
| <nobr>[NVMESH‑7269](https://jirasw.nvidia.com/browse/NVMESH-7269)</nobr> | The manual upgrade option appears as an option in the CLI, while in practice it will be rejected as an incorrect option by management. | The manual mode is not a product feature, rather used for debug. |
| <nobr>[NVMESH‑7547](https://jirasw.nvidia.com/browse/NVMESH-7547)</nobr> | After upgrading to a hotfix release (e.g., 3.4.0-HF1), the Dashboard may incorrectly display a "New version available" popup for the already-installed hotfix version, and the Create Upgrade page may list both the base release and the hotfix as destination versions. | Cosmetic issue only; does not affect upgrade functionality. Resolved in 3.5.0. |
| <nobr>[NVMESH‑7676](https://jirasw.nvidia.com/browse/NVMESH-7676)</nobr> | When performing mNDU from 3.3.2 to 3.4.0, a management with 3.3.2 that is coming up and running sanity checks may encounter a KAFKA error. | The management performing the mNDU will still continue the upgrade including the failed management. |
| <nobr>[NVMESH‑7755](https://jirasw.nvidia.com/browse/NVMESH-7755)</nobr> | When mNDU is performed on a client that is encrypting a volume, that volume may remain attached in limbo on the client, in the atom state indefinitely. | Reboot the node to clean the state. |
| <nobr>[NVMESH‑7826](https://jirasw.nvidia.com/browse/NVMESH-7826)</nobr> | On nodes with IOMMU enabled, with some kernels and with some drives, unbinding and then binding a drive to an NVMe driver, either the built-in kernel one or NVMesh's driver, may cause corrupt memory writes. | This behavior is not related to NVMesh directly. <br><br> Kernels such as the following exhibit this behavior: <ul style="padding-left: 24px;"><li>`5.15.0-164-generic`</li> <li>`5.19.0-50-generic`</li> <li>`6.2.0-25-generic`</li></ul> <br><br> Newer kernels such as the following do not exhibit this behavior: <ul style="padding-left: 24px;"><li>`6.5.0-25-generic`</li><li>`6.8.0-100-generic`</li></ul> <br><br> Using strict IOMMU, i.e., setting the kernel command line parameter `iommu.strict=1`, prevents this, but affects performance significantly and so is not recommended. |

# Documentation

[Documentation Repository](https://gitlab-master.nvidia.com/excelero/nvmesh-documentation)

## Support Matrix Update

The updated NVMesh support matrix is available at [NVMesh Support Matrix](https://confluence.nvidia.com/display/NSV/NVMesh+Support+Matrix).

**<u>Note:</u>** Kernels from 6.8.0 up until 6.14.6 suffer from a kernel workqueue crash in ​​cma_netevent_work_handler, as described [here](https://bugzilla.redhat.com/show_bug.cgi?id=2363273). NVMesh is incompatible with these kernels ([NVMESH-6447](https://jirasw.nvidia.com/browse/NVMESH-6447)) as is. The NVMesh team has inserted a patched version of the relevant non-NVMesh kernel modules to fix this issue and make NVMesh compatible.

# Pre-built Packages

All NVMesh 3.4.0 packages can be found in NVIDIA's artifactory [here](https://urm.nvidia.com/artifactory/sw-ngc-nvmesh-generic-local/3.4.0).

# Upgrade

Upgrading from this version to future versions will be best conducted using the mNDU feature, see above for more details.

Upgrading from versions prior to NVMesh 3.2.0-HF2 is not possible. Upgrading from 3.2.0-HF2 is with a cold upgrade. From NVMesh 3.3.0 and onwards, it is recommended to perform upgrades using mNDU. For these versions, hot upgrade is supported.

**<u>Note:</u>** [NVMESH-7874](https://jirasw.nvidia.com/browse/NVMESH-7874) - For Ubuntu 22.04 and above, it is required to disable automatic service restarts during package upgrades. Ubuntu uses a utility called `needrestart` that may trigger an automatic restart of NVMesh services during an `apt install` or `apt upgrade`. To prevent this for the NVMesh services, create the a configuration file named `/etc/needrestart/conf.d/needrestart_nvmesh.conf` with the following contents:

```
$nrconf{override_rc} ||= {};
$nrconf{override_rc}->{qr(^nvmesh.\*\.service$)} = 0;
```

# NVMesh 3.4.0-HF1

## Functionality Changes

1. [NVMESH-7602](https://jirasw.nvidia.com/browse/NVMESH-7602) - Added exporter visibility for disk timeout counters to improve disk timeout monitoring.
2. [NVMESH-8141](https://jirasw.nvidia.com/browse/NVMESH-8141), [NVMESH-8446](https://jirasw.nvidia.com/browse/NVMESH-8446) - Added generic Linux kernel 6.17 support, including Linux kernel 6.17.0-1007-oracle.
3. [NVMESH-8164](https://jirasw.nvidia.com/browse/NVMESH-8164) - Added support for MongoDB 8.0 and updated the `mongosh` dependency constraint accordingly.
4. [NVMESH-8227](https://jirasw.nvidia.com/browse/NVMESH-8227) - Added support for Rocky Linux 10.1 with kernel 6.12.0 and DOCA-OFED 3.2.2 for the kernel modules. Management testing is still underway.

## Fixed Issues

1. [NVMESH-5149](https://jirasw.nvidia.com/browse/NVMESH-5149) - Fixed TOMA handling when the system clock moves backward.
2. [NVMESH-7198](https://jirasw.nvidia.com/browse/NVMESH-7198) - Fixed a TOMA issue where rebuilds could require a TOMA leader restart to begin after drive re-eviction.
3. [NVMESH-7422](https://jirasw.nvidia.com/browse/NVMESH-7422) - Fixed a TOMA shutdown delay caused by long-running snapshot activity.
4. [NVMESH-7595](https://jirasw.nvidia.com/browse/NVMESH-7595) - Improved TOMA handling of Kafka connection failures that could prevent it from receiving management configuration.
5. [NVMESH-7600](https://jirasw.nvidia.com/browse/NVMESH-7600) - Removed the obsolete `RPM/docker` directory from the management package sources.
6. [NVMESH-7682](https://jirasw.nvidia.com/browse/NVMESH-7682) - Fixed RPM packaging so `nvmeshmgr` is enabled only on first installation and is not re-enabled during upgrades when it was disabled.
7. [NVMESH-7968](https://jirasw.nvidia.com/browse/NVMESH-7968) - Fixed SERJIO statistics reporting so successful `alloc_rng` operations are not incorrectly counted as errors.
8. [NVMESH-7970](https://jirasw.nvidia.com/browse/NVMESH-7970) - Fixed a TOMA issue that could leave newly created volumes degraded or not fully online.
9. [NVMESH-8050](https://jirasw.nvidia.com/browse/NVMESH-8050) - Preserved TLS certificate file permissions during certificate rotation so monitoring tools can continue reading certificate metadata.
10. [NVMESH-8111](https://jirasw.nvidia.com/browse/NVMESH-8111) - Fixed a rare exporter issue where SERJIO metrics could omit a drive.
11. [NVMESH-8298](https://jirasw.nvidia.com/browse/NVMESH-8298) - Added a TOMA memory limit to prevent excessive memory consumption.
12. [NVMESH-8310](https://jirasw.nvidia.com/browse/NVMESH-8310) - Fixed cleanup of Kafka topics used for TOMA commands so cleared TOMA commands are garbage collected.
13. [NVMESH-8353](https://jirasw.nvidia.com/browse/NVMESH-8353) - Fixed TOMA certificate reload handling so a TOMA that was down during certificate expiration can recover after renewal.
14. [NVMESH-8508](https://jirasw.nvidia.com/browse/NVMESH-8508) - Fixed client upgrade from NVMesh 3.3.2 to 3.4.0-HF1 so the client module loads correctly after restart.
15. [NVMESH-8527](https://jirasw.nvidia.com/browse/NVMESH-8527) - Fixed core compilation with DOCA-OFED 25.10.

## Known Issues

There are no known bugs in NVMesh 3.4.0-HF1.

## Deferred Issues

1. [NVMESH-8535](https://jirasw.nvidia.com/browse/NVMESH-8535) - NDU is not supported on 6.x kernels.
2. [NVMESH-8539](https://jirasw.nvidia.com/browse/NVMESH-8539) - An issue was found with slow IO response. This is still under investigation, as the IO load may have been excessive. The decision was made not to hold this version back at this point.
3. [NVMESH-8583](https://jirasw.nvidia.com/browse/NVMESH-8583) - A TOMA may not connect to the system initially. To recover, restart the TOMA. If that is insufficient, delete the target and restart `nvmeshtarget`.

# Rocky Linux 10.1 Environment Setup

## Overview

This section describes how to configure a **Rocky Linux 10.1** system with:

- **Python 3.12** — system default, application runtime
- **Python 3.10** — side-by-side, required for build/compile-time dependencies
- **Node.js 18** — replacing the native newer version
- **Java** — OpenJDK with version fallback
- **MongoDB 7.0**
- **Kafka client** — confluent-kafka 2.8.0 with bundled librdkafka 2.8.0, running under Python 3.12

**All commands must be run as root or with sudo.**

## 1. Base System Preparation

    yum update -y
    yum install -y wget curl tar gzip ca-certificates epel-release dnf-plugins-core
    dnf config-manager --set-enabled crb

## 2. Install Python 3.10 (Build/Compile Dependency)

    yum groupinstall -y "Development Tools"
    yum install -y gcc make openssl-devel bzip2-devel libffi-devel zlib-devel \
                   xz-devel sqlite-devel readline-devel tk-devel ncurses-devel

    cd /tmp
    wget https://www.python.org/ftp/python/3.10.14/Python-3.10.14.tgz
    tar xzf Python-3.10.14.tgz
    cd Python-3.10.14
    ./configure --enable-optimizations --prefix=/usr/local
    make -j"$(nproc)"
    make altinstall

**Verify:** python3.10 --version → Python 3.10.14

## 3. Install Node.js 18

### Remove the native Node.js

    yum remove -y nodejs npm || true
    yum module reset -y nodejs || true
    yum module disable -y nodejs || true

### Install Node.js 18 from NodeSource

    curl -fsSL https://rpm.nodesource.com/setup_18.x | bash -
    yum install -y nodejs-18.* --allowerasing

**Verify:** node --version → v18.x.x

## 4. Install Java (with version fallback)

    yum install -y java-21-openjdk-headless \
      || yum install -y java-21-openjdk \
      || yum install -y java-17-openjdk-headless \
      || yum install -y java-1.8.0-openjdk

    echo 'export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))' | tee /etc/profile.d/java.sh
    echo 'export PATH=$JAVA_HOME/bin:$PATH' | tee -a /etc/profile.d/java.sh
    source /etc/profile.d/java.sh

**Verify:** java -version

## 5. Install MongoDB 7.0

> Uses RHEL 9 repo (no RHEL 10 repo available yet). Requires AVX CPU support.

### Add the MongoDB 7.0 repo

    cat > /etc/yum.repos.d/mongodb-org-7.0.repo <<REPO
    [mongodb-org-7.0]
    name=MongoDB Repository
    baseurl=https://repo.mongodb.org/yum/redhat/9/mongodb-org/7.0/x86_64/
    gpgcheck=0
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
    REPO

### Install

    yum install -y mongodb-org

### (Optional) Pin MongoDB 7.0

    yum install -y python3-dnf-plugin-versionlock
    dnf versionlock add mongodb-org mongodb-org-database mongodb-org-server \
                        mongodb-org-mongos mongodb-org-tools mongodb-mongosh

### Enable and start

    systemctl daemon-reload
    systemctl enable --now mongod

**Verify:** mongod --version → db version v7.0.x

## 6. Install Kafka Client (confluent-kafka)

The confluent-kafka wheel from PyPI bundles its own **librdkafka 2.8.0**.
The system package librdkafka-2.1.1-7.el10 may be present but is **not loaded** by Python.

### Install

    python3.12 -m pip install confluent-kafka

### Verify

    python3.12 -c "import confluent_kafka; print('confluent-kafka:', confluent_kafka.__version__); print('librdkafka   :', confluent_kafka.libversion()[0])"

Expected:

    confluent-kafka: 2.8.0
    librdkafka   : 2.8.0

## 7. Verification Checklist

| Component           | Command                                              | Expected         |
|---------------------|------------------------------------------------------|------------------|
| OS                  | cat /etc/rocky-release                               | Rocky Linux 10.1 |
| Python (runtime)    | python3.12 --version                                 | Python 3.12.x    |
| Python (build-only) | python3.10 --version                                 | Python 3.10.14   |
| Node.js             | node --version                                       | v18.x.x          |
| Java                | java -version                                        | 21 / 17 / 1.8    |
| MongoDB             | mongod --version                                     | v7.0.x           |
| confluent-kafka     | python3.12 -c "import confluent_kafka; ..."          | 2.8.0            |
| librdkafka runtime  | python3.12 -c "... confluent_kafka.libversion() ..." | 2.8.0            |

## 8. Known Notes

1. **Python 3.12** is the application runtime.
2. **Python 3.10** is build-only (make altinstall, does not overwrite system Python).
3. **Node.js** native newer version must be removed and module disabled before installing Node 18.
4. **MongoDB 7.0** — RHEL 9 repo, requires AVX CPU, EOL August 2026.
5. **Kafka client** — bundled librdkafka 2.8.0 inside wheel; system RPM 2.1.1 unused by Python.
6. **Java** — JAVA_HOME only needed if running a Kafka broker locally.
