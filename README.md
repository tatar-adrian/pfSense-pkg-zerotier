# pfSense-pkg-zerotier
pfSense package to support zerotier.

# The package is provided as is so use it on your own risc

ZeroTier is not available as a built-in package in pfSense, so you need to install it manually using SSH.

**Tested on pfSense 2.7.2**

#### 1. SSH into pfSense

Activate Shell account access to admin account and ssh as root to the firewall

#### 2. Install ZeroTier
Run the following commands to download and install ZeroTier:

```sh

pkg update

pkg install nano

pkg add https://pkg.freebsd.org/FreeBSD:14:amd64/release_0/All/zerotier-1.12.2.pkg

pkg add https://github.com/asvdvl/pfSense-pkg-zerotier/releases/download/2.7/pfSense-pkg-zerotier-0.00.1.pkg
```
---

### 3. Configure ZeroTier
- Tunable

If you are running other daemons or require firewall rules to depend on
zerotier interfaces being available at startup, you may need to enable
the following sysctl in /etc/sysctl.conf:

1. In WebUi Go to **System** → **Advanced** → **System Tunables**.
2. Click **New** to create a new record:
   - **Tunable**: net.link.tap.up_on_open
   - **Value**: 1
   - **Description**: ZeroTier config
3. Click **Save** and **Apply Changes**.

- Allow zerotier in webui

open file `nano /var/db/zerotier-one/local.conf`

```json
{
  "settings": {
      "allowManagementFrom": [ "127.0.0.1", "::1", "ffff:127.0.0.1" ]
  }
}
```

- Enable and start the ZeroTier service:

`sudo nano /etc/rc.conf`

add the line

```sh
sysrc zerotier_enable="YES"
```

start the service

    service zerotier start

After this changes is best to reboot pfsense

- Setup from WebUi

1. In WebUi go to **VPN** → **Zerotier** → **Configuration**.

Check **Enable** From **Enable Zerotier** section and **Save**

#### 4. Join a ZeroTier Network
1. Log in to your ZeroTier Central account: [https://my.zerotier.com](https://my.zerotier.com).
2. Create a new network and copy the **Network ID**.
3. Join the ZeroTier network on pfSense:

Warning! From the web interface you can't join a network, but from cli you can.

```sh
zerotier-cli join <your-network-id>
```

4. Verify the connection:

```sh
zerotier-cli listnetworks
```

You should see your **Network ID** along with the status.

After you have joined go back in the WebUi and check if the network is displayed

In WebUi go to **VPN** → **Zerotier** → **Networks**.

---

### **5. Assign an Interface for ZeroTier**
1. Go to **Interfaces** → **Assignments**.
2. Find **ZeroTier** (ztxxxxxxxxxxxxx) in the dropdown and assign it.
3. Click **Save** and **Apply Changes**.
4. Go to **Interfaces** → **ZeroTier**, enable it, and set it as a **static IP (e.g., 10.10.10.1/24)**.

---

### **6. Test Connectivity**
- Run `ping` from a client connected to ZeroTier to your pfSense LAN IP.
- Use `traceroute` to ensure traffic is routing correctly.
- Check **Status** → **System Logs** for any errors.

---

### **7. WARNING !**

Since this setup is done outside pfsense WebUi and the plugin and packages were installed manually, when you upgrade pfsense there is a strong posibility that this functionality to be removed. Then you will need to recheck and redo some or all of the stepts written above.

Reference:

how to fix zerotier for freebsd 14 & pfsense 2.7.X:
- [instructions taken from here](https://discuss.zerotier.com/t/freebsd-14-0-zerotier-401-error/16919)
