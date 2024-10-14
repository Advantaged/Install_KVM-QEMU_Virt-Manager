# Install_KVM-QEMU_Virt-Manager.md

### Step 1: Install packages

1. Update system, install only needed packages, not confirm for installation & list of packages to be installed.

```
paru -Syu --needed --noconfirm qemu-full iptables-nft libvirt virt-manager dnsmasq vde2 edk2-ovmf dmidecode \
bridge-utils spice-vdagent xf86-video-qxl

```

2. Additional optional packages:

`paru ebtables` & choose the best available

`paru virt-viewer` a kind of light-weight "virt-manager"

### Step 2: Here's a breakdown of the additional packages and their relevance:

1. `ebtables`:

* **Purpose:** It's a tool used for bridging firewall (similar to iptables but for Layer 2 traffic, useful for
filtering on bridge interfaces).
* **Is it necessary?:** If you plan to have more advanced firewall rules on your bridge or more control over network
traffic at Layer 2, it could be helpful. However, for a basic bridged setup, you can skip it unless specific use cases
arise.

2. `virt-viewer`:

* **Difference from** `virt-manager`:
	* `virt-manager`: Full management interface for virtual machines, including creation, configuration, and console
access.
	* `virt-viewer`: A more lightweight application focused on connecting to and viewing a VM's graphical console,
typically over SPICE or VNC.
* **Should you install?:** If you want a simple viewer for VM consoles without the full `virt-manager` interface, it
can be useful. But if you're already using `virt-manager`, this might not be necessary unless you need a lightweight
alternative.

3. `spice-vdagent`:

* **Purpose:** Provides better integration between host and guest in VMs running SPICE (enhanced graphical features like
clipboard sharing, resolution resizing, and seamless mouse control).
* **Is it necessary?:** If you’re using SPICE as your display protocol (common for Linux VMs with SPICE), it's highly
recommended for better performance and integration.

4. `xf86-video-qxl`:

* **Purpose:** This is a video driver for QXL devices, which are used in SPICE-based virtual machines.
* **Is it necessary?:** If your VM will be using SPICE for display, this driver improves performance and allows higher
resolutions. If you're using QXL video with SPICE, it's worth installing.


### Step 3: Configure Firewall

1. Then set `firewall_backend="iptables"` option in `/etc/libvirt/network.conf`.
* Use `nano` with admin-privileges, e.g. `sudo nano /etc/libvirt/network.conf`.
* Here just add to last line: `#firewall_backend = "nftables"` the new line: `firewall_backend="iptables"`
	* See following last two lines below:

```
#firewall_backend = "nftables"
firewall_backend="iptables"

```


### Step 4: Add User to Libvirt Arch

* To add a user to the `libvirt` group on Arch Linux, follow these steps:

1. Check if the group exists: Run `getent group | grep libvirt` to verify if the `libvirt` group already exists. If it
does, proceed to the next step. If not, create it using `sudo groupadd --system libvirt`.
2. Add the user to the group: Use the following command to add the user to the `libvirt` group:


```
sudo usermod -aG libvirt <username>

sudo usermod -aG libvirt tony

```

* Replace `<username>` with the actual username you want to add to the group as shown above.
* Find user-name with command `whoami`.

* **Note:** If you are adding the current user, you will need to log out and back in for the new group membership to
take effect.

* **Verify the group membership: Run** `id <username>`, in my case `id tony`, to confirm that the user has been added
to the `libvirt` group. The command & output should look like this:

```
id tony
uid=1000(tony) gid=1000(tony)
groups=1000(tony),3(sys),90(network),998(wheel),996(audio),991(lp),987(storage),985(video),984(users),981(rfkill),956(
libvirt)

```

### Step 5: Configure `libvirt`

1. **Edit the libvirtd configuration file (optional):**
* If you want to grant the user access to the advanced networking options or to start `virt-manager` without password,
uncomment the `unix_sock_group` line and set it to `libvirt` in the `/etc/libvirt/libvirtd.conf` file. For example:

`sudo nano /etc/libvirt/libvirtd.conf`

* Uncomment the line `unix_sock_group = "libvirt"` and save the changes.

* **Enable & Start service at once:**
	* If you not yet enabled the `libwirt` service. use the command below.

`sudo systemctl enable --now libvirtd.service`


* **Restart the libvirtd service:**
	* In case the service was started before modify configuration file, after modifying the configuration file,
restart the `libvirtd` service to apply the changes:

`sudo systemctl restart libvirtd`

