---
url: https://blog.remijouan.net/posts/libzmq-vsock-pyzmq/
date_fetched: 2026-09-11
---

# Use VSOCK with libzmq

## AF_VSOCK

VSOCK is not new anymore it has been around in Linux kernel since 4.8. For those who do not know what this is, it was previously developed by VMware under the name VMCI. AF_VSOCK is a socket address family like AF_INET or AF_UNIX. It’s an address family designed for communication between a VM (guest) and the underlying hypervisor (host), you can also build communication between multiple guests on the same host. Previously, this kind of communication was done with a serial port, a great example is https://pve.proxmox.com/wiki/Qemu-guest-agent.

AF_VSOCK looks and feels like a Unix socket with TCP/UDP-like features. You have addresses and ports, you also have stream and datagram.

The man page https://man7.org/linux/man-pages/man7/vsock.7.html

You have 32-bit addresses that are called “context identifier”, with reserved CID: VMADDR_CID_ANY, VMADDR_CID_HYPERVISOR, VMADDR_CID_LOCAL (new in 5.6), VMADDR_CID_HOST. Regarding port numbers, you can allocate 32-bit ports and those under 1024 need root access. just like TCP/UDP you can then have different communication with the same CID by using different port.

VSOCK has been around for some time, but the support is growing slowly. Python/C/Golang/rust support it and you have some basic tools and SDK.

- https://stefano-garzarella.github.io/posts/2021-01-22-socat-vsock/
- https://github.com/rust-vsock/tokio-vsock
- https://mdlayher.com/blog/linux-vm-sockets-in-go/
- https://docs.python.org/3/library/socket.html#socket.AF_VSOCK
- https://gitlab.com/vsock/vsock
- AWS uses it for its Nitro Enclaves feature https://docs.aws.amazon.com/enclaves/latest/user/developing-applications-linux.html

## Support in libzmq

If you are planning to use vsock, you’ll quickly notice that you only have low-level bindings available, there is no magical library that abstracts the socket for you. You probably have not opened a socket yourself in a long time, you are trying to remember how to: poll, recv, send, open, listen, bind.

Fortunately there is a great library for that https://zeromq.org/, it implements basic patterns and good practices over almost any kind of socket, it even has security features http://curvezmq.org/.

To my great surprise, there was a VMCI implementation https://libzmq.readthedocs.io/en/latest/zmq_vmci.html, but no VSOCK.

Proposing something was straightforward; I almost just copied and pasted the code from VMCI. https://github.com/zeromq/libzmq/pull/4822

Having VSOCK in libzmq offers numerous advantages.

- use AF_VSOCK in every language that has a libzmq binding: python, ruby, nodejs, perl, java, lua …
- use libzmq features over AF_VSOCK : authentication with curve, different message pattern, req/rep, pub/sub.

libzmq does not make regular releases because it is stable and does not move that much. It is also a library made for embedded software it’s easy to build it statically over a specific commit.

VSOCK will certainly be available in a stable release someday, in the meantime i have forked pyzmq to build it with the latest libzmq commit, https://github.com/remijouannet/pyzmq-vsock.

libzmq documentation on vsock https://github.com/zeromq/libzmq/blob/master/doc/zmq_vsock.adoc.

## pyzmq-vsock examples

## Hello world

Since Linux 5.6 there is a loopback CID (VMADDR_CID_LOCAL) so you can test VSOCK without running a VM. Use @ in ZMQ to directly bind on this loopback.

