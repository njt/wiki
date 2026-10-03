---
url: https://devblogs.microsoft.com/commandline/wslc-architecture-deep-dive/
date_fetched: 2026-10-03
---

WSL containers is now generally available! Check out this blog post to learn more about this overall feature enabling seamless access to Linux containers on Windows via the new “wslc.exe” command. This new Linux container platform comes with multiple architectures changes compared to WSL, which we’ll detail in this post.

## Session model

Similarly to WSL, client processes call into “wslservice.exe”, which is a privileged Windows service. That service has the capability to create virtual machines (via HCS), and use those to run Linux workflows.

A key difference with WSL’s architecture though is that wslservice.exe does not retain ownership of the virtual machine. Instead, it creates a child process, wslcsession.exe, which runs on behalf of the calling user and will perform all session operations (creating containers, mounting directories, binding networking ports, etc) on behalf of the user.

This new model allows WSLC to have both strong isolation between sessions, since they live in different processes, but also reinforced security boundaries, since sessions operations are executed in a less privileged process than wslservice.exe. The below diagram gives an overview of how a WSLC session is created:

## Storage

WSLC offers several primitives to store data within a WSLC session, or in the Windows storage stack. This section details the different storage options, and how they’re designed. Session VHDs Each WSLC session has its own storage VHD. This VHD is use to store the state of sessions (available images, containers, networks, volumes, etc).

When using wslc.exe, these VHDs are stored in `%AppData%\Local\wslc\sessions`. Container volumes allow containers to store data outside of their scratch space (which is discarded when the container is deleted).

The simplest usecase is to use a volume to share a Windows path with a container, like this:

```
$ wslc container run -v C:\Windows\System32\drivers\etc:/volume -it debian:latest ls /volume
hosts hosts.ics lmhosts.sam networks protocol services
```
Under the hood, these volumes are implemented by mounting virtiofs shares inside the Linux virtual machine, and making them available to the container. Inside the virtual machine, these mounts are created under `/mnt`, and then attached to the container as bind mounts:

On the Windows side, the mountpoint is accessed over virtiofs, which is high performance filesystem designed specifically for interoperability between a hypervisor and a virtual machine. Compared to plan9, virtiofs is about twice as fast. VHD volumes VHD volumes are a special kind of container volumes that is backed by a VHD instead of a Windows path. This kind of volume is useful when a container needs a native linux filesystem or wants to enforce a limit how the volume size. Here’s an example on how to create a VHD volume:

```
$ wslc volume create --driver vhd -o SizeBytes=200000000 my-volume
```
Once created, the volume can be mounted by name in one or multiple containers:

```
$ wslc container run -v my-volume:/volume -it debian:latest findmnt /volume TARGET SOURCE FSTYPE OPTIONS /volume /dev/sdf ext4 rw,relatime,stripe=4
```
## Networking

WSLC offers network connectivity through a new networking model: Consommé. This networking setup allows WSLC to have fine control over the networking behavior for the Linux Virtual machine (which is needed for advanced scenarios like port mapping, host loopback, …) while integrating with the Windows networking stack. In this model, all the Linux virtual machines’ traffic is sent as ethernet frames to a virtio queue, which is then read by a Windows process running on behalf of the user.

That process then provides access to various networking service to the virtual machine, such as:

- Answering DNS queries
- Routing for UDP & TCP traffic
- Port mapping

One of the major advantages that this approach offers is that the traffic that’s routed out of the virtual machine is sent on behalf of the user owning the wslc session, so traffic flows as it was emitted by a regular Windows process, which provides extensive compatibility with VPNs and firewalls. Below is an example flow for a scenario where a container runs nginx with port 8000 mapped to port 80 into the container:

## Learning more

Would you like to learn more about WSLC’s technical details? WSLC is open source! Head over to microsoft/WSL to read the code, build your own, and contribute!

Isn’t there a typo in?

“Container volumes Container volumes allow containers to store data outside of their scratch space (which is discarded when the container is deleted).”

Container volumes twice?

Fixed, thank you !

Are there any plans to support compose files?

The linked “WSL containers is now generally available” blog post contains: “Our top feature request for WSLc is adding compose support, and this will be our focus for our next iterations.”

That’s unfortunate.
