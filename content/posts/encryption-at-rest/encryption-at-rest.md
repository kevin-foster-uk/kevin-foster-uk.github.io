
---
title: "Sleeping Soundly"
draft: false
date: 2026-05-12T7:00:00.000Z
description: "We obsess over TLS. We lock down data *in transit*, but what about when the data stops moving? That is where **encryption at rest** comes in, and it is easy to misunderstand."
categories:
  - Security
  - SaaS
tags:
  - Security
  - SaaS
---

### Adding LUKS Encryption to Your Servers

We obsess over TLS. We lock down data *in transit*, but what about when the data stops moving? That is where **encryption at rest** comes in, and it is easy to misunderstand.

### What Is Encryption at Rest?

It is encryption applied to data stored on a physical disk (HDD or SSD). If someone steals a hard drive, or gains raw access to your cloud storage or colocated hardware, encryption at rest ensures the data is just noise without the key.

### Why Bother?

- **Physical theft:** Servers get decommissioned, laptops get lost, drives get recycled. Encryption renders the data useless.
- **Insider threats:** Even a rogue datacentre engineer or hosting provider staffer cannot read your raw disk blocks.
- **Compliance:** GDPR, HIPAA and PCI-DSS increasingly expect it.

### It Is Not That Common

Despite being critical, encryption at rest is deployed far less often than in-transit encryption. Why? Many assume "the cloud provider handles it" (they often do not by default). Others fear the performance hit or the complexity of managing boot-time passwords for a fleet of mail servers, API servers, database servers, or anything else that runs unattended.

I once saw a service claim: *"We don't encrypt your data at rest because your data is not at rest when used by our system."*

It is a tempting misunderstanding. "At rest" sounds like "inactive", so it is easy to assume that actively read or written data does not count. But any data sitting on a disk, whether your application is busy or idle, is considered "at rest". When we are focused on other issues, it is easy to make assumptions about new concepts. But this is something worth giving time to.

### A Practical Fix: LUKS and Tang

For a recent project, I needed automatic, network-bound decryption for a handful of servers. Enter **Clevis and Tang**.

- **LUKS** (Linux Unified Key Setup) encrypts the partition.
- **Tang** is a stateless network daemon that unlocks LUKS without storing the actual key.
- **Clevis** (on each server) automates the unlock process.

Here is the high-level flow:

1.  **On your storage server or the server you want to protect:** Set up LUKS on the partition you want to encrypt.
2.  **Deploy Tang:** Run a lightweight Tang container somewhere on your network, reachable via HTTPS. It will advertise its public key.
3.  **Bind Clevis:** On each target server (mail, API, database and so on), run:
    ```bash
    sudo clevis luks bind -d /dev/sda1 tang '{"url":"http://tang-server"}'
    ```
4.  **Test:** Reboot the server. Clevis will automatically contact Tang, retrieve the secret and unlock `/dev/sda1`. No human needs to type a password.

This gives you **disk encryption** with **seamless reboots** across your entire fleet. The Tang server acts as an automated key master, but without network access to Tang, the locked server stays locked.

### The Bottom Line

Encryption is a must for any server that holds sensitive data, whether it is a mail server, an API server, a database or a worker. By combining LUKS with Tang and Clevis, we can bridge the gap between "should encrypt" and "actually encrypted", automatically and without forcing someone to run to the datacentre every time a server reboots.

Give it a try. Your sleeping data will thank you.