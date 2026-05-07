# 🌐 How Linux Routing & NAT Works: The "Device C" Bridge

This guide explains how to turn a Linux machine (**Device C**) into a router that connects two private subnets to a WAN gateway (**Device D**).

---

## 📍 The Network Map

* **Subnet A (`192.168.0.0/24`):** Contains **Device A**.
* **Subnet B (`192.168.1.0/24`):** Contains **Device B**.
* **Subnet C (`192.168.2.0/24`):** The "Transit" network connecting our router to the Internet gateway.
* **WAN:** The Public Internet (represented by the IP of your ISP's router).

---

## ⚙️ Netplan Configurations

### Device A (The Client)

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.0.10/24]
      routes:
        - to: default
          via: 192.168.0.1  # Points to Device C
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

keep in mind, the section:

```yaml
routes:
        - to: default
          via: 192.168.0.1  # Points to Device C
```

basically says that ghe gateway of this network is `192.168.0.1` the same way it does on you regular home router.

### Device B (The second Client)

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.1.10/24]
      routes:
        - to: default
          via: 192.168.1.1  # Points to Device C
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

### Device C (The "Router")

*Note: This config shows 3 physical interfaces (`eth0`, `eth1`, `eth2`). If you only had **one** interface, you would simply list all addresses under `eth0`.*

```yaml
network:
  version: 2
  ethernets:
    eth0: # Connection to Subnet A
      addresses: [192.168.0.1/24]
    eth1: # Connection to Subnet B
      addresses: [192.168.1.1/24]
    eth2: # Connection to Subnet C (Transit to WAN)
      addresses: [192.168.2.10/24]
      routes:
        - to: default
          via: 192.168.2.1  # Points to Device D (The "Gateway")
```

## 🚀 Enabling the "Engine" (Device C)

By default, Linux drops packets that aren't meant for it. To turn it into a router, you must enable **IP Forwarding**.

**1. Enable in the Kernel:**

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

To make it permanent, edit `/etc/sysctl.conf` and uncomment the line: `net.ipv4.ip_forward=1`

alternatively, if there is no `/etc/sysctl.conf`, and only a folder named `/etc/sysctl.d/`,
then just add a file in `/etc/sysctl.d/` and call it some name.conf like for example:

```Bash
/etc/sysctl.d/ip_forwarding.conf
```

and add the line:

```Bash
net.ipv4.ip_forward=1
```

then run the command:

```bash
sudo sysctl --system
```

to reload the system control's config
and you can verify the status by running:

```Bash
sysctl net.ipv4.ip_forward
```

if it returns `1` the success, if it returns `0` you have failed :p

**2. Configure the Firewall (UFW):**
You must tell the firewall to allow traffic to pass *through* the device, not just to it.

```bash
# Allow forwarding in UFW settings
sudo sed -i 's/DEFAULT_FORWARD_POLICY="DROP"/DEFAULT_FORWARD_POLICY="ACCEPT"/' /etc/default/ufw

```

* Once you set this to ACCEPT, your Linux box will forward any traffic it receives. If you want to be more secure, you can leave the default as DROP and instead write specific "route" rules for only the subnets you trust:

then if you have things on more than one interface:

```bash
# Allow traffic between the interfaces
sudo ufw route allow in on eth0 out on eth2
sudo ufw route allow in on eth1 out on eth2
```

or:

```bash
# Allow any traffic to be routed as long as it's on eth0
sudo ufw route allow in on eth0 out on eth0
```

if you have all on the same interface. (in general you need ufw to allow all traffic from one interface to another interface [or itself])

finally to enable the ufw:

```bash
sudo ufw enable
```

or:

```bash
sudo ufw reload
```

to reload if it was already enabled...

## 🎭 The "Masquerade" (NAT)

Because **Device D** (the ISP router) doesn't know Subnet A or B exists, **Device C** must "hide" them using its own IP. This is **IP Masquerading**.

(keep in mind that this step is only if you want to route from **Device A** or **Device B** to the ISP, through **Device C**)

**The Command:**

```bash
sudo iptables -t nat -A POSTROUTING -o eth2 -j MASQUERADE
```

or if you only have one interace ( for example `eth0`)

```bash
# "Masquerade only if the packet is NOT staying in the 192.168.0.0/16 range"
sudo iptables -t nat -A POSTROUTING -o eth0 ! -d 192.168.0.0/16 -j MASQUERADE
```

(The `!` means "NOT" and `-d` means "Destination")

Summary Checklist for your Command:
`-t nat`: Essential (tells it which table to use).

`-A POSTROUTING`: Essential (tells it when to swap the IP).

`-o eth0`: Essential (defines the "exit door").

`-j MASQUERADE`: Essential (defines the action).

* **What this does:** When Device A sends a packet to the web, Device C replaces A's IP (`192.168.0.10`) with its own Transit IP (`192.168.2.10`). Device D then thinks the request came from Device C.

## 🛣️ The Life of a Packet (Step-by-Step)

**The Outbound Trip (Device A → Google):**

1. **Device A** realizes `google.com` is outside its network. It sends the packet to its gateway: **192.168.0.1** (Device C).
2. **Device C** receives the packet. It sees `ip_forward=1`, so it looks at the destination. It sends it out towards its own gateway: **192.168.2.1** (Device D).
3. **The Masquerade:** Just as the packet leaves Device C, it changes the "From" address to `192.168.2.10`.
4. **Device D** receives it, changes the "From" address again to your **Public WAN IP**, and sends it to the Internet.

**The Return Trip (Google → Device A):**

1. The Internet sends the data back to your **Public WAN IP**.
2. **Device D** sees the reply and sends it to **Device C** (`192.168.2.10`).
3. **Device C** checks its "Translation Table," remembers Device A asked for this, and swaps the destination back to `192.168.0.10`.
4. **Device A** receives the data.

## 🔍 The Traceroute Reveal

If you run `traceroute 8.8.8.8` from **Device A**, you will see the physical "hops" of this chain:

```text
1  192.168.0.1   (Device C - The Internal Bridge)
2  192.168.2.1   (Device D - The WAN Gateway)
3  10.x.x.x      (Your ISP's first router)
4  ...           (The rest of the Internet)
```

**Why you see your WAN IP on "WhatIsMyIP.org":**
Websites only see the **last** public envelope. Since Device D performs NAT to the public web, the website never sees your local `192.168.x.x` addresses. They are kept private for security!
