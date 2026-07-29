# ⚡ KIX Deployment Suite: Autonomous Payment Layer

![Bitcoin](https://img.shields.io/badge/Bitcoin-Lightning-orange)
![Docker](https://img.shields.io/badge/Docker-Required-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Architecture](https://img.shields.io/badge/Focus-Autonomy-black)

KIX is an **Independent Merchant Framework** designed to integrate businesses with the Bitcoin Lightning Network. Engineered as a direct response to centralized payment architectures and automated fiscal tracing models (such as Brazil's upcoming automated *split payment* mechanisms), KIX replicates the streamlined user experience of instant QR-code payments while preserving financial operational independence.

🌐 General information and documentation:  
https://satoshicanvas.com/kix-eng/

---

# 🇧🇷 The "Pix" Strategy: Familiarity and Adoption

Brazil successfully trained 150 million people to adopt instant QR-code transactions. KIX leverages this established habit to reduce friction in alternative payment adoption.

* **Familiar Flow:** KIX maintains a user experience identical to standard instant payment rails, avoiding complex onboarding barriers.
* **Operational Autonomy:** By utilizing the Lightning Network as a settlement layer, merchants operate independently from traditional banking intermediaries and potential account restrictions.
* **Value Retention:** Designed to optimize merchant margins by eliminating high intermediary fees associated with conventional credit and debit processors.

---

# 🏗️ System Architecture

                 ┌────────────────────────────┐
                 │        Merchant UI         │
                 │  (KIX Dashboard-homepage)  │
                 └─────────┬──────────────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │      LNbits       │
                 │  Payment Engine   │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │  Lightning Node   │
                 │ Phoenixd / Alby   │
                 └─────────┬─────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
        Tor Hidden Service     VPS Gateway
          (.onion access)       (Port 80)


The system utilizes Docker encapsulation to provide horizontal scalability on a single host. Each instance contains:

* **Dashboard:** A centralized web interface (Homepage) for management.
* **Engine (LNbits):** A powerful Lightning Network payment processor.
* **Vault (Phoenixd / Alby Hub):** Secure key management and lightweight node connectivity.
* **Tor Bridge:** An Alpine-based routing container that generates unique Onion URLs in real-time.

---

# 🚀 Getting Started

## 1. Requirements

* **Docker**
* **Docker Compose**
* **Sudo privileges** (required for volume management and reading Tor hostnames)
* **Entropy helper (optional but recommended for Tor)**

If Tor is slow generating keys:

```bash
sudo apt install haveged
```

Docker must be installed and running before executing any deployment scripts.

---

# 2. Installation & Deployment Methods

| Method | Script | Description |
| :--- | :--- | :--- |
| 1. **Autonomous** | kix_tor_sovereign.sh | 🌑 Advanced privacy deployment via Tor (.onion). Runs LNbits with **Alby Hub**. Requires configuring the **Nostr key from AlbyHub inside LNbits**. |
| 2. **Phoenixd** | kix_phoenixd.sh | 🔥 Ultra-lightweight multi-instance Phoenix node + LNbits + Tor. |
| 3. **Clearnet VPS** | kix_vps_dedicated.sh | 🌍 Public clearnet deployment on a dedicated VPS (Port 80). |

---

# 2.1 The Phoenixd Multi-Hunter Method (Recommended)

This method uses the `kix_phoenixd.sh` orchestrator to deploy isolated Phoenix nodes. It includes automatic volume management and environment backups.

### Setup

```bash
sudo chmod +x kix_phoenixd.sh
```

### Deploy Instance (Default is v1)

```bash
./kix_phoenixd.sh v1
```

### Deploy Instance "v8"

```bash
./kix_phoenixd.sh v8
```

### How it works

**Isolation**

Creates a directory like:

```text
~/PHOENIX_[ID]
```

Each instance has unique Docker volumes preventing collisions.

### Automatic LNbits Configuration

The `kix_phoenixd.sh` script automatically configures **LNbits** with the correct **Phoenixd API endpoint and password**.

After deployment:

* LNbits is already connected to the Phoenix node
* The wallet is ready to **generate Lightning invoices immediately**
* No manual API configuration is required

### Important

Phoenix automatically opens its **first Lightning channel** when the wallet receives funds.

⚠️ The **first ~20,000 sats** are used by **ACINQ** to open the initial channel.

For this reason it is **critical to safely store the Phoenix seed words**.  
The seed is the only way to recover the wallet and its funds if the node or server is lost.

**Credentials**

The script generates:

* a **16-hex API password**
* `.env` file
* `env_backup.txt`

**Discovery**

The script scans Docker volumes to locate:

```text
seed.dat
```

and prints the generated **Tor .onion address**.

---

# 2.2 The Autonomous Method (Tor + Alby Hub)

This is the **maximum autonomy configuration**.

It runs:

* LNbits
* Tor hidden service
* **Alby Hub wallet backend**

LNbits must be connected to Alby Hub using the **Nostr key**.

### AlbyHub Configuration

1. Open **Alby Hub**
2. Copy your **Nostr private key**
3. Paste it inside **LNbits settings**
4. This links LNbits to the Alby Hub wallet.

### Run

```bash 
sudo chmod +x kix_tor_sovereign.sh
```

```bash
./kix_tor_sovereign.sh 1
```

### Access

Use **Tor Browser** and open the `.onion` URL printed at the end of the script.

Allow **1–2 minutes** for Tor circuit propagation.

---

## 2.2.1 The Multi-Instance Bind Mount Method

This is the evolution of the Autonomous Method, specifically engineered for high-availability and easy backups. Unlike standard Docker volumes, this method uses Bind Mounts, mapping the container data directly to visible folders in your home directory.

### Key Advantages:

- **Data Visibility:** All Lightning database and Tor keys are stored in `~/KIX_PROTOTIPO[ID]/data`.
- **Easy Backups:** You can backup your entire node by simply copying a local folder, without complex volume export commands.
- **Robustness:** Prevents data loss during Docker updates or container migrations.

### Deployment (the number at the end is the instance number):

```bash 
sudo chmod +x kix_multi_tor_bind_mount.sh
./kix_multi_tor_bind_mount.sh 9
```

### Folder Structure Created:

The script automatically organizes your operational directories:

- `~/KIX_PROTOTIPO9/data/lnbits`: SQLite databases and extensions.
- `~/KIX_PROTOTIPO9/data/alby`: Your Alby Hub keys and settings.
- `~/KIX_PROTOTIPO9/data/tor`: Permanent `.onion` addresses (they won't change if the container restarts).

### Post-Deployment:

After the script prints your `.onion` links, remember to:

- Access your Alby Hub via the Tor link.
- Link it to LNbits using the Nostr Wallet Connect (NWC) or the account's internal key.

Your data is now persistent and physically located at `~/KIX_PROTOTIPO9`.


# 2.3 Dedicated VPS Method (Clearnet)

Use this when the VPS is **dedicated exclusively to KIX** and you want maximum performance on **Port 80**.

### Run

```bash 
sudo chmod +x kix_vps_dedicated.sh
```

```bash
./kix_vps_dedicated.sh 1
```

---

## Future Integrations & Compliance Note

As state architectures evolve toward automated multi-party fiscal diversion systems (such as mandatory automated tax splitting at the point of sale), KIX serves as a non-split-payment baseline ("nusplit" reference model) ensuring native self-custody. Future iterations may include optional export hooks and modular plugins designed to interface cleanly with government electronic invoice standards (such as **Nota Fiscal / NFE APIs**), allowing merchants to maintain transparent fiscal compliance while preserving local control over settlement liquidity.

---

## Monitor Container Resources

View real-time network and disk I/O:

```bash
sudo docker stats
```

---

## Cleanup Dangling Docker Data

Remove unused Docker volumes from old experiments:

```bash
sudo docker volume prune -f
```

---

# 🔐 Security Disclaimer

KIX is an open-source framework for financial management and infrastructure independence.

Users are responsible for:

* Their **keys and backups**
* Their **node security posture**
* Their **local regulatory and fiscal compliance**

---

# 📦 Repository Metadata

**Repository:** `kix-protocol-suite`

**Topics**

```text
bitcoin
lightning-network
lnbits
phoenixd
albyhub
pix-brazil
self-custody
privacy
autonomy
```
