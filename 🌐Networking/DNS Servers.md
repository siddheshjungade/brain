---
title: DNS Server
tags:
    - Networking

---
---

![[../../images/networking/4 DNS Server.webp]] 


### 📘 **DNS Summary for om-mapari.com**

- **Domain:** `siddheshjungade.dev`
- **Registrar:** Cloudflare
- **Authoritative Name Servers:**
    - `norm.ns.cloudflare.com`
    - `ollie.ns.cloudflare.com`

---

### ⚙️ **How DNS Lookup Works**

1️⃣ Browser checks local **cache** or **hosts file**.  
2️⃣ If not found → asks **Recursive DNS (ISP)**.  
3️⃣ Recursive DNS → queries **Root DNS** (knows `.com`).  
4️⃣ **TLD (.com)** server → points to **Cloudflare’s NS**.  
5️⃣ **Cloudflare (Authoritative NS)** → returns actual **IP address** of `siddheshjungade.dev`.  
6️⃣ Browser connects to that IP → website loads ✅


### 🔍 **Details Explanation**

#### 1️⃣ The Hosts File
Before the internet uses external servers, it checks a local text file that maps hostnames to IP addresses. This file acts like a private address book that overrides public DNS records.

**File Locations:**
- **Linux (Fedora):** `/etc/hosts`
- **Windows:** `C:\Windows\System32\drivers\etc\hosts`
- **macOS:** `/private/etc/hosts`

**Example entry:**
```
127.0.0.1   localhost
192.168.1.50   homelab.local
```

#### 2️⃣ The Local DNS Cache
The browser and the operating system also maintain a temporary "cache" in memory so they don't have to read the hosts file or query the ISP every single time you refresh a page.

**Browser Cache:** Modern browsers (Chrome, Firefox, Edge) have their own internal DNS cache. You can usually view or "flush" this by navigating to `chrome://net-internals/#dns`.

**OS Resolver Cache:** 
- **Linux (systemd-resolved):** Check statistics and cache status using `resolvectl statistics` or `resolvectl query`.
- **Windows:** View via Command Prompt with `ipconfig /displaydns`.

#### 3️⃣ DNS Configuration Files
If the IP isn't in the hosts file or the cache, the system needs to know which Recursive DNS (ISP) to ask. This is defined in your network configuration files:

**Linux:** `/etc/resolv.conf` — This file typically lists the nameservers (like `8.8.8.8` for Google or `1.1.1.1` for Cloudflare) that your system will contact.

**Systemd-based systems:** Often use `/run/systemd/resolve/resolv.conf` for dynamic configurations.

#### 📋 Summary of the "Local Search" Order:

1. **Browser Cache:** "Did I visit siddheshjungade.dev in the last few minutes?"
2. **OS Cache:** "Did any other app on this laptop ask for this IP recently?"
3. **Hosts File:** "Is there a manual override written in `/etc/hosts`?"
4. **Resolver:** "No? Okay, let's look at `/etc/resolv.conf` and ask the ISP/Recursive DNS."