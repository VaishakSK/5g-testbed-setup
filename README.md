
````markdown
# 📡 Private 5G Network Setup – Open5GS + srsRAN + USRP B210

A **comprehensive step-by-step guide** to deploy a **private 5G network** using open-source components.  
This setup integrates **Open5GS** as the 5G Core, **srsRAN Project** as the RAN, and **USRP B210** SDR hardware, along with **programmable SIM cards** for real-device testing.

---

## 🚀 Overview

This guide enables you to:

✅ Deploy a standalone private 5G network  
✅ Configure and run **Open5GS (5G Core)**  
✅ Integrate **srsRAN Project** as **gNB (RAN)**  
✅ Program custom SIM cards  
✅ Test with real 5G phones  
✅ Apply **NAT & routing** for internet access  
✅ Troubleshoot common issues  

---

## 🖥️ Hardware Requirements

| Component   | Recommended Specs |
|-------------|------------------|
| **PC/Server** | Intel i7, 32GB RAM, Ubuntu 22.04 LTS |
| **SDR**      | USRP B210 (USB 3.0) |
| **SIM**      | Programmable SIM + USB card reader |
| **Device**   | 5G-capable smartphone |
| **Network**  | Stable internet connection |

---

## 🧩 Software Components

- **Open5GS** → 5G Core Network (AMF, SMF, UPF, etc.)
- **srsRAN Project** → RAN (gNB)
- **MongoDB** → Subscriber database
- **Node.js** → WebUI for Open5GS
- **UHD drivers** → USRP connectivity
- **Open-Cells tools** → SIM programming

---

## 📋 Installation Steps

### 1️⃣ MongoDB Setup
```bash
sudo apt update
sudo apt install -y gnupg
curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
````

---

### 2️⃣ Open5GS Installation

```bash
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install -y open5gs
sudo systemctl restart open5gs-*
sudo systemctl status open5gs-* | grep -c "active"
```

---

### 3️⃣ Open5GS WebUI Setup

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
sudo apt update
sudo apt install -y nodejs
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```

**Default WebUI:**

```
http://localhost:9999
Username: admin
Password: 1423
```

---

### 4️⃣ NAT & Routing Configuration

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o $(ip route show default | awk '/default/ {print $5}') -j MASQUERADE
sudo iptables -I FORWARD 1 -j ACCEPT
sudo apt install -y iptables-persistent
sudo dpkg-reconfigure iptables-persistent
```

---

### 5️⃣ srsRAN Project Installation

```bash
sudo apt update
sudo apt install -y cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev libuhd-dev uhd-host
git clone https://github.com/srsRAN/srsRAN_Project.git
cd srsRAN_Project
mkdir build && cd build
cmake ../
make -j $(nproc)
make test -j $(nproc)
sudo make install
```

---

### 6️⃣ USRP B210 Setup

```bash
sudo apt install -y libuhd-dev uhd-host
sudo python3 /usr/lib/uhd/utils/uhd_images_downloader.py
uhd_find_devices
uhd_usrp_probe
```

---

### 7️⃣ SIM Card Programming

**Download & compile UICC tools:**

```bash
wget https://open-cells.com/d5138782a8739209ec5760865b1e53b0/uicc-v3.3.tgz
tar -xvzf uicc-v3.3.tgz
cd uicc-v3.3
make
```

**Program SIM:**

```bash
sudo ./program_uicc --adm 12345678 --imsi 999700000000001 --isdn 00000001 --acc 0001 --key 6874736969202073796d4b2079650a73 --opc 504f20634f6320504f50206363500a4f --spn "CSE" --authenticate --noreadafter
```

---

## ⚙️ Configuration

**gNB Configuration (`gnb_rf_b200_tdd_n78_20mhz.yml`):**

```yaml
cu_cp:
  amf:
    addr: 127.0.0.5
    port: 38412
    bind_addr: 127.0.0.5
    supported_tracking_areas:
      - tac: 7
        plmn_list:
          - plmn: "99970"

cell_cfg:
  dl_arfcn: 632628
  band: 78
  channel_bandwidth_MHz: 20
  common_scs: 30
  plmn: "99970"
  tac: 7
  pci: 1
```

---

## 🚀 Running the Network

**Start Core Network:**

```bash
sudo systemctl restart open5gs-*
sudo systemctl status open5gs-* | grep -c "active"
```

**Configure Subscribers in WebUI:**

```
IMSI: 999700000000001
Key: 6874736969202073796d4b2079650a73
OPC: 504f20634f6320504f50206363500a4f
APN: internet
```

**Start gNB:**

```bash
cd srsRAN_Project/build/apps/gnb
sudo ./gnb -c gnb_rf_b200_tdd_n78_20mhz.yml
```

---

## 🔧 Troubleshooting

| Issue                      | Command / Solution            |
| -------------------------- | ----------------------------- |
| **MongoDB not running**    | `sudo journalctl -u mongod`   |
| **USRP not detected**      | `uhd_find_devices`            |
| **gNB fails to connect**   | Check AMF IP in config        |
| **SIM programming errors** | `sudo DEBUG=y ./program_uicc` |

---

## 📊 Testing

**Throughput Test (iperf3):**

```bash
iperf3 -s   # Server
iperf3 -c <server_ip>   # Client
```

**Signal Quality:**
Monitor `gnb` logs for connection stability.

---

## 🛡️ Security & Compliance

* Change default **WebUI credentials**
* Restrict WebUI access to **trusted networks**
* Ensure **compliance with local spectrum regulations**
* Use **licensed bands** or approved experimental bands

---

## 📚 References

* [Open5GS Documentation](https://open5gs.org/open5gs/docs/)
* [srsRAN User Manual](https://docs.srsran.com/projects/project/en/latest/)
* [UHD USRP Docs](https://files.ettus.com/manual/)
* [Open-Cells UICC Tools](https://open-cells.com/)

---

```

---

If you want, I can also make a **visually enhanced version** with **diagrams and workflow images** in the README so it’s not just text but also visually explains the flow of the private 5G setup. That would make it even more appealing for GitHub.
```
