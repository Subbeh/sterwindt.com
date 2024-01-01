+++
title = 'Homelab'
date = 2023-12-31T15:55:10+11:00
draft = true
+++

# Introduction

After starting off with a simple Raspberry Pi to run Pihole, I got sucked into the world of homelabs and went down the rabbit hole of spending way too much time and effort creating my own ecosystem of locally hosted services.

Two years down the road, this is the current state of my setup.

[repository](https://github.com/Subbeh/homelab)

# Hardware

| Device                   | OS Disk Size | Data Disk Size | Cores | Ram  | Operating System | Purpose           |
| ------------------------ | ------------ | -------------- | ----- | ---- | ---------------- | ----------------- |
| NUC 12 Pro (i7 1260P)    | 1TB SSD      | 1x 500GB SSD   | 16    | 64GB | Proxmox          | VMs               |
| Dell OptiPlex (i5-8500T) | 1TB SSD      | 1x 500GB SSD   | 6     | 32GB | Proxmox          | VMs               |
| Dell OptiPlex (i5-6500T) | 256GB SSD    | 1x 500GB SSD   | 4     | 16GB | Proxmox          | VMs               |
| Topton n5105 NAS         | 1TB SSD      | 2x 6TB HDD     | 4     | 32GB | Proxmox          | NAS / VMs         |
| Topton n5105             | 128GB SSD    | -              | 4     | 16GB | OPNsense         | Firewall / Router |
| RPi 4                    | 32GB         | -              | 4     | 4GB  | PiKVM            | Network KVM       |
| Unifi Swtich Lite 16 PoE | -            | -              | -     | -    | -                | Network Switch    |

## Infrastructure

![Infrastructure](/images/infra.png)

# Components

Most of the infrastructure and services are deployed and managed by utilizing infrastructure as Code and GitOps practices.

## Core Components

<table>
  <tr>
    <th></th>
    <th>Name</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><img width="32" src="https://github.com/simple-icons/simple-icons/raw/master/icons/opnsense.svg"></td>
    <td><a href="https://opnsense.org">OPNsense</a></td>
    <td>Open source firewall and routing software</td>
  </tr>
  <tr>
    <td><img width="32" src="https://github.com/simple-icons/simple-icons/raw/master/icons/proxmox.svg"></td>
    <td><a href="https://www.proxmox.com">Proxmox</a></td>
    <td>Hyper-converged infrastructure open-source software</td>
  </tr>
  <tr>
    <td><img width="32" src="https://www.vectorlogo.zone/logos/debian/debian-icon.svg"></td>
    <td><a href="https://www.debian.com">Debian</a></td>
    <td>Linux distribution</td>
  </tr>
  <tr>
    <td><img width="32" src="https://vectorlogo.zone/logos/ansible/ansible-icon.svg"></td>
    <td><a href="https://ansible.com">Ansible</a></td>
    <td>Bare metal provisioning and configuration</td>
  </tr>
  <tr>
    <td><img width="32" src="https://vectorlogo.zone/logos/terraformio/terraformio-icon.svg"></td>
    <td><a href="https://terraform.io">Terraform</a></td>
    <td>Provision resources on external environments</td>
  </tr>
  <tr>
    <td><img width="32" src="https://www.vectorlogo.zone/logos/giteaio/giteaio-icon.svg"></td>
    <td><a href="https://www.gitea.com">Gitea</a></td>
    <td>Open-source Git hosting and artifact platform</td>
  </tr>
  <tr>
    <td><img width="32" src="https://vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg"></td>
    <td><a href="https://kubernetes.io">Kubernetes</a></td>
    <td>Orchestration system for managing containers</td>
  </tr>
  <tr>
    <td><img width="32" src="https://vectorlogo.zone/logos/traefikio/traefikio-icon.svg"></td>
    <td><a href="https://traefik.io">Traefik</a></td>
    <td>Cloud native ingress controller for Kubernetes</td>
  </tr>
  <tr>
    <td><img width="32" src="https://www.vectorlogo.zone/logos/argoprojio/argoprojio-icon.svg"></td>
    <td><a href="https://argo-cd.readthedocs.io/">Argo CD</a></td>
    <td>Declarative GitOps Continuous Delivery for Kubernetes</td>
  </tr>
  <tr>
    <td><img width="32" src="https://www.vectorlogo.zone/logos/droneio/droneio-icon.svg"></td>
    <td><a href="https://www.drone.io">Drone CI</a></td>
    <td>Self-service Continuous Integration platform</td>
  </tr>
  <tr>
    <td><img width="32" src="https://github.com/vscode-icons/vscode-icons/raw/master/icons/file_type_doppler.svg"></td>
    <td><a href="https://www.doppler.com">Doppler</a></td>
    <td>Secrets Management platform</td>
  </tr>
</table>

# Automation & CI/CD

![CICD](/images/cicd.png)

## Repositories

- [homelab](https://github.com/Subbeh/homelab)
- [ansible-runner](https://github.com/Subbeh/ansible-runner)
- [renovate](https://github.com/Subbeh/renovate)

## Dependency Updates

Automated dependency updates are performed by [Renovate](https://github.com/renovatebot/renovate). There are several ways to run Renovate, but I chose to have it run locally as part of my CI pipelines. Drone CI kicks off a scheduled job to run Renovate daily, which in turn scans my repositories for updated versions in packages and images, and creates a pull request in the target repositories with the updated versions. I then manually review the pull requests and merge the ones I want to have updated.

Once new versions in the manifests are committed inside the homelab repository, Drone CI and Argo CD are triggered to update my self-hosted services and Kubernetes deployments respectively.

[Self-hosted services](https://github.com/Subbeh/homelab/tree/main/ansible/apps) outside of my Kubernetes cluster are [updated using Ansible](https://github.com/Subbeh/homelab/tree/main/.drone) (ansible-runner), while [kubernetes changes](https://github.com/Subbeh/homelab/tree/main/k8s/management/apps) are picked up by Argo CD.
