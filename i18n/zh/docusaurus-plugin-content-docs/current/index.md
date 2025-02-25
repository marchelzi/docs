---
sidebar_position: 1
sidebar_label: Harvester 介绍
slug: /
title: "Harvester 介绍"
keywords:
  - Harvester
  - harvester
  - Rancher
  - rancher
  - Harvester 介绍
Description: Harvester 是基于 Kubernetes 构建的开源超融合基础架构 (HCI) 软件。它是 vSphere 和 Nutanix 的开源替代方案。
---

Harvester 是基于 Kubernetes 构建的开源[超融合基础架构](https://en.wikipedia.org/wiki/Hyper-converged_infrastructure) (HCI) 软件。它是使用专有 HCI 堆栈的一种开放替代方案，该堆栈结合了 [Cloud Native Computing](https://en.wikipedia.org/wiki/Cloud_native_computing) 的设计和精神。

![harvester-ui](/img/v1.1/dashboard.png)

## Harvester 功能

Harvester 支持在裸机服务器上实施 HCI。Harvester 使用本地、直接连接的存储，而不是复杂的外部 SANs。它提供了一个可集成启动的镜像进行交付，可以通过 ISO 或 PXE 引导直接部署到服务器上。

以下是 Harvester 当下提供的一些主要功能：

1. 虚拟机生命周期管理，包括 SSH-Key 注入、Cloud-init 配置、和图形、串行端口控制台
1. 虚拟机热迁移支持
1. 虚拟机备份、快照和还原
1. 分布式块存储和存储分层
1. 虚拟机中的多个网卡支持、可连接到管理或 VLAN网络
1. 虚拟机和 cloud-init 模板
1. 集成 [Rancher](https://github.com/rancher/rancher) 并提供多集群管理和 Harvester 主机驱动
1. [PXE/ipxe 引导支持](https://docs.harvesterhci.io/latest/install/pxe-boot-install)
1. 虚拟 IP (VIP) 和 Bond 网卡支持
1. 监控和日志集成

## Harvester 架构
下图为 Harvester 的概览架构：

![](/img/v1.1/architecture.svg)

- [Longhorn](https://longhorn.io/) 是一个轻量级、可靠、易用的 Kubernetes 分布式块存储系统。
- [KubeVirt](https://kubevirt.io/) 是一个 Kubernetes 的虚拟机管理插件。
- [Elemental for SLE-Micro 5.2](https://github.com/rancher-sandbox/cOS-toolkit)（v1.0.3 之前的版本基于 openSUSE Leap 15.3）是一个不可更改的 Linux 发行版，旨在尽量减少 Kubernetes 集群中节点的操作系统维护工作。

## 硬件要求

硬件需要满足以下要求，才可以启动和运行 Harvester：

| 类型 | 要求 |
|:---|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CPU | 仅支持 x86_64。需要硬件辅助虚拟化。8 核处理器（至少）用于测试，建议在生产环境中使用 16 核处理器。 |
| 内存 | 32 GB（至少），建议使用 64 GB 或以上的内存。 |
| 磁盘容量 | 200 GB（至少）用于测试，建议在生产中使用 500 GB 或以上的磁盘。 |
| 磁盘性能 | 每个磁盘 5,000+ 随机 IOPS (SSD/NVMe)。管理节点（前三个节点）必须[对 etcd 而言足够快](https://www.ibm.com/cloud/blog/using-fio-to-tell-whether-your-storage-is-fast-enough-for-etcd)。 |
| 网卡 | 1 Gbps 以太网（至少）用于测试，建议在生产中使用 10 Gbps 或以上的以太网。 |
| 网络交换机 | VLAN 支持所需的端口中继。 |

## 快速开始

你可以通过 [ISO](./install/iso-install.md) 或 [PXE](./install/pxe-boot-install.md) 来安装 Harvester。详情请参见以下说明。

### ISO 安装

你可以使用 ISO，将 Harvester 直接安装在裸金属服务器上，来组成一个 Harvester 集群。用户可以添加一个或多个计算节点到现有的集群。

如果需要获取 Harvester ISO，访问 [GitHub 上的发行版本](https://github.com/harvester/harvester/releases)进行下载。

在安装过程中，你可以选择组建一个新的集群，或者将节点加入到现有的集群中。

1. 安装 Harvester ISO 磁盘并通过选择 `Harvester Installer` 来启动服务器。
   ![iso-install.png](/img/v1.1/install/iso-install.png)
2. 通过创建新的 Harvester 集群或加入现有集群来选择安装模式。
3. 选择要安装 Harvester 集群的设备。
   - 注意：默认情况下，Harvester 对 UEFI 和 BIOS 使用 [GPT](https://en.wikipedia.org/wiki/GUID_Partition_Table) 分区表。如果你使用 BIOS 引导，则可以选择 [MBR](https://en.wikipedia.org/wiki/Master_boot_record)。
      ![iso-install-disk.png](/img/v1.1/install/iso-install-disk.png)
5. 建议选择单独的磁盘来存储 VM 数据。
   ![iso-install-disk.png](/img/v1.1/install/iso-select-data-disk.png)
6. 配置主机名并选择管理网络的网络接口。默认情况下，Harvester 将创建一个名为 `mgmt-bo` 的 Bond NIC，IP 地址可以通过 DHCP 进行配置或静态分配。<small>（注意：节点 IP 在Harvester 集群的生命周期中不可更改。如果使用了 DHCP，用户必须确保 DHCP 服务器始终为同一个节点提供相同的 IP。如果节点 IP 发生变化，相关节点无法加入集群，甚至可能破坏集群）</small>。

   ![iso-installed.png](/img/v1.1/install/iso-nic-config.png)
7. （可选）配置 DNS 服务器；使用逗号作为分隔符。
8. 配置用于访问集群或加入集群中其他节点的 `Virtual IP`<small>（注意：如果你的 IP 地址是通过 DHCP 配置的，则需要在 DHCP 服务器上配置静态 MAC 到 IP 地址的映射，以便拥有持久的 Virtual IP，VIP 必须与所有节点 IP 都不一样）。</small>
9. 配置 `cluster token`。这个 Token 会用于将其他节点添加到集群中。
10. 为主机配置登录密码。默认的 SSH 用户是 `rancher`。
11. 建议配置 NTP 服务器以确保所有节点的时间同步。默认值是 `0.suse.pool.ntp.org`。
12. （可选）如果你需要使用 HTTP 代理来访问外部环境，在此处输入代理的 URL。否则，请留空。
13. （可选）你可以从远端服务器 URL 导入 SSH 密钥。你的 GitHub 公钥可以与 `https://github.com/<username>.keys` 一起使用。
14. （可选）如果你需要使用 [Harvester 配置文件](./install/harvester-configuration.md)来自定义主机，在此处输入 HTTP URL。
15. 确认安装选项后，Harvester 会安装到你的主机上。安装过程可能需要几分钟。
16. 主机会在安装完成后重启，然后会显示一个带有管理 URL 和状态的控制台 UI<small>（你可以使用 F12 在 Harvester 控制台和 Shell 之间切换）</small>。
17. 网页界面的默认 URL 是 `https://your-virtual-ip`。
   ![iso-installed.png](/img/v1.1/install/iso-installed.png)
18. 在首次登录时，你会收到为默认 `admin` 用户设置密码的提示。
   ![first-login.png](/img/v1.1/install/first-time-login.png)

<div class="text-center">
<iframe width="950" height="475" src="https://www.youtube.com/embed/Ngsk7m6NYf4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

### PXE/iPXE 安装

Harvester 可以自动安装。如果需要获取详细的说明和指导，请参见 [PXE 引导安装](./install/pxe-boot-install.md)。

:::note

你可访问 [harvester/ipxe-examples](https://github.com/harvester/ipxe-examples) 获取更多 iPXE 用法示例。

:::
