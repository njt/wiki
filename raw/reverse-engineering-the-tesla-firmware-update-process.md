---
url: https://www.pentestpartners.com/security-blog/reverse-engineering-the-tesla-firmware-update-process/
date_fetched: 2026-09-13
---

### TL;DR

How does the Tesla Model S update its firmware? What did we find when reverse engineering the display and instrument cluster?

Here’s the result of a couple of weeks work, working on a real vehicle that (mostly) worked after we had finished.

Part 1: analysing the hardware, complete with a 14 layer PCB in the CID

Part 2: reversing the firmware update process & the importance of Suicide Bomber mode

We’re publishing now as the process has recently changed with the Model 3

### CID VCM Software Architecture

The CID is much bigger and more complicated than the majority of embedded systems and – unusually for an embedded system – its operating system is the full version of Ubuntu – distributed by Linux.

In the kernel, we found custom elements such as Harman Redbend, The kernel was built with the Linaro toolchain.

In order to fulfil the legal requirements of using open-source software, Tesla revealed the source of the kernel half way through the project. However, this does not help a great deal with reverse engineering as it only represents a small part of the system.

### Bootloading the VCM

This seems similar to many Tegra devices.

The BPMP (Boot and Power Management Processor) is a secondary processor of the Tegra SoC. It executes a read-only bootloader stored in ROM and is an ARM7 processor. We powered down the main processor in the Tegra at this point.

A significant amount of information is provided when the BPMP ROM bootloader reads a piece of data from the NOR flash. This is called the BCT (Boot Configuration Table). Information includes:

- Addresses of several bootloaders in memory
- Where to load the bootloader into SDRAM
- Entry point for the bootloader in SDRAM
- Configuration of the SDRAM connected to the system (needs to be accessed in order to boot)

The BPMP begins execution after copying the first stage bootloader into SDRAM. We had not switched on the main processor in the Tegra at this point.

