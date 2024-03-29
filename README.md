# Personal Jenkins

**This is work in progress, actually at its very beginning. Please be patient! Completion might take weeks.**

This repo is meant to implement a Jenkins for personal use on Ubuntu Server.

## Tasks at hand

- Create a working compose-file and install a Caddy-proxy

## The idea ...

... is to take a VM or some local low-cost-server and implement a Jenkins CI/CD to locally build and deploy things. Let us see how far we can take it.

## Hardware

This is just meant to be an example: My own Hardware is a Fujitsu S720 Thinclient. I bought this from a remarketer at 7€ per piece (pack of ten, I like playing) and pimped it with 8GB RAM and a new 250GB mSATA SSD. Overall costs for this device: 58€. It offers me an AMD DualCore CPU, good enough. Theoretically any other decent sized Thinclient that can be upgraded to some acceptable size should do fine. You want 4-8 GB RAM and above 120GB disk, the later clearly depending on the projects you build. I see no need to spend more than 70€ in total. The trick is: mine uses only about 11W in normal operation. This reduced the costs to below any price for a suitable Internet VM from any of the providers I know. So it really makes sense to run such a box at home: costs, control, security, nerd-factor, everything optimal ;-).

## Location

Run this at home. We solve the webhook-matter using some Internet Service, so you do not need to be able to access this Box from the Internet, even if your GIT repos are at Github or Bitbucket or alike. Run Jenkins at home in some corner. The hardware is fanless, so it is silent and in the Winter contributes about 11W to your heating, as well ;-).

## Setup

The setup runs in phases:

- You are responsible to build your hardware and install Ubuntu Server (mininal) on it. You will need Internet-Connection. Once you got this ready, this little project kicks in.
- Some manual configurations will be required.
- An Ansible-playbook will do the main work.
- Some config in the Jenkins will complete the box.

## Goals

In the end I will have a Jenkins running on a Box in my home/office. It requires Internet, but it does not require to be reachable from the Internet. The box will be reasonable secure and somewhat hardened. I will be able to receive notifications (webhooks) from Github or alike to initiate builds when code is pushed. Jenkins will build using docker containers which give me maximum freedom in tailoring my build-environment. Where Jenkins puts the build-artefacts is absolutely free to choose. For my own part: I will use this for mkdocs and hugo to generate websites from code. Webspace is cheap, github is reachable from everywhere, build and deployment is automated, then, and I assume site generators to be way more secure than any CMS. Many reasons to do this little project.

## System

The playbook is made for Ubuntu 22.04 Server LTS. I assume the full server install, though the minimal install might work as well. In a later phase I will add a shell-script to add missing packages for the minimal install.

## The playbook

### Upgrades

At first some packages will be updated if necessary. These would have initially been installed manuall or via a shell-script.

### Docker via the original docker-repository

This part will:

  - register Docker's repo key
  - add Docker's repo
  - install docker-ce

This part is comprised in the docker-role.

### Fixes to the admin user

The admin user (default: prefect) will be added to the docker group and the alias play will be configured.

### jkagent user

The jkagent user will be configured to be an agent with docker support. The user will be created by ansible, but the integration as agent to Jenkins remains a manual task:

  - create an access key pair for SSH in Jenkins
  - add the public key to the authorized keys of the user

The user will already be added to the docker group.

### SSH and fail2ban

Two roles will harden SSH to:

  - only allow pubkey authentication (no username/password via SSH)
  - only allow login to users that are members of group ssh-users
  - use fail2ban to lock out users trying a brute force attack

An explicit restart of the daemon is still missing. Somehow it seems to be inactive after install.

I see no need to tighten the ruleset for fail2ban as this device is local anyway and usually not exposed to the Internet.

### OpenJDK

OpenJDK is required to have an agent.

