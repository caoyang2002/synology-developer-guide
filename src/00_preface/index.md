# 前言

## 系统要求

### 工具包要求

1. 具有 root 权限的 64 位通用 Linux 环境（例如 Ubuntu 18.04 LTS）
2. bash (>= 4.1.5)
3. python (>= 2.7.3)

请不要在 Synology NAS 上安装工具包作为您的开发环境。NAS 专门用于存储，而不是用于通用开发目的。相反，您可以在 NAS 上安装 Docker 包，然后设置通用 linux 容器来安装工具包。

### 运行时要求

由于软件包是用于 DSM7 的，所以您应该拥有一台 DSM7 NAS。

## 准备环境

### 安装工具包

#### 工具包安装

您需要从[此链接](https://github.com/SynologyOpenSource/pkgscripts-ng/tree/DSM7.2)克隆前端脚本。从现在开始，我们将在本文档中使用 `/toolkit` 作为工具包基础。

```bash
apt-get install git
mkdir -p /toolkit
cd /toolkit
git clone https://github.com/SynologyOpenSource/pkgscripts-ng
```

然后，您需要安装一些工具来使构建的工具正常工作：

```bash
apt-get install cifs-utils \
    python \
    python-pip \
    python3 \
    python3-pip
```

此时，您可以找到如下工具包文件：

```bash
/toolkit
├── pkgscripts-ng/
│   ├── include/
│   ├── EnvDeploy    (deployment tool for chroot environment)
│   └── PkgCreate.py (build tool for package)
└── build_env/       (directory to store chroot environments)
```

### 为不同的 NAS 目标部署 Chroot 环境

为了加快开发速度，我们准备了几个不同架构的构建环境，其中包含一些预构建的项目，其可执行二进制文件或共享库是在 DSM 中构建的，例如 zlib、libxml2 等。

您可以使用 EnvDeploy 部署 NAS 的相应环境。例如，如果 avoton 架构中有一个 NAS，可以使用以下命令为 avoton 部署环境：

```bash
cd /toolkit/pkgscripts-ng/
git checkout DSM7.2
./EnvDeploy -v 7.2 -p avoton # for DSM7.2.2
```

可以手动下载环境压缩包。你必须把 `base_env-{version}.txz`， `ds.{platform}-{version}.dev.txz` 和 `ds.{platform}-{version}.env.txz` 到 `toolkit/toolkit_tarballs` 中。

```bash
/toolkit
├── pkgscripts-ng/
└── toolkit_tarballs/
    ├── base_env-7.2.txz
    ├── ds.avoton-7.2.dev.txz
    └── ds.avoton-7.2.env.txz
```



```bash
cd /toolkit/pkgscripts-ng/
./EnvDeploy -v 7.2 -p avoton -D # -D implies no download
```

如前所述，部署的环境包含一些预构建的库和标头，可以在 cross gcc sysroot 下找到。Sysroot 是编译器的默认搜索路径。如果 gcc 无法从给定路径中找到标头或库，它将搜索 sysroot/usr/{lib，include}。

```bash
/toolkit
├── pkgscripts-ng/
│   ├── include/
│   ├── EnvDeploy
│   └── PkgCreate.py
└── build_env/
    ├── ds.avoton-7.2/
    └── ds.avoton-6.2/
        └── usr/local/x86_64-pc-linux-gnu/x86_64-pc-linux-gnu/sys-root/
```

### 可用平台

您可以使用以下命令之一来显示可用平台。如果未给出 -v，则将列出所有版本的可用平台。

```bash
./EnvDeploy -v 7.2 --list
./EnvDeploy -v 7.2 --info platform
```

您可以使用属于同一平台系列的任何工具包为同一平台系列中的所有平台创建 spk。例如，您可以使用 Toolkit for Braswell 在所有x86_64兼容平台上创建包运行。对于平台系列，请查看平台和拱形值映射表。

### 更新环境

再次使用 EnvDeploy 更新环境。例如，您可以按如下方式更新 DSM 7.2.2 的 avoton 。

```bash
./EnvDeploy -v 7.2 -p avoton
```

### 移除环境

要删除环境，您首先需要卸载 /proc 文件夹，然后删除环境文件夹。以下命令说明了如何删除具有 7.2 版和平台 avoton 的环境。

```bash
umount /toolkit/build_env/ds.avoton-7.2/proc
rm -rf /toolkit/build_env/ds.avoton-7.2
```
