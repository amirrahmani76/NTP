# Intro

NTP is a protocol used to synchronize clocks over a network. It works by exchanging time information between devices and adjusting 
their clocks accordingly. NTP can synchronize clocks to within a few milliseconds of each other and can be used to synchronize 
devices across a LAN or WAN.

# NTP Configuration on Windows Server as Master Server Clock:

open your PowerShell and use these commands:

#### Step 1: Enable NTP Server

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\NtpServer" -Name "Enabled" -Value 1
```

#### Step 2: Make the Announce Flags 5

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\services\W32Time\Config" -Name "AnnounceFlags" -Value 5
```

#### Step 3: Restart NTP Server

```powershell
Restart-Service w32Time
```

### Step 4: Allow NTP Port (123) on Firewall

```powershell
New-NetFirewallRule `
-Name "NTP Server Port" `
-DisplayName "NTP Server Port" `
-Description 'Allow NTP Server Port' `
-Profile Any `
-Direction Inbound `
-Action Allow `
-Protocol UDP `
-Program Any `
-LocalAddress Any `
-LocalPort 123
```

# Configure the ESXi host to synchronize time with the Windows Server:

1. Connect to the ESXi host using the vSphere Client or vSphere Web Client.
2. Select the ESXi host from the inventory.
3. Go to the "Configure" tab.
4. Under "System," click on "Time Configuration."
5. In the "Time Configuration" window, click on the "Edit" button.
6. Select the checkbox for "Enable NTP Client."
7. Enter the IP address or hostname of the Windows Server acting as the NTP server.
8. Click "OK" to save the changes.

# Update Time Zone for Linux

```bash
tar -xzvf tzdata2022b.tar.gz
```

Compile to apply the updates for the region(s) of interest (Asia) the system's time zone data through the 
'zic' (timezone compiler) command (you should be root/root-privileged user):

```bash
zic asia
```

Note: the changes will be applied on the Asia/Tehran directly as well as other regions in Asia.
Relink the localtime /etc/localtime with the corrected timeZone information using the following command:

```bash
find /home/ -maxdepth 2 -type f -exec grep -wH 'TZ' {} +
```

# Configure the ntpd service to start on every boot in CentOS 6:

```bash
Configure the ntpd service to start on every boot in CentOS 6:
```

# Configure the ntpd service to start on every boot in Ubuntu:

```bash
sudo systemctl is-enabled ntp
```

# Install NTP on CentOS with usb drive:

#### See usb in disk list:

```bash
lsblk
```

#### Mount USB:

```bash
sudo mkdir /mnt/usb
sudo mount /dev/sdX1 /mnt/usb
```

Replace /dev/sdX1 with the appropriate device path for your USB drive. You can find the correct device path by running the lsblk 
command before and after inserting the USB drive.

#### Extract ntp.tar.gz

```bash
sudo tar -zxvf ntp.tar.gz
cd ntp
```

#### Make install:

```bash
sudo ./configure
sudo make
sudo make install
ntpq –version
```

#### Edit ntp config file:

```bash
sudo nano /etc/ntp.conf
```

```tex
driftfile /var/lib/ntp/ntp.driftfile
# Specify one or more NTP servers.
server <NTP-Server-IP>
```

#### Restart ntpd service:

```bash
sudo service ntpd restart
```

#### Check client connection to ntp server:

```bash
ntpq -p
```

#### Unmount USB:

```bash
sudo umount /mnt/usb
```

# Install NTP on Ubuntu Client with USB Drive:

#### Install NTP on Ubuntu Client with USB Drive:

```bash
sudo apt install ./ntp.deb
```

#### Edit ntp config file:

```bash
sudo nano /etc/ntp.conf
```

```tex
driftfile /var/lib/ntp/ntp.driftfile
# Specify one or more NTP servers.
server <NTP-Server-IP>
```

#### Restart ntp service:

```bash
sudo service ntp restart
```

#### Check client connection to ntp server:

```bash
ntpq -p
```

```tex
remote         refid           st  t  when  poll  reach  delay  pffset  jitter
===============================================================================
192.168.67.129 51.137.137.111  4   u   63   64    377    1.496  -1.164  3.200
```

Here are some details from the output:
• remote: IP address of the NTP server.
• refid: Reference ID of the NTP server.
• st: Stratum level of the NTP server.
• t: Type of the NTP server (in this case, u for unicast client).
• when: Time since the last response from the NTP server.
• poll: Polling interval to contact the NTP server.
• reach: A bitmask indicating the success of recent communications with the NTP server.
• delay: Round-trip delay to the NTP server.
• offset: Difference in time between the system clock and the NTP server.
• jitter: Variability in the time measurements.

#### Check ntp server date in client:

```bash
sudo ntpdate <Server-IP>
```

# Configure NTP on Windows Client:

1. Open the Control Panel and navigate to the "Date and Time" settings.
2. Select the "Internet Time" tab and click on the "Change settings" button.
3. Check the box for "Synchronize with an Internet time server" and enter the address of your master clock's NTP server.
4. Click "Update now" to synchronize the time with the master clock.
5. Click "OK" to save the changes.

# Install and configure NTP Server on the host computer

#### Step 1: Update the package repository index

```bash
sudo apt update
```

#### Step 2: Install NTP Server with apt-get

```bash
sudo apt install ntp
```

#### Step 3: Switch to an NTP server pool closest to your location

When you install the NTP server, it is mostly configured to fetch the proper time. However, you can switch the server pool to the ones closest to your location. This includes making some changes in the /etc/*ntp*.conf file.

Open the file in the nano editor as sudo by running the following command:

```bash
sudo nano /etc/ntp.conf
```

```tex
pool 0.asia.pool.ntp.org iburst
pool 1.asia.pool.ntp.org iburst
pool 2.asia.pool.ntp.org iburst
pool 3.asia.pool.ntp.org iburst

# Use Ubuntu's ntp server as a fallback.
pool ntp.ubuntu.com
```

In this file, you will be able to see a pool list. We have highlighted this list in the above image. The task here is to replace this pool list by a pool of time servers closest to your location. The pol.ntp.org project provides reliable NTP service from a big cluster of time servers. To choose a pool list according to your location, visit the following page:

[NTPPoolServers &lt; Servers &lt; Network Time Foundation's NTP Support Wiki](https://support.ntp.org/bin/view/Servers/NTPPoolServers)

Quit the file by hitting Ctrl+X and entering y to save changes.

#### Step 4: Restart the NTP server

For the above changes to take effect, you need to restart the NTP server. Run the following command as sudo to do so:

```bash
sudo service ntp restart
```

#### Step 5: Verify that the NTP Server is running

Now, check the status of the NTP service through the following command:

```bash
sudo service ntp status
```

Make sure your service is running.

#### Step 6: Configure Firewall so that client(s) can access the NTP server

> If your firewall is not active, you don't need to go through this step.

Finally, it is time to configure your system’s UFW firewall so incoming connections can access the NTP server at UDP Port number 123.

Run the following command as sudo to open port 123 for incoming traffic:

```bash
sudo ufw allow from any to any port 123 proto udp
```
