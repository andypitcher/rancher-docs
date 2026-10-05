---
title: About rancher-selinux
---

<head>
  <link rel="canonical" href="https://ranchermanager.docs.rancher.com/reference-guides/rancher-security/selinux-rpm/about-rancher-selinux"/>
</head>

To allow Rancher to work with SELinux, some functionality has to be manually enabled for the SELinux nodes. To help with that, Rancher provides an SELinux RPM.

:::tip Why SELinux?

By assigning a dedicated SELinux type to each container, we ensure that containers are limited to their minimal needs and cannot pivot to other resources if compromised.

:::

The `rancher-selinux` RPM contains a set of SELinux policies designed to grant the necessary privileges to various Rancher components running on Linux systems with SELinux enabled.

The source code, policy files and development guide are available in the [rancher-selinux GitHub repository](https://github.com/rancher/rancher-selinux).

## Supported Operating Systems

| Operating System                | Version | Repo path   |
| :------------------------------ | :------ | :---------- |
| openSUSE Leap / SLE / SLE Micro | 16 / 6  | `sle/16`    |
| openSUSE MicroOS / Tumbleweed   | Rolling | `microos`   |
| RHEL / CentOS / Rocky Linux     | 9       | `centos/9`  |
| RHEL / CentOS / Rocky Linux     | 10      | `centos/10` |

All policies are covered by end-to-end tests. For the most up-to-date matrix, see the [rancher-selinux support matrix](https://github.com/rancher/rancher-selinux#support-matrix).

## Installing the rancher-selinux RPM

Install `rancher-selinux` on **all SELinux-enabled nodes** in the cluster. Replace `<repo-path>` with the value from the table above.

### openSUSE Leap / SLE / SLE Micro / MicroOS

```bash
cat << EOF > /etc/zypp/repos.d/rancher.repo
[rancher]
name=Rancher
baseurl=https://rpm.rancher.io/rancher/production/<repo-path>/noarch
enabled=1
gpgcheck=1
gpgkey=https://rpm.rancher.io/public.key
EOF

zypper --gpg-auto-import-keys refresh
zypper install -y rancher-selinux
```

### RHEL / CentOS / Rocky Linux / Fedora

```bash
cat << EOF > /etc/yum.repos.d/rancher.repo
[rancher]
name=Rancher
baseurl=https://rpm.rancher.io/rancher/production/<repo-path>/noarch
enabled=1
gpgcheck=1
gpgkey=https://rpm.rancher.io/public.key
EOF

dnf install -y rancher-selinux
```

:::note Transactional systems

On SLE Micro and openSUSE MicroOS, install the package with `transactional-update pkg install rancher-selinux` and reboot the node.

:::

### Verifying the installation

```bash
semodule -l | grep rancher
```

## Configuring Applications to Work with SELinux

Applications do not automatically work once the `rancher-selinux` RPM is installed on the host. They need to be configured to run in an allowed SELinux container domain provided by the RPM.

To configure the supported charts to be SELinux aware, set `global.seLinux.enabled` to `true` in the `values.yaml` when installing the chart.

| Chart              | Container         | SELinux type                  |
| :----------------- | :---------------- | :---------------------------- |
| Rancher Monitoring | node-exporter     | `prom_node_exporter_t`        |
| Rancher Monitoring | pushprox          | `rke_kubereader_t`            |
| Rancher Logging    | fluentbit         | `rke_logreader_t`             |
| Rancher AI         | rancher-ai-agent  | `rancher_aiagent_container_t` |
| Rancher AI         | rancher-ai-mcp    | `rancher_aimcp_container_t`   |

:::note

Only the components listed above get a dedicated SELinux type. Other Rancher components and workloads use the default `container_t` type provided by `container-selinux`.

:::