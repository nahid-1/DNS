

📘 **bronahid.lab – DNS Infrastructure Setup (BIND9 + Redundant NS + DoH + Nginx)**

This repository contains configuration files and documentation for deploying a **production-ready DNS infrastructure** on Ubuntu 22.04 LTS using:

* **BIND9** – Authoritative & Recursive DNS
* **Redundant Nameservers (ns1 + ns2)**
* **Glue Records**
* **Forward/Reverse Zone Files**
* **Nginx Web Application Hosting**
* **DNS over HTTPS (DoH)** using `cloudflared`
* **Systemd-resolved integration**

This project is built to simulate an enterprise-level DNS environment for the domain **`bronahid.lab`** with high availability and secure DNS transport.

---

## 🚀 **Architecture Overview**

```
                 ┌────────────────────┐
                 │   Client Machine   │
                 │  (resolves via ns1 │
                 │   or DoH gateway)  │
                 └─────────┬──────────┘
                           │
                ┌──────────┴──────────┐
                │    DNS Infrastructure│
                └──────────────────────┘
     ┌────────────────────┐     ┌───────────────────────┐
     │   ns1.bronahid.lab │     │    ns2.bronahid.lab    │
     │ IP: 172.16.30.34   │     │ IP: 172.16.30.40       │
     │ (Master Authoritative)    │ (Secondary / Slave DNS) │
     └───────────┬────────┘     └───────────────┬────────┘
                 │                              │
                 ▼                              ▼
     ┌──────────────────────┐        ┌──────────────────────┐
     │ NGINX + DoH Gateway  │        │ Web Server (optional) │
     │ 172.16.30.73 (DoH)   │        │    172.16.30.73       │
     └──────────────────────┘        └──────────────────────┘
```

---

## 🛠️ **Features Implemented**

### ✔️ **Primary and Secondary DNS (Master–Slave BIND9)**

* `ns1.bronahid.lab` → Master
* `ns2.bronahid.lab` → Slave (zone transfer enabled)

---

### ✔️ **Forward Lookup Zone (`bronahid.lab`)**

Includes:

* `A` records
* `NS` records
* `MX` records
* Host entries (www, mail, dns)

---

### ✔️ **Reverse Lookup Zone (`30.16.172.in-addr.arpa`)**

PTR records for IP → hostname mapping.

---

### ✔️ **Glue Records**

Useful when **nameservers are inside the domain they serve**.
Example:

```
ns1.bronahid.lab → 172.16.30.34
ns2.bronahid.lab → 172.16.30.40
```

---

### ✔️ **DNS over HTTPS (DoH)**

DoH server runs using **cloudflared**:

```
https://doh.bronahid.lab/dns-query
```

This encrypts DNS queries for security.

---

### ✔️ **Nginx Web Application hosting**

Static or dynamic website served from:

```
/var/www/bronahid.lab/
```

With valid A record:
`www.bronahid.lab → 172.16.30.73`

---

## 📂 **Repository Contents**

```
├── named.conf.local
├── named.conf.options
├── db.bronahid.lab
├── db.reverse (db.172)
├── doh-nginx.conf
├── cloudflared-config.yaml
└── README.md
```

---

## ⚙️ **How to Deploy (Summary)**

### 1️⃣ Install BIND9

```bash
sudo apt update
sudo apt install -y bind9 bind9utils bind9-doc
```

---

### 2️⃣ Configure `/etc/bind/named.conf.options`

Includes:

* recursion
* forwarders
* DNSSEC
* listening on correct interfaces

---

### 3️⃣ Configure Zones (Forward + Reverse)

Add to `/etc/bind/named.conf.local`:

```bash
zone "bronahid.lab" {
    type master;
    file "/etc/bind/db.bronahid.lab";
};

zone "30.16.172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.172";
};
```

---

### 4️⃣ Create Zone Files

`db.bronahid.lab`
`db.172`

(Stored in this repo)

---

### 5️⃣ Set up ns2 (Slave DNS)

```
allow-transfer { 172.16.30.40; };
also-notify { 172.16.30.40; };
```

On ns2:

```
type slave;
masters { 172.16.30.34; };
```

---

### 6️⃣ Restart DNS

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

---

## 🔐 **DNS Over HTTPS Setup Summary**

Install cloudflared:

```bash
sudo apt install cloudflared
```

Configure DoH server:

```
/etc/cloudflared/config.yml
```

Expose via NGINX reverse proxy:

```
https://doh.bronahid.lab/dns-query
```

---

## 🌐 **Nginx Web Application**

Create the root folder:

```bash
sudo mkdir -p /var/www/bronahid.lab
sudo nano /etc/nginx/sites-available/bronahid.lab
```

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/bronahid.lab /etc/nginx/sites-enabled/
nginx -t
sudo systemctl reload nginx
```

---

## 🧪 **Testing Commands**

### Test forward resolution:

```bash
dig bronahid.lab @172.16.30.34
```

### Test reverse resolution:

```bash
dig -x 172.16.30.34 @172.16.30.34
```

### Test DoH:

```bash
curl -H 'accept: application/dns-json' \
'https://doh.bronahid.lab/dns-query?name=bronahid.lab&type=A'
```

---

## 🧑‍💻 **Author**

**Nahid Abdullah**
System Administration & Network Enthusiast
BIND9 | DNS Security | Nginx | Web Hosting | Linux

---

## ⭐ Contributions

Pull requests are welcome — especially improvements to DNS security, DoH implementation, and automation scripts.

---

## 📜 License

This project is licensed under the MIT License.

---




