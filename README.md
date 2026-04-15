# 🔐 VPS + Tailscale Hardening Guide (Hermes / Agents Setup)

This guide sets up a **fully locked-down VPS** that is:

- ❌ Not exposed to the public internet
- ✅ Accessible only via Tailscale (private network)
- ✅ Secured with SSH keys (no passwords)

**Ideal for:** Hermes agents, scraping nodes, automation servers

---

## 🧠 TL;DR

- Use SSH keys (no passwords)
- Connect via Tailscale (100.x.x.x IP)
- Block all public access (especially port 22)

> We're not securing SSH — we're removing it from the internet entirely.

---

## 🔑 Phase 0 — Generate SSH Key (Local Machine)

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Check keys:

```bash
ls ~/.ssh
```

You should see:

```
id_ed25519
id_ed25519.pub
```

Copy public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

👉 Paste this into Hetzner when creating the server

---

## 🧱 Phase 1 — Server Setup

1. **Create VPS**
   - Ubuntu 24.04 LTS
   - Add your SSH public key

2. **Connect as root**

```bash
ssh root@YOUR_SERVER_IP
```

(or if using custom key)

```bash
ssh -i ~/.ssh/your_key root@YOUR_SERVER_IP
```

3. **Update system**

```bash
apt update && apt upgrade -y
```

---

## 👤 Phase 2 — Create Secure User

```bash
adduser hermes
usermod -aG sudo hermes
```

Switch user:

```bash
su - hermes
sudo ls -la /root
```

---

## 🔗 Phase 3 — Install Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Connect:

```bash
sudo tailscale up
```

Get private IP:

```bash
tailscale ip -4
```

Example output:

```
100.x.x.x
```

---

## 🔑 Phase 4 — Enable SSH for User

```bash
sudo mkdir -p /home/hermes/.ssh
sudo cp /root/.ssh/authorized_keys /home/hermes/.ssh/authorized_keys
sudo chown -R hermes:hermes /home/hermes/.ssh
```

Fix permissions:

```bash
chmod 700 /home/hermes/.ssh
chmod 600 /home/hermes/.ssh/authorized_keys
```

Test login:

```bash
ssh hermes@100.x.x.x
```

---

## 🔒 Phase 5 — Disable Password SSH

Edit config:

```bash
sudo nano /etc/ssh/sshd_config
```

Set these values:

```
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## 🔥 Phase 6 — UFW Firewall (Server-Level)

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow in on tailscale0 to any port 22
sudo ufw enable
sudo ufw status verbose
```

---

## 🌐 Phase 7 — Hetzner Firewall (Network-Level)

In **Hetzner Console**:

**Inbound rules:**
- ✅ Allow UDP 41641 (Tailscale)
- ❌ DO NOT allow port 22
- ❌ DO NOT allow anything else

**Outbound:**
- Allow all (default)

---

## ✅ Final Result

Your server is now:

- ❌ Not reachable from public internet
- ❌ SSH not exposed publicly
- ✅ Accessible only via Tailscale
- ✅ Key-based authentication only

---

## 🧠 How to Connect

❌ This should FAIL:

```bash
ssh hermes@PUBLIC_IP
```

✅ This should WORK:

```bash
ssh hermes@100.x.x.x
```

---

## ⚡ Optional: SSH Config (Local)

Create/edit `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Add:

```
Host hermes-vps
  HostName 100.x.x.x
  User hermes
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes
```

Connect with:

```bash
ssh hermes-vps
```

---

## 📚 Reference

- [Tailscale Documentation](https://tailscale.com/docs/)
- [Hetzner Cloud](https://www.hetzner.cloud/)
- [UFW Firewall Guide](https://help.ubuntu.com/community/UFW)