Here is a basic example of using req/rep socket pair over VSOCK loopback. if you ever want to test VSOCK in real condition with guest and host, your QEMU command line must add vsock device (or your libvirt configuration https://libvirt.org/formatdomain.html#vsock).

```
1export CID=100
2/usr/local/bin/qemu-system-x86_64 \
3...
4    -device vhost-vsock-pci,id=vhost-vsock-pci0,guest-cid=$CID
5...
```
```
 1
 2# vsock_loopback is probably not loaded on your machine
 3# sudo modprobe vsock_loopback
 4
 5python3 -m venv venv
 6
 7venv/bin/pip install \
 8    https://github.com/remijouannet/pyzmq-vsock/releases/download/27.2.0.dev0%2B4649337/pyzmq-27.2.0.dev0+4649337-cp312-abi3-manylinux_2_26_x86_64.manylinux_2_28_x86_64.whl
 9
10# Examples from https://zeromq.org/get-started/?language=python# adapted for vsock
11cat <<EOF > rep.py
12#
13#   Hello World server in Python
14#   Binds REP socket to vsock://@:5555
15#   Expects b"Hello" from client, replies with b"World"
16#
17
18import time
19import zmq
20
21context = zmq.Context()
22socket = context.socket(zmq.REP)
23socket.bind("vsock://@:5555")
24
25while True:
26    #  Wait for next request from client
27    message = socket.recv()
28    print(f"Received request: {message}")
29
30    #  Do some 'work'
31    time.sleep(1)
32
33    #  Send reply back to client
34    socket.send(b"World")
35EOF
36
37cat <<EOF > req.py
38#
39#   Hello World client in Python
40#   Connects REQ socket to vsock://@:5555
41#   Sends "Hello" to server, expects "World" back
42#
43
44import zmq
45
46context = zmq.Context()
47
48#  Socket to talk to server
49print("Connecting to hello world server…")
50socket = context.socket(zmq.REQ)
51socket.connect("vsock://@:5555")
52
53#  Do 5 requests, waiting each time for a response
54for request in range(5):
55    print(f"Sending request {request} …")
56    socket.send(b"Hello")
57
58    #  Get the reply.
59    message = socket.recv()
60    print(f"Received reply {request} [ {message} ]")
61EOF
62
63venv/bin/python3 rep.py &
64[1] 88069
65
66venv/bin/python3 req.py  
67Connecting to hello world server…
68Sending request 0 …
69Received request: b'Hello'
70Received reply 0 [ b'World' ]
71Sending request 1 …
72Received request: b'Hello'
73Received reply 1 [ b'World' ]
74Sending request 2 …
75Received request: b'Hello'
76Received reply 2 [ b'World' ]
77Sending request 3 …
78Received request: b'Hello'
79Received reply 3 [ b'World' ]
80Sending request 4 …
81Received request: b'Hello'
82Received reply 4 [ b'World' ]
```
## Curve with asyncio

Hello World is great, but you’re probably going to use asyncio and curve in production. The following example is taken from pyzmq https://github.com/zeromq/pyzmq/blob/main/examples/security/asyncio-ironhouse.py.

Script to generate Curve keys

```
 1#!/usr/bin/env python
 2
 3import json
 4
 5import zmq
 6import zmq.auth
 7
 8keys_file = "keys.json"
 9
10client_pub, client_priv = zmq.curve_keypair()
11client2_pub, client2_priv = zmq.curve_keypair()
12server_pub, server_priv = zmq.curve_keypair()
13
14with open(keys_file, "w") as f:
15    json.dump(
16        {
17            "client": [client_pub.decode(), client_priv.decode()],
18            "client2": [client2_pub.decode(), client2_priv.decode()],
19            "server": [server_pub.decode(), server_priv.decode()],
20        },
21        f,
22        sort_keys=True,
23        indent=4,
24    )
```
REP server

```
 1#!/usr/bin/env python
 2
 3import asyncio
 4import json
 5import logging
 6
 7import zmq
 8import zmq.auth
 9from zmq.asyncio import Context
10from zmq.auth.asyncio import AsyncioAuthenticator
11
12LOGGER = logging.getLogger(__name__)
13
14
15async def run(keys: dict) -> None:
16    ctx = Context.instance()
17
18    # Start an authenticator for this context.
19    auth = AsyncioAuthenticator(ctx)
20    auth.start()
21    auth.certs["*"] = {keys["client"][0].encode(): True}
22
23    server = ctx.socket(zmq.REP)
24
25    server.curve_publickey = zmq.utils.z85.decode(keys['server'][0])
26    server.curve_secretkey = zmq.utils.z85.decode(keys['server'][1])
27    server.curve_server = True  # must come before bind
28    server.bind('vsock://@:9000')
29
30    msg = await server.recv()
31    LOGGER.info(f"Received {msg!r}")
32    if msg == b"Hello":
33        LOGGER.info("Ironhouse test OK")
34    await server.send(b"World")
35
36    # close sockets
37    server.close()
38    auth.stop()
39
40
41if __name__ == '__main__':
42    if not zmq.has("vsock") or not zmq.has("curve"):
43        raise RuntimeError(
44            f"Security is not supported in libzmq version < 4.0. libzmq version {zmq.zmq_version()}"
45        )
46
47    level = logging.DEBUG
48
49    logging.basicConfig(level=level, format="[%(levelname)s] %(message)s")
50
51    with open("keys.json") as f:
52        keys = json.load(f)
53
54    asyncio.run(run(keys))
```
REQ client

```
 1#!/usr/bin/env python
 2
 3import asyncio
 4import json
 5import logging
 6
 7import zmq
 8import zmq.auth
 9from zmq.asyncio import Context
10from zmq.auth.asyncio import AsyncioAuthenticator
11
12LOGGER = logging.getLogger(__name__)
13
14
15async def run(keys: dict) -> None:
16    ctx = Context.instance()
17
18    # Start an authenticator for this context.
19    auth = AsyncioAuthenticator(ctx)
20    auth.start()
21
22    client = ctx.socket(zmq.REQ)
23    client.curve_publickey = zmq.utils.z85.decode(keys['client'][0])
24    client.curve_secretkey = zmq.utils.z85.decode(keys['client'][1])
25
26    client.curve_serverkey = zmq.utils.z85.decode(keys['server'][0])
27    client.connect('vsock://@:9000')
28
29    await client.send(b"Hello")
30    reply = await client.recv()
31    LOGGER.info(f"Received reply {reply!r}")
32
33    client.close()
34    auth.stop()
35
36
37if __name__ == '__main__':
38    if not zmq.has("vsock") or not zmq.has("curve"):
39        raise RuntimeError(
40            f"Security is not supported in libzmq version < 4.0. libzmq version {zmq.zmq_version()}"
41        )
42
43    level = logging.DEBUG
44
45    logging.basicConfig(level=level, format="[%(levelname)s] %(message)s")
46
47    with open("keys.json") as f:
48        keys = json.load(f)
49
50    asyncio.run(run(keys))
```
The output when authentication succeeds

```
 1venv/bin/python generate_keys.py
 2
 3venv/bin/python rep_asyncio_curve.py 
 4[DEBUG] Using selector: EpollSelector
 5[DEBUG] Starting
 6[DEBUG] version: b'1.0', request_id: b'1', domain: '', address: '', identity: b'', mechanism: b'CURVE'
 7[DEBUG] ALLOWED (CURVE) domain=* client_key=b'HhdIwzo4=a}1F#eL{}rs4C1Hgx.Z4nd#/JqIasmP'
 8[DEBUG] ZAP reply code=b'200' text=b'OK'
 9[INFO] Received b'Hello'
10[INFO] Ironhouse test OK
11
12venv/bin/python req_asyncio_curve.py 
13[DEBUG] Using selector: EpollSelector
14[DEBUG] Starting
15[INFO] Received reply b'World'
```
The output when using an unauthorized key

```
 1venv/bin/python generate_keys.py
 2
 3venv/bin/python rep_asyncio_curve.py 
 4[DEBUG] Using selector: EpollSelector
 5[DEBUG] Starting
 6[DEBUG] version: b'1.0', request_id: b'1', domain: '', address: '', identity: b'', mechanism: b'CURVE'
 7[DEBUG] DENIED (CURVE) domain=* client_key=b'P4//#^+&*nKTcb]6*u:zy<blBUAX%IaSn=PLNG-/'
 8[DEBUG] ZAP reply code=b'400' text=b'Unknown key'
 9
10venv/bin/python req_asyncio_curve_wrong_key.py
11[DEBUG] Using selector: EpollSelector
12[DEBUG] Starting
13[INFO] No reply: server rejected this client key
```
