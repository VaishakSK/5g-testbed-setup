# 5G Testbed Setup using Open5GS and srsRAN

This repository provides a comprehensive guide to setting up a private 5G testbed using Open5GS as the core network and srsRAN as the RAN. It covers the entire process from installing software dependencies, configuring hardware (USRP B210), programming SIM cards, and running a standalone 5G network for research or development purposes.

---

## 🚀 Features

✅ Deploy a standalone 5G network  
✅ Configure Open5GS core components  
✅ Integrate srsRAN as gNB  
✅ Program custom SIM cards  
✅ Test with real 5G phones  
✅ Troubleshooting tips for stable operation

---

## 🛠️ Hardware Requirements

- PC (Ubuntu 22.04 LTS recommended)
- USRP B210 SDR (Software Defined Radio)
- Programmable SIM cards + USB reader
- 5G-capable smartphone for testing

---

## 🧩 Software Components

- **Open5GS** → Core network (AMF, SMF, UPF, etc.)
- **srsRAN Project** → RAN components (gNB)
- **MongoDB** → Subscriber data storage
- **Node.js** → Web UI for Open5GS
- **UHD drivers** → USRP connectivity
- **Open-Cells tools** → SIM programming

---

## 📋 Installation Steps

### 1. Install MongoDB

```bash
sudo apt update
sudo apt install -y gnupg
curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
````

---

### 2. Install Open5GS

```bash
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install -y open5gs
```

Start services:

```bash
sudo systemctl restart open5gs-*
sudo systemctl status open5gs-* | grep -c "active"
```

---

### 3. Set Up Web UI

```bash
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
sudo apt update
sudo apt install -y nodejs
```

Install Open5GS WebUI:

```bash
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```

Default Web UI:

```
http://localhost:9999
Username: admin
Password: 1423
```

---

### 4. Configure NAT

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -I FORWARD 1 -j ACCEPT
sudo apt install -y iptables-persistent
sudo dpkg-reconfigure iptables-persistent
```

---

### 5. Build and Install srsRAN

```bash
sudo apt update
sudo apt install -y cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev libuhd-dev uhd-host
git clone https://github.com/srsRAN/srsRAN_Project.git
cd srsRAN_Project
mkdir build
cd build
cmake ../
make -j $(nproc)
make test -j $(nproc)
sudo make install
```

---

### 6. Program SIM Cards

Download and extract:

```
https://open-cells.com/d5138782a8739209ec5760865b1e53b0/uicc-v3.3.tgz
```

Compile and run:

```bash
tar -xvzf uicc-v3.3.tgz
cd uicc-v3.3
make
sudo ./program_uicc --adm 12345678 --imsi 999700000000001 --isdn 00000001 --acc 0001 --key 6874736969202073796d4b2079650a73 --opc 504f20634f6320504f50206363500a4f --spn "CSE" --authenticate --noreadafter
```

---

### 7. Set Up USRP B210

Install UHD:

```bash
sudo apt install -y libuhd-dev uhd-host
sudo python3 /usr/lib/uhd/utils/uhd_images_downloader.py
```

Check devices:

```bash
uhd_find_devices
uhd_usrp_probe
```

---

### 8. Run the Network

Start Open5GS core:

```bash
sudo systemctl restart open5gs-*
```

Configure subscribers in Web UI:

* IMSI
* Key
* OPC
* APN: internet

Launch gNB:

```bash
cd srsRAN_Project/build/apps/gnb
sudo ./gnb -c gnb_rf_b200_tdd_n78_20mhz.yml
```

---

## 🐛 Troubleshooting

* MongoDB not running:

  ```bash
  sudo journalctl -u mongod
  ```

* USRP not detected:

  ```bash
  uhd_find_devices
  ```

* gNB fails to connect:

  * Check AMF IP in config files.

* SIM programming errors:

  ```bash
  sudo DEBUG=y ./program_uicc
  ```

---

## 📚 References

* [Open5GS Documentation](https://open5gs.org/open5gs/docs/)
* [srsRAN User Manual](https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html)
* [UHD USRP Docs](https://kb.ettus.com/USRP_Host_Software)

---
