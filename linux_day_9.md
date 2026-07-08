The Nautilus system administrators team has rolled out a web UI application for their backup utility on the Nautilus application server 1 within the Stratos Datacenter. This application runs on port 8086and appropriate firewall rules must be configured to allow incoming traffic. To achieve this, firewalld needs to be installed and configured on the application server. To ensure proper functionality, the following requirements have been identified:



Install and enable the firewalld service.
Allow all incoming connections on port 8086/tcp.
Ensure the zone is set to public.


#solution

This is another common **KodeKloud** task. The objective is to install and configure **firewalld** so the web application running on **TCP port 8086** is reachable.

---

# Step 1: SSH into App Server 1

```bash
ssh tony@stapp01
```

(or use the username provided in your lab)

Become root:

```bash
sudo su -
```

---

# Step 2: Install firewalld

### On CentOS/RHEL/Rocky/AlmaLinux

```bash
yum install -y firewalld
```

or on newer systems:

```bash
dnf install -y firewalld
```

**Why?**

`firewalld` is the firewall management service used on Red Hat-based distributions.

---

# Step 3: Enable and start the service

```bash
systemctl enable firewalld
systemctl start firewalld
```

Verify:

```bash
systemctl status firewalld
```

Expected:

```text
Active: active (running)
```

**Why?**

* `start` → starts it immediately.
* `enable` → starts it automatically after reboot.

---

# Step 4: Set the default zone to `public`

Check the current default zone:

```bash
firewall-cmd --get-default-zone
```

If it isn't `public`, set it:

```bash
firewall-cmd --set-default-zone=public
```

Verify again:

```bash
firewall-cmd --get-default-zone
```

Expected:

```text
public
```

**Why?**

The task specifically requires the **public** zone to be used.

---

# Step 5: Allow port 8086/tcp permanently

```bash
firewall-cmd --permanent --zone=public --add-port=8086/tcp
```

**Why `--permanent`?**

Without it, the rule disappears after a reboot or firewall reload.

---

# Step 6: Reload the firewall

```bash
firewall-cmd --reload
```

**Why?**

Permanent changes are applied only after reloading.

---

# Step 7: Verify the rule

```bash
firewall-cmd --zone=public --list-ports
```

Expected:

```text
8086/tcp
```

Or verify directly:

```bash
firewall-cmd --query-port=8086/tcp
```

Expected:

```text
yes
```

---

# Step 8: Verify the service

```bash
systemctl is-enabled firewalld
```

Expected:

```text
enabled
```

And:

```bash
systemctl is-active firewalld
```

Expected:

```text
active
```

---

# Final Commands (Quick Solution)

```bash
sudo su -

yum install -y firewalld

systemctl enable firewalld
systemctl start firewalld

firewall-cmd --set-default-zone=public

firewall-cmd --permanent --zone=public --add-port=8086/tcp

firewall-cmd --reload

firewall-cmd --query-port=8086/tcp
firewall-cmd --get-default-zone
```

---

## KodeKloud Exam Tips

* Always use `--permanent` unless the question explicitly asks for a temporary rule.
* After making permanent changes, always run `firewall-cmd --reload`.
* If the task mentions a specific zone (like `public`), add the rule to that zone and verify the default zone if requested.
* A quick verification checklist before submitting:

  * `systemctl status firewalld` → `active (running)`
  * `systemctl is-enabled firewalld` → `enabled`
  * `firewall-cmd --get-default-zone` → `public`
  * `firewall-cmd --query-port=8086/tcp` → `yes`