2. Check status of service:

`sudo systemctl status libvirtd`

the Output should be like this:
```
systemctl status libvirtd
● libvirtd.service - libvirt legacy monolithic daemon
     Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; preset: disabled)
     Active: active (running) since Sat 2024-10-12 11:50:59 CEST; 53s ago
 Invocation: 8c278fad6bc946bba70c417b81f31b6f
TriggeredBy: ● libvirtd-ro.socket
             ● libvirtd-admin.socket
             ● libvirtd.socket
       Docs: man:libvirtd(8)
             https://libvirt.org/
   Main PID: 316419 (libvirtd)
      Tasks: 21 (limit: 32768)
     Memory: 10.9M (peak: 11.5M)
        CPU: 294ms
     CGroup: /system.slice/libvirtd.service
             └─316419 /usr/bin/libvirtd --timeout 120

Okt 12 11:50:59 cachyos-zfs-disk-2 systemd[1]: Starting libvirt legacy monolithic daemon...
Okt 12 11:50:59 cachyos-zfs-disk-2 systemd[1]: Started libvirt legacy monolithic daemon.
```

* With these steps, the user should now be able to manage virtual machines using `virsh` and other `libvirt` tools
without needing to use `sudo`.

### Step 6: Disable LXC in Virt-Manager

* **Note:** In some cases `virt-manager` configure the connection to data-carrier (CD) to use "LXC" (Linux-Container),
that, by adding a new VM ask for installing a `virt-bootstrap` from AUR. You should use "QEMU/KVM" instead. To do this
follow the instruction below:

**Based on the provided search results, here’s a step-by-step guide to disable LXC in Virt-Manager on Arch Linux:**

1. **Check if LXC is enabled:** Open Virt-Manager and go to the “Edit” menu. If you see “LXC” as a connection option,
it’s currently enabled.

2. **Disconnect and remove LXC connection:** Right-click on the LXC connection and select “Disconnect”. Then, click on
“Remove Connection” to remove/delete it from the list.

3. **Configure Virt-Manager to use KVM:** Go to “File” > “Add Connection” and select “QEMU/KVM” as the hypervisor. This
will allow you to create and manage KVM-based virtual machines.

4. **Verify LXC is disabled:** After adding the KVM connection, Virt-Manager should no longer show LXC as an option.
You can verify this by going back to the “Edit” menu.

**Additional notes:**

1. **Make sure you have the `libvirt` and `kvm` packages installed on your system.**

2. If you’re using a non-root user, ensure you’re a member of the `libvirt-qemu` and `kvm` groups to use system-level
virtual machines (qemu:///system).
3. You can check the Virt-Manager logs for any errors or issues related to LXC.

By following these steps, you should be able to disable LXC in Virt-Manager on Arch Linux and use KVM instead.


### Step 7: Create and Activate the Bridge

* **Note:** i try to make the bridge with help of "ChatGPT", but finally didn't work, sorry! As soon i found god or
reliable instruction on this… i will add the final instruction for the bridged Net-connection.

1. Create the Bridge Configuration: First, create the file `/etc/netctl/bridge` and populate it with the following
content:

`sudo nano /etc/netctl/bridge`
Content:

```
Description="Bridge connection"
Interface=br0
Connection=bridge
BindsToInterfaces=(enp8s0)
IP=dhcp

```
* **Note:** If your router assign your PC a static IP, you should let the configuration use `dchp`.


* You can keep the static IP you mentioned (`192.168.178.20/24`) if you want your bridge to use this specific IP. If
your network manages IP addresses dynamically, you can change `IP=192.168.178.20/24` to `IP=dhcp` to allow the bridge
to get an IP automatically.

2. Disable the Current Network Interface (`enp8s0`): You will now need to disable the existing `enp8s0` interface to
let the bridge (`br0`) take over:

```
sudo netctl stop enp8s0 &
sudo netctl disable enp8s0

```

3. Enable and Start the Bridge: Now, enable and start the bridge profile:

```
sudo netctl enable bridge &
sudo netctl start bridge

```

4. Verify Bridge Activation:

* Use the `ip a` command to check if the bridge `br0` is up and bound to enp8s0:

`ip a`

* You should see `br0` in the list with the IP address and `enp8s0` enslaved under it.

5. Configure VMs in `virt-manager`: After the bridge is set up, go into `virt-manager` and assign the `br0` bridge
interface to your VM.


✅ **Done 👍 & Enjoy**❗️

.