The first stage bootloader appears to be called QUICKBOOT and its size is approximately 56KiBytes. Clear references are made to AES-CMAC – a Message Authentication Code. “AES-CMAC Xor” are among the strings used here which are also found in U-boot, the open-source bootloader (https://github.com/u-boot/u-boot/commit/b149c4c399b111cec1ff7505ca9fabbeeb4fe394).

**Strings from the first stage bootloader**

One might assume that the bootloader is based on U-boot, but the specific crypto functions were in fact written by Nvidia, and could therefore be freely used elsewhere by Nvidia.

The main processor powers up when the first stage bootloader sets up the SoC correctly. The second stage bootloader is then loaded, to allow the main processor to run.

When we unpacked and ran the kernel, we found that the second-stage bootloader was very simple. We did not see any signs of cryptographic protections. It does not seem to be based on any widely known bootloaders but rather we think it is fully custom.

Loosely following the Android bootimg format, the kernel image is made up of a kernel and ramdisk in one file.

A script called /sbin/init-stage-0.sh runs once the kernel is activated. Its primary purpose is to render the system fully operative by mounting the NAND flash partitions.

### NOR flash

We found the BCT, first and second stage bootloaders, and kernel in the NOR flash .

**Layout of the NOR flash on CID**

We found that numerous partitions were mirrored into primary and recovery partitions.

### NAND flash

Stored as sqaushfs filesystem, the /usr/ filesystem contains binaries and scripts that do not change during normal operation. Common in low-resource embedded systems, SquashFS is a compressed, read-only file system that allows unchangeable parts of a filesystem to occupy a smaller amount of the flash memory.

Containing logs, the /var/ directory is a small 128Mbyte ext4 filesystem.

Containing upgrade files and transient data, the /home/ directory is stored in the remaining flash as an ext4 filesystem.

Overlayed onto the read-only file system, the /home and /var directories only allow a select few directories to be read/write, which means the bulk of the system is read-only.

### Dual-bank firmware

After the initial ROM bootloader has run, the bootloading process has the choice of two images:

- ROM bootloader – only a single copy of this is available. It is immutable and unlikely to fail unless there are catastrophic hardware issues.
- stage1_primary/stage1_recovery – chosen sequentially by the ROM bootloader. The second one runs if the first fails. The two bootloaders look almost identical except for the location in memory
- stage2_primary/stage2_recovery – the bootloader that runs here is determined by which of the stage1 bootloaders ran. Again, the two bootloaders look almost identical except for the location in memory
- kernel_a/kernel_b – these two kernels are very different from one another, in terms of their boot parameters
- online/offline usr partitions can be mounted when the system starts

Here are the boot parameters of the two kernels:

The init-stage-0.sh script reads the value of thispartid, and loads one of two /usr/ squashfs partitions. If one is damaged, the other one can be used to recover.

We could also see that AppArmor is normally enabled. This Mandatory Access Control system allows the kernel to confine binaries to restricted sets of resources. Ubuntu enables this by default. The configuration on the Tesla looks to be standard.

### Secure Boot

Secure boot is supported by the Tegra, however the documentation and code examples around it look fragmented. In developer forums, there was some confusion about how to implement it. This is common with higher-end processors.

We could not conclusively validate to what extent secure boot was used, but we did discover several things.

The signature of the BCT is verified by the ROM bootloader via an internally stored key. This is called the SBK. An AES-CMAC is taken of part of the BCT and verified. This is symmetric cryptography – if the key was discovered, a valid BCT could be generated.

We were unable to ascertain whether the SBK is unique per device/vehicle, or common to larger groups. We would need access to multiple vehicles to determine this.

Alternatively, storing only a public key internally, the Tegra can use public key cryptography. Even if the public key were discovered, it would still not be possible to generate a correctly signed BCT image. This is a much stronger alternative to the symmetric keys. By our reckoning, software support was not in place before 2015 – after the Tesla CID was deployed.

The first stage bootloader performs an AES-CMAC of the second stage bootloader. We could not ascertain which key was used here, but Tegra documentation indicates the SBK could also be used.

Only a CRC of the kernel is performed by the second stage bootloader. An attacker could potentially load a malicious kernel onto the device at this stage, if the chain of trust is broken.

### Userspace Software

The CID enters userspace once it has booted.

A series of Qt binaries renders the user-interface of the CID. We performed only very limited reverse engineering of these. An 8 character PIN protects a service menu on the CID, which seems to change daily and is implemented by the CID.

Shell scripts implement a larger amount of the functionality in the vehicle. Reverse engineering is made easy since these are human-readable text files.

### Firmware update mechanisms

We observed multiple firmware update mechanisms.

A VPN connection, outbound from the vehicle to Tesla’s systems, was central to all the mechanisms. We did not observe any other form of transport encryption; all requests from the CID were made with unencrypted HTTP.

We saw the following update mechanisms:

- Shell script – looks like a legacy system, but has much in common with the later updater binary mechanism.
- Update binary – a large multi-call binary polls for updates, downloads them, and can apply them to the system.
- Kernel/bootloader – the kernel and bootloader are updated by a specific binary.
- Map – the VPN connection is used to download mapping data
- Conventional ECU – to distribute firmware updates from the CID to the ECUs via the CAN gateway, a complex process is used.
- USB firmware for the Wi-Fi module and cellular modem.

We consider each of these below.

### VPN connection

An OpenVPN connection out to a Tesla server is established by the CID. Per-vehicle keys and certificates are used to perform this. The VIN of the vehicle is the subject of the certificate.

**Example VPN key**

We were able to extract these keys from the filesystem with local access to the CID, and then connect into the Tesla private network by using them on another machine. The keys expired on the 31st May 2018. There did not appear to be a fallback mechanism if they had not since been renewed by a firmware update.

With the VPN securely configured, it is not possible to intercept communications or tamper with them.

VPN can be established via Wi-Fi or cellular – whichever is available. Certain aspects of communication (for example, large downloads) showed affinity for Wi-Fi. This is presumably to reduce costs to Tesla.

When connecting to the VPN, a number of routes were established automatically:

- 32.0.0/16
- 33.0.0/16
- 224.0.0/24
- 232.75.0/24
- 232.79.0/24

Full port scans of these ranges were carried out, under the terms of the Tesla bug bounty, and only uncovered a handful of hosts:

- vn.teslamotors.com – vehicle data and status, security token update for access to diagnostic and IC SSH
- vn.teslamotors.com – firmware downloads and updates
- Firmware-bundles.vn.teslamotors.com – a defunct firmware update server
- vn.teslamotors.com – map data and updates

VPN keys were stored on the large SD card that is connected to the gateway, according to earlier reports on Tesla security. By removing the SD card, these could be trivially recovered. This was not the case on the vehicle under test; here they were stored on the NAND flash on the VCM in the CID.

**The firmware server**

**Example JSON response from server**

### Shell Script Firmware Updates

We found a legacy firmware update mechanism on the Tesla filesystem, employing a series of shell scripts to perform updates. It provided a lot of useful information, even though it was no longer operational (no valid download URL was ever returned).

### Handshake firmware download

Over the Tesla VPN, the shell script /local/bin/do-firmware-handshake obtains and installs firmware updates.

This script can be started via several sources:

- Using the upstart event manager periodically
- (Although not seen in operation) on response to an upstart event named “firmware-handshake”
- Manually (although not observed, it is noted in comments in the file)

The script obtains and applies updates from a high-level, as follows.

Using a sentinel file, the script checks to see if an existing upgrade is taking place.

The shell script, that unpacks the upgrade package and installs it, checks to see if the “unpack.sh” script is running. Do-firmware-handshake exits if an upgrade is ongoing, since an update is already in progress.

One can assume that the gateway is processing the upgrade and do-hardware-handshake exits if the sentinel file is younger than 20 minutes. Therefore no active check against the gateway is performed to see if it is occupied. It is assumed that the gateway update has hung if it is older than 20 minutes, and the handshake process continues.

To see if the VPN is connected, the script checks that the interface tun0 is present. The script waits 30s to try again if the VPN is not connected. No active attempt is made to establish a VPN tunnel – another external process does this.

The script makes a connection to the following URL once the VPN connection is established:

http://firmware.vn.teslamotors.com:4567/vehicles/<VIN>/handshake

The following data are sent:

vehicle_hardware_configuration_string – derived from the “Hardware IDs”, this is a comma separated string describing the vehicle’s configuration, stored by the gateway. Another shell script, /usr/local/bin/vehicle_hardware_configuration_string, generates this string. It’s format:

– read from /etc/swver, this is the software version running on the CID.

Then the server returns a JSON string, expected to contain the following fields:

- Firmware_download_url – the location of the file we will be downloading
- Firmware_download_file_md5 – the MD5 checksum of the file we will be downloading
- Download_status_url – a URL to post back the status of the upgrade
- Vehicle_job_status_url
- Unpack_size – size of the unpacked firmware file
- Install_size – size required to install the firmware file

We found that it was still possible to make these requests and receive responses. Crucially, we found it was possible to make requests for any VIN using the VPN for another car.

We were surprised to find that, rather than a robust solution like jq, the JSON response is manually parsed using awk, gsub and split.

Some basic sanity checking of these fields is then performed.

To indicate that an upgrade has commenced, the download_status_url is stored in the sentinel file.

By making a HTTP HEAD request for the file, the size of the firmware_download_url file is then checked. We thought this an odd mechanism to use, since it could simply be transmitted in the JSON response. The firmware_download_url pointed to the server firmware-bundles.vn.teslamotors.com in all responses obtained. Despite being issued via the handshake process, none of the links worked.

Using the unpack_size and install_size from the JSON response, basic checks are performed for spare space on the flash filesystem.

To prevent the CID from sleeping for 60 minutes, a request is made to the web API running on the CID on port 4035.

Now to download the file. It is downloaded as one part – not chunked or split in any way. If it is not the expected size, the download is attempted again. There is no way of escaping this condition, meaning that the script could end up in an infinite loop.

These files would only be downloaded only over the Tesla VPN for the example firmware_download_url obtained during testing.

The md5 checksum of the file is then checked against the expected value. Two further attempts are made to download the file if it does not match.

The user is prompted to upgrade via a pop-up on the CID screen, once the file is downloaded. Another shell script does this, /usr/local/bin/get-response. It makes a request to the web API running on the CID on port 4070, and waits an hour for a response.

The downloaded file is moved to /home/tesla/dropbox, ready to be installed, if the user accepts the update.

### USB update

Using a USB memory stick, it is also possible to place update files into /home/tesla/dropbox. Shell script /usr/local/bin/usb-upgrade performs this.

The script looks for files in the folder /toinstall, with the filename format:

ui_full_package_*_<code>_pedigree_v*.tar.gz

Using the following table, the <code> is looked up from the VIN number of the vehicle:

**Certain VINs are handled differently via USB update**

This table indicates there are development vehicles with pre-set VINS.

The file is copied across to the dropbox and then installed, if the current version of the software on the CID is not present in this filename. Since the check is naïve and ignores the actual version, this would allow both upgrade and downgrade of firmware to occur.

It seems that there is a bug in the script. The <code> returned is simply blank if the VIN of the vehicle is not present in the script. This causes the following form of files to match:

ui_full_package_*__pedigree_v*.tar.gz

In a given folder on a USB stick, it would appear that an attacker could place a crafted firmware update file and execute arbitrary code. We tried this many times, but could not trigger it. Another security control was preventing the script from being called, but we could not determine what it was.

### Dropbox install

The script “unpack.sh”, located in /local/bin, installs packages that are placed in the /home/tesla/dropbox folder. This script describes the process and is heavily commented and.

Although it is custom, the package format is very simple and would be easy for an attacker to recreate.

The process, from a high-level, is as follows:

Check that a filename has been provided, and the file exists.

Once the car is “parked”, another shell script performs this check – /usr/local/bin/car-is-parked. On port 4035 of the CID, this makes a HTTP request to a web API to check that the car is in gear “park”. Of note, car-is-parked script contains functions to check the speed is zero, but these are not called.

The script waits for 5 minutes and checks again if the car is not parked.

To prevent the CID from sleeping for 20 minutes, a request is then made to the web API on port 4035.

Using the utility mktemp, a temporary directory is created in /home/tesla/unpack.tmp-XXXXX.

The tar file is unpacked into the temporary directory.

The existence of 4 files is checked:

- NAME of the package
- VERSION number of the package
- md5sums of files in the package
- tar.gz – files associated with the package

The process does not check to see if it exists, but it expects a file “install.sh” to be present.

To check the integrity of the files, the file md5sums is fed to the tool md5sum. This adds no security and is purely an integrity check.

With the name, version and data.tar.gz passed as parameters, the file install.sh is executed. For ultimate control over what the packages do, install.sh can perform any actions that the root user normally can.

The install.sh file can contain arbitrary commands and the entire process runs as root. In order to take control, an attacker can place a valid upgrade package in dropbox to carry out arbitrary actions on the CID or IC.

We have seen comments that indicate that it is known that this process is not robust and has issues:

Far from being secure, this mechanism has few protections outside of the transport encryption provided by the VPN. This is probably the primary reason why it is deprecated.

We do not know why it is still present on the system. Developers may be concerned that removing of one of the numerous scripts could cause unintended consequences.

### Updater updates

We think this is the normal mechanism for updating firmware. It is a large, monolithic binary containing a huge amount of functionality. Fortunately, reverse engineering is accelerated by it being statically linked (i.e., all code is compiled into it) and a debug build (i.e., it contains strings and function names that would normally be stripped).

**Statically linked with debug information**

Activated by changing the name of the binary, the updater can take several different “personalities”. Each one can open a command port and a HTTP port, which listen on all interfaces of the device.

- ic-updater – updater for the IC. Opens port 28493 for commands, and 21576 as a web server.
- cid-updater – updater for the CID. Opens port 25956 for commands, and 20564 as a web server.
- gwxfer – used to transfer files to the gateway, a binary replacement for the gwxfer shell script,
- sm-updater – it is unclear what “sm” refers to. This could be something used at the factory, since the rest of the binary has reference to “sitemaster”.
- ethdeploy – appears to be a means to deploy packages to other devices within the vehicle
- upackager – related to the gateway, taking parameters for release.tgz (update for ECUs) and internal.dat (configuration of the vehicle) and vhcs (vehicle configuration string)

**Different names for different personalities**

Depending on which name it is started with, the binary behaves differently; this includes the services it performs, paths it uses, and techniques. The code references the variable where the personality is stored over 330 times.

To make it easy to determine when the code deviates based on personality, 0 corresponds to the IC, 1 to the CID, and 5 to the SM.

**Ports for the command/http for different personalities**

Running the updater as a “cluster”, the system is intended to work with multiple devices (the CID and IC). Connecting out and downloading firmware over the VPN, the CID acts as a master. Termed “relay” in the code, the command/HTTP ports are then used to distribute the firmware to the IC.

The updater performs certain tasks on startup, such as examining the current system, connecting outbound to the firmware server (known as a “handshake”), and starting the command and web servers.

**Example of conditional flows based on personality**

Although many errors are seen, it is possible to run the binary in the QEMU emulator. The binary starts two listening services, as expected.

**Running IC-updater in an emulator**

The binary performs a sha512 hash of itself during initialisation. We were surprised to find that, although sha512 outputs 64 bytes, only the first 8 bytes are retained. This means that it possible to find a hash collision using brute force.


**hash_self performs a sha512 hash of the binary**

**Truncated sha512 hash output**

**Normal sha512sum of file**

Later, when making requests to the firmware update server, this hash is used as part of the User-Agent string.

**Hash being put into the User-Agent part of the request**

An attacker could report the expected sha512 hash whilst running alternative malicious software, although this means that the server can determine which version of the updater is in-use. It is not a strong protection against malicious action.

Carried out by a timer or on-demand, the updater can process a series of commands in a buffer. They can be inputted by cid-updater or the opened command port, allowing other devices on the Tesla ethernet network to carry out actions.

The following is an example of the task “fwheartbeat” being started with a 1hr interval with a call to the function start_regular_timer:

**fwhearbeat added on a timer**

By calling the function do_after_e_ms, it is also possible to run a command after a fixed period. Here is an example of a firmware download being restarted:

**Download restart being run after a time has elapsed**

Referenced by strings, these commands can take many values, stored in a large array.

**Array of commands**

With different parameters, commands “install” and “patch” both end up calling the function do_install.

Like an interactive command processor, the command port of the process provides usage tips and other help.

**Example of the ic-updater being asked to serve a file over HTTP**

**Downloading the served file over HTTP**

**Current status reported over command port**

A session token is required to secure the commands between the IC and CID. This changes once daily and is synchronised from the Tesla servers over the VPN. An attacker can intercept this token if they can sniff the connection between the IC and CID, and use it to send their own commands.

We were able to put the binary into a development mode. When we had done this, most of the security functionality, including signature checking, was disabled.

**Development mode causing signature check to be skipped**

The binary contains a vast amount of functionality. The most important processes, from a high level, are as follows:

- Handshake –the process of sending a request to a Tesla server containing vehicle details and receiving a response to carry out actions on the vehicle.
- Downloading and decrypting various firmware update files based on the handshake
- Installing downloaded and decrypted firmware update files, either termed patches (full firmware updates), bsdiff40 (binary difs) or Redbend deltas (proprietary binary diffs) into the offline flash partitions
- Relay – copying firmware from the CID to IC
- Redeploy – copying offline aspects of the firmware to online, for recovery Other functionality is contained within the binary, but is not used.

### Handshake

Similar to the shell script update, data about the vehicle is keyed using the VIN and sent to a remote server which responds with any available update.

A POST request to the firmware server is made by the function do_handshake. To connect from a file, this takes the configuration of the firmware server (including server name, port and path).

**Config for the handshake request**

More complex than in the shell script version, the data contained within the POST request include the following:

- Vehicle VIN – retrieved and cached from the gateway rather than read from a file. Sent in the path of the request.
- VHCS (vehicle hardware configuration string) – a string based on hwids.txt recovered from the gateway (generated using function fetch_vhcs). Sent as a POST parameter.
- Current firmware signature – read using the function read_firmware_signatures. Reads the firmware signature from the last 40 bytes of the memory partition holding the /usr/ partition. Sent base64 encoded as a POST parameter.

**Format string for the POST request**

The function request_HTTP makes the POST request. It can only make HTTP requests, not HTTPS ones. The updater binary has no TLS functionality at all, meaning handshakes are always carried out in the plain, relying on the security of the Tesla VPN.

The function tun0_is_up checks the VPN connection. This seems naïve, simply checking for the existence of the device tun0. An attacker with control of the CID could establish their own VPN without the updater binary being aware.

The updater binary itself does not appear to have any functionality to establish the VPN connection. Although it could not be determined which one, this must be carried out by an external process.

The server gives a JSON response to the HTTP POST request. Generally, no response is given if the request is malformed or incorrect.

**A typical handshake response**

The function handle_handshake_download handles the JSON response.

In the handshake response, tens of fields can be present. Individual fields can be pulled from the JSON copy_handshake_var_at, which takes a parameter of the field name. At least 35 different field names are used in 72 references to this function.

**Multiple calls to find fields in the JSON response**

This indicates the huge complexity of the firmware update mechanism in the Tesla. Whilst the vehicle was under test, very few of these fields were seen to be used.

Here is an example of the md5 hash being read from the handshake response in the function do_install.

**Reading the field firmware_download_file_md5 from the response**

The function handshake_is_actionable is then called. The handshake will be prevented from taking action if there is an upgrade being downloaded, already staged on the CID, being relayed to the IC, or being installed.

**Reasons for not actioning a handshake**

The function initiate_handshake_install is called if the handshake can be actioned. The handshake response is stored in a file for later processing, and a command is added to the queue.

### Download

Three different fields can be contained in the handshake response for files to download:

- Firmware_download_url – a conventional update, as per the shell script method
- Bsdiff40_download_url – binary diffs using an open-source solution
- Rbdlt_download_url – proprietary Redbend deltas

Using HTTP, they all appear to be downloaded in a similar fashion. Here’s an example URL:

We noted several interesting characteristics of these URLs:

- Generated in response to each handshake request and apparently unique
- Always have an expiry date of two weeks after the handshake response is received.
- A HMAC (hashed message authentication code) appears to check the integrity of the rest of the request.
- Server they are downloaded from is available from the public Internet –no requirement for VPN connection to be established.

Due to the expiry and HMAC, we were unable to guess or brute-force other firmware download links.

Of note, there is risk of interception and tampering due to the download being performed using HTTP (no encryption) over the public Internet.

The field firmware_download_file_md5 checks the integrity of the download. While the hash is downloaded over the secured VPN, the file is downloaded over the public Internet. It is likely that the MD5 hash would no longer match if an attacker tampered with the download.

Wi-Fi or cellular connection can be used to perform the download. Another field, wifi_wait_until, will allow the download to happen over Wi-Fi for a limited period of time. We believe this is to avoid costs of downloads over cellular connections, while permitting them for essential updates.

There are fields concerning cryptography, as can be seen in the firmware handshake response:

**The crypto key for a firmware download over open Internet**

Salsa20 – a light weight algorithm – can be used to encrypt downloaded update files. The function decrypt_and_save_file implements the decryption.

The handshake response sends the entire 256bit key over the VPN, so should remain secure, preventing the firmware downloads over the Internet being intercepted.

All the downloaded files we saw were encrypted. It should be possible to send unencrypted files, but we did not see any evidence of this.

### Install

Many different aspects of the system can be updated:

- /usr partition
- Kernel
- bootloader

The /usr partition updates concerns the bulk of the functionality in the updater binary. The system is divided up into an online and offline usr partition for this. At every point they are dealt with as raw memory devices (e.g., /dev/mmcblock0p1), although they are referred to as usr, not as mounted filesystems. “usr” refers to read-only aspects of the memory devices.

Generally speaking, it appears that most firmware updates are applied to the offline partition. Then changes made in the offline partition can either be copied back to the online partition, or run directly from the offline partition.

The following occurs in order to execute changes from the offline partition,:

- Changes made to the offline partitions (using patch, bsdiff, or Redbend)
- Signatures of the online and offline partitions are checked to make sure patches are applied correctly.
- Offline partition is mounted as /newusr; the contents of /newusr/deploy/ can then be executed

The following occurs to copy changes from offline to online:

- Changes are made to the offline partitions (using patch, bsdiff, or Redbend)
- Signatures of the online and offline partitions are checked to make sure patches are applied correctly.
- Update then “redeploys” firmware, which involves copying bootloader, kernel, and usr partitions across to the online partitions.

Applying changes to an offline partition means the system can keep on running while firmware is updated. This appears to go against the concept of a recovery partition – these are overwritten first. It should be difficult for the car to become bricked thanks to the use of multiple checksums and signatures.

The updater binary carries out updates to the kernel and bootloader by making system calls to the sample update program mentioned above, as seen from the parameters passed.

**Choosing cid/ic/sm-udpate_sample**

**Making calls to *-update_sample**

The method by which changes are applied can vary.

### Conventional full update

tar.gz files are downloaded, unpacked, and a shell script run to action the changes, almost identical to the shell script firmware update above.

### Bsdiff firmware update

Downloaded over HTTP, decrypted and then installed, some firmware updates are bsdiff binary diff files.

The function patch_from_bsdiff40_to_offline_dev applies updates.

**Overview of patch_from_bsdiff40_to_offline_dev function**

/this carries out the following, from a high level:

- Check the downloaded file is in BSDIFF40 format.
- Start processing the BSDIFF40 file using statically linked bzip functions
- Determine which of the flash memory banks is the offline usr (unused) bank.
- Apply the binary differences across the entire offline usr partition

Operating at the raw flash level, the process is entirely unaware of the filesystem or files contained within it. Prior to the update being applied, it requires that the /usr partition is unmodified. The entire partition is read-only and is not problematic. To ensure the starting point is as expected, a signature check is performed before the update is applied.

### Redbend firmware update

The Tesla CID can use Redbend to perform updates, as an alternative to bsdiff. This takes and applies update files (“deltas”) containing differences between the current and new firmware. While it has also been seen in Android mobile phones, marketing material for Redbend suggest it is specifically targeted towards automotive.

It appears, from strings in the updater, that the technology used is called vRM or vRapid Mobile. There is no detailed technical information, but this is mentioned in some locations on the Internet.

The UPI or Update Installer is the software running inside the cid-update. UPG or Update Generator is another piece of software that creates the updates that are deployed.

There is no functionality to download updates in the Redbend portion of the updater. This is handled by Tesla code only.

**Large number of RB_ prefixed functions for Redbend**

DP files (Delta Package), which can contain multiple individual updates, are processed by the UPI. These files have a CRC32 checksum, called “signature” in the Redbend software. This is an incorrect term; a CRC is only useful for non-malicious integrity protection, not malicious manipulation.

**The version of Redbend vRM in use**

Very little practical difference was seen between Redbend and bsdiff40 changes applied to the system.

### Relay

Designed to act as a master, the CID makes requests out to the handshake server, downloads updates, applies them, and “relays” them to the IC.

**ic-updater will refuse to perform a handshake**

We believe there are several different methods to relay, including sending the entire offline /usr partition, and serving individual files on the HTTP server on the CID, and requesting that the IC downloads them.

Prior to the IC, the CID updates firmware. During our testing, the update to the CID did not proceed (we do not know why). The process of relaying the updates to the IC was not seen in action, so we struggled to determine exactly how it worked.

### Redeploy

The updater will “redeploy” the firmware once changes have been applied to the offline flash partitions. Before it reboots the device, this seems to copy the bootloader and kernel (using the cid-update_sample binary) and the /usr partitions across to the online partitions.

As with the relaying, we did not observe this during testing, since neither the CID or IC would accept an update.

### Kernel/bootloader updates

A utility called cid-update_sample can be used to update the bootloader and kernel of the device. We found the naming of this file curious, however it is referred to in several other locations.

This does not have multiple personalities for the CID and IC, unlike the main updater binary – although it has different names, the same file is present on both devices.

**Running the cid-update_sample binary in an emulator**

The cid-update_sample binary is called from two places:

- A shell script called /usr/local/bin/qber updates the BCT and primary stage 1 and primary stage 2 bootloaders. This script did not appear to be called from anywhere else, but it could be called from inside a downloaded firmware update.
- From inside cid-update or ic-updater

Interestingly, this binary can perform partial updates of the BCT. We noted this in the help “only SDRAM & Device timing will be updated”.

The SBK – the AES key stored internally on the Tegra SoC – signs most of the data in the BCT. Signed data include the SDRAM and device timing. The file must be signed using the SBK to alter the SDRAM and device timing.

We found an example BCT file in the /usr/deploy folder. It did not contain any signature or bootloader data – only the SDRAM and device timing.

We concluded that the cid-update_sample must retrieve the current BCT, merge the new SDRAM and device timings, and re-sign the data. This means the binary must either be able to access the SBK inside the SoC, or contain a copy of it. We would need to do further reverse engineering to confirm this.

**BCT file in update does not contain all signed data**

### Map updates

Mapping data is stored by the CID on an external microSD card. The VPN connection can be used to periodically update it.

the scripts /usr/local/bin/nav-sync-and-apply-map-patch.sh and nav-apply-map-patch.sh carry out the mechanism by which these are updated.

This performs the following tasks, from a very high-level:

- Ensures car is awake and does not sleep for 15 minutes
- Uses rsync to download updates from a URL (rsync://filesync.vn.teslamotors.com/mapdata/patches). Rsync is a file synchronisation tool that can be used to efficiently transmit changes to large datasets across slow or unreliable network links.
- The updates are tar files which are unpacked onto the SD card.

We saw no signs of integrity protection or authenticity checks.

There is no access control or authentication to access the data – in all regions, anyone with valid VPN keys can download the entire mapping dataset.

The map update remains as shell scripts, despite the other firmware update mechanisms moving from shell scripts to compiled binaries.

### Conventional ECU update via gateway

Integrated into the CID, the Tesla gateway is a similar architecture to many IVIs, with a higher power system running the media/UI components, coupled to a lower powered gateway, which in turn is connected to a CAN bus. The IVI is prevented from injecting onto the CAN bus by this gateway.

By keeping the gateway limited in functionality, it is possible to limit the attack surface and complexity of the code. Both are likely to improve security.

Generally speaking, most vehicles tend to have additional CAN gateways separating the CAN buses from each other, for example the drive train from the body systems. The gateway in the Tesla CID is the only CAN gateway found in the vehicle and also performs this function.

With 2MiByte of integrated flash, the gateway is a MPC5668G microcontroller. Specifically marketed for automotive Gateway applications, it employs an e200z6 core which is PowerPC architecture.

We were surprised to find that full datasheets and a reference manual are available for this device. This is unusual for automotive specific microcontrollers.

The Ethernet switch connects the gateway to the CID VCM, IC, and diagnostics port. The entities communicate using various UDP and TCP protocols and are all assigned IP addresses.

Connected to external CAN transceivers are multiple CAN interfaces on the gateway. There a LIN interface, too.

On the mainboard of the CID, the gateway is directly connected to the full-size SD card.

On examining the firmware upgrade we had obtained, we found the gateway runs a customised version of FreeRTOS. A small and simple real-time operating system, it includes functionality such as threads/tasks, mutexes, semaphores,etc. commonly found in automotive applications.

Files transferred over UDP port 3500 are accepted by the firmware running on the gateway. These transfers are performed either by the Perl script /usr/local/bin/gwxfer or the updater binary on the CID. We found no other use of Perl on the system.

**Perl gwxfer script**

This service has no authentication. The SD card stores any files sent to the gateway.

Files routinely transferred:

- dat – the configuration of the vehicle, where certain options are enabled or disabled. Sent both to, and received from, the gateway.
- tgz – a compressed file containing updates for the CAN-connected ECUs.
- img – an alternative firmware transferred to the gateway to perform ECU updates.

**Part of internal.dat**

When trying to enable/disable certain features on the vehicle, we were unable to modify the internal.dat file. It seems likely that the gateway makes requests to the individual ECUs to build up the car’s capabilities.

UDP port 1050 accepts certain commands, such as sending/receiving CAN messages, triggering an update, etc, and is also open.

ECU update process from a high level:

Employing one of the other firmware update mechanisms previously mentioned, the CID downloads an update package.

Two files are required to update the ECUs. One is an archive containing updates for all the individual ECUS called “Release.tgz”, and the other is Noboot.img, which is specific firmware for the gateway to update other ECUs.

Multiple mechanisms build a release.tgz file.

A “premastered” release is the first. With no pre-processing, this update is downloaded from firmware servers and applied directly to the ECUs.

The second uses “seeding”. Individual ECU update files are downloaded by the firmware update mechanism into a “seed” directory, then a custom release.tgz is created based on the ECUs which need updating. We think the aim of this may be to reduce how long the gateway spends in “update” mode.

Using the script/binary gwxfer, files are transferred to the gateway and stored directly onto the SD card.

**Contents of SD card during upgrade**

A public key that is stored in the normal gateway firmware checks the signature of noboot.img.

noboot.img is renamed boot.img when a command is sent to the gateway. boot.img is copied into RAM at 0x40000000 and executed when the gateway is restarted.

(We found that a file called “boot.img” will not be accepted directly by the gateway using gwxfer – we attempted this to bypass signature verification)

Very little of the normal functionality of the gateway is contained in noboot.img. The vehicle largely ceases to operate while the gateway is in update mode. We could not move the vehicle.

**Strings from inside the noboot.img**

**Contents of a recent release.tgz file**

The code in noboot.img unpacks the Release.tgz file, which uncovered the following:

- A series of .hex files – upgrades for the individual ECUs
- A manifest file containing the list of .hex files along with their versions
- A metadata file containing CRC32s of the files with a signature

**Manifest file**

**Signed_metadata_map.tsv**

Once again, using a public key stored in the gateway firmware, the gateway appears to check the CRC32 and signature of each file.

The binary firmware for the named ECU contains the .hex files.

The following occurs to program an ECU:

- To make the update safe, prescribed actions are carried out (e.g., disconnect the battery using the main contactor)
- Using UDS Security Access commands, unlocks the ECU
- Send the firmware over UDS
- Restart the ECU

By writing directly to flash rather than using UDS, the gateway can also update itself. The filename is gtw.hex.

**Strings from the gateway firmware**

Updates are carried out sequentially. The same ECUs were updated for the two firmware updates that we observed, and this appeared to be most of those in the vehicle. We could not determine if partial updates are carried out.

Firmware updates are not accepted by default, by the ECUs around the vehicle, over the CAN interface. They must be unlocked over CAN via UDS Security Access. During a firmware update, several UDS Security Access transfers were sniffed.

We found that some ECUs, namely the IC, were using static seed/key pairs. Others seemed to be using a different seed. We did not attempt to gather seeds from the individual ECUs to gauge the quality of their randomness.

In noboot.img, there were several sections of code that seemed to handle different UDS seed/key algorithms, but we could not work out how it determined which one a given ECU used.

The firmware files sent to the ECUs do not appear to have any specific validation. This is probably performed by the ECU itself.

There was no signing of files according to analysis of the .hex files. Digital signatures can be identified using entropy analysis. They are almost always high entropy.

**Entropy of the RCCM firmware**

**Entropy of the DSP firmware**

**Entropy of GTW firmware – note spikes showing suspected key/signature material**

**Entropy of noboot.img showing high spike – likely public key for checking signatures**

During previous ECU updates, research found only CRC32 checks were performed. This made the process vulnerable because it allowed malicious firmware to be loaded onto the gateway and then onto other ECUs.

It appears that some issues still persist.

There is no integrity protection or signature for the SD card. An attacker could modify the contents, which could have a security implication, for example time-of-check to time-of-use (TOCTOU) vulnerabilities, where the firmware signature is verified, an attacker modifies the firmware, and it is loaded onto an ECU. This could be difficult to exploit.

UDS Security Access is used to update the individual ECUs. Security Access is a simple challenge/response algorithm. The following occurs to enter the mode whereby certain operations can be carried out:

- Gateway requests a seed from ECU
- ECU sends seed
- Gateway uses algorithm to convert seed into correct key
- Key is sent back to ECU to enable secure operations
- Firmware loaded onto ECU by gateway

To prevent replay attacks, the seed should be random and of an adequate length. On the Tesla, the ECUs responded with fixed seed values in several cases. Of note, this occurred with either the FPGA or the power management microcontroller – the devices in the instrument cluster.

Typically one or more of the following, the algorithm used to transform the seed into a key varies from ECU to ECU:

- Bitwise XOR
- Bitwise Shift
- Mixing bits
- AES Encryption (symmetric encryption, where key is known by both parties)
- RSA Encryption (asymmetric encryption, where ECU knows only a public key and updater a private key)

Observation of seed/key pairs usually renders the first three methods easy to reverse engineer. The number of pairs required varies from just a few pairs up to tens or hundreds.

Because it isn’t possible to determine the key from encrypted data, AES encryption is generally secure from an attacker intercepting communication. However, there remains a risk that the key could be recovered directly from an endpoint – either the ECU or the device programming it.

Because most ECUs are microcontrollers with integrated flash, AES keys can be stored internally which prevents trivial readout. To obtain an AES key this way, an attacker would probably have to go to extreme lengths.

RSA encryption only places a public key on the ECUs. It would not be possible for an attacker to generate a seed/key pair without access to the private key, even if they recover this. This adds additional security to symmetric encryption. It seems to be a comparatively rare mechanism in ECUs.

The security of the programming device is tantamount with all of these methods. An attacker can recover the algorithm, AES key or RSA private key, if they can gain access to this, and thereby activate UDS on ECUs.

Various techniques have been used by traditional automotive diagnostic and programming tools to make themselves secure:

- Restricting distribution of software to trusted parties (although this is now largely ineffective as copied software is spread through the Internet)
- Restrictive licensing, including online activation and hardware dongles
- Use of hardware dongles to perform seed/key transform
- Remote generation of seed/key pairs using an online service

Thanks to these techniques, it is more difficulty to attack UDS Security Access seed/key algorithms.

But Tesla is different. Traditionally, a specific laptop or device – which can be tightly controlled – has been used to perform diagnostics and programming by a dealer or garage.

Tesla upgrades are performed remotely. The gateway must implement the means to perform UDS Secure Access on all the ECUs in the vehicle because the programming device must be implemented in the gateway itself.

This makes it easy for an attacker to get their hands on them, to recover and reverse engineer at their leisure. However, this could still be a very time-consuming process, especially given the large number of ECUs the vehicle has.

The overall process of UDS should not be considered secure from an active attacker on the CAN bus, even if the seed/key process is considered secure. There is nothing to prevent an attacker from interacting with the device once it has been unlocked by the gateway using UDS. This can allow them to execute code, read back firmware, and write their own firmware to the device.

Although the gateway checks the cryptographic signatures of the individual .hex files for the ECUs, there is scarce evidence to suggest that the ECUs perform signature checks on the firmware that is sent to them via UDS.

ECUs, especially the more basic ones, rarely do any firmware signature checking. As part of the firmware update process, it is common for only basic checks such as CRC or a simple module checksum to be performed.

### Wi-Fi and Cellular Modem USB firmware updates

Connected over USB to the VCM in the CID, the Wi-Fi module and Cellular Modem also receive updates via this channel.

### Wi-Fi module

A series of tools and scripts in /usr/local/bin updates the Parrot FC6050W Wi-Fi module.

If the firmware update is being carried out by the shell script method, the script new-pflasher is called, providing several arguments for a loader, installer, and finally application. The binary new-pflasher-core is then called by this scrip.

The pflasher-core binary is called directly from within the function do_upgrade_parrot if the firmware update is being carried out by the binary updater.

We did not find any Tesla specific functionality when we examined the firmware files located in /usr/deploy/. We could not find any generic firmware for the modules either.

**The Parrot Wi-Fi module firmware files**

### Cellular Modem

The shell script /usr/local/bin/sierra-update is called, regardless of the mechanism used to update the modem. This calls a binary SwiFirmwareDownloadUMTS that sends firmware to the modem.

We found the firmware for the cellular modem in /usr/deploy/sierra/. These are two .cwe files

**The modem firmware files**

Like the Wi-Fi module, we could not find any signs that this firmware has any specific Tesla functionality.

**Notes and Further Work**

### Interesting Findings

We found a number of other interesting points whilst investigating the firmware update mechanism.

### Remote Feature Enable

In order to enable the autopilot, the shell script enable-autopilot-after-purchase.sh updates the gateway internal.dat.

We believe this indicates that functionality may be remotely enabled and disabled.

No range or battery information was contained in Internal.dat, so we have been unable to ascertain how Tesla remotely alter these.

**The shell script to enable autopilot**

### Security Token Update

A security token – downloaded daily from Tesla’s servers, over the VPN – must be known in order to access diagnostics, login as root, or send commands between the CID and IC.

**Curl command to obtain security token**

The current security token is posted to the server to obtain the next, one. This prevents someone with valid VPN keys from obtaining tokens for any car.

### Escalation Strategy

With names ranging from “INDIFFERENT” to “SUICIDE_BOMBER”, there is a list of escalation strategies in the updater binary, which appear to be strategies for retries of downloads and user prompts on the UI.

**Escalation strategies**

### Heavy Use of Shell Scripts

There are 85 shell scripts implementing various functionality in the /usr/local/bin directory..

The CID still uses many of these are, although functionality has moved into binaries for some of them.

**Some of the many shell scripts**

With many of these scripts being heavily commented, you can get an interesting insight into how they function.

### Further Work

We have only really seen a fraction of what the Tesla can do here. Our biggest obstacle is the lack of spare parts – we could do a lot more detailed reverse engineering if we had a spare CID.

There remain areas to investigate:

- We have not explored much of the Qt binaries that drive the CID UI; they contain functionality for connecting out to servers, including the daily service menu PIN code algorithm.
- It was not possible for us to determine how certain functionality (for example, the vehicle’s range) could be altered remotely by Tesla.
- We did not find any evidence of stolen vehicle tracking, remote cut-out or eCall in any of the aspects of the system that we examined. We would be interested to know how this is handled.
- There was no Autopilot module in the vehicle we tested, and there were scant references to it on the CID. The autopilot would probably employ additional security to protect significant intellectual property it would contain.

**Read Part 1: analysing the hardware, complete with a 14 layer PCB in the CID**
