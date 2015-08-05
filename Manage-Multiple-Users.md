Note: this feature is currently in progress on `manager` branch and not released yet.

If you want to develop a user management system, Shadowsocks provides an API that allows you to add/remove ports on the fly, as well as get transfer statistics from Shadowsocks.

If you only want to add multiple users without the on the fly feature, you can check [this tutorial](https://github.com/shadowsocks/shadowsocks/wiki/Configure-Multiple-Users).

Setup
-----

Turn on manager API by specifying `--manager-address`, which is either a Unix socket or an IP address:
```
# Use a Unix socket
ssserver --manager-address /var/run/shadowsocks-manager.sock -c tests/server-multi-passwd.json
# Use an IP address
ssserver --manager-address 127.0.0.1:6001 -c tests/server-multi-passwd.json
```

For security reasons, you should use Unix sockets.

When manager is enabled, [workers](https://github.com/shadowsocks/shadowsocks/wiki/Workers) and [graceful restart](https://github.com/shadowsocks/shadowsocks/wiki/Graceful-shutdown-and-restart) are disabled.

Protocol
--------

You can send UDP data to Shadowsocks.

```
command[: JSON data]
```

To add a port:

```
add: {"server_port": 8001, "password":"7cd308cc059"}
```

To remove a port:

```
remove: {"server_port": 8001}
```

To receive a pong:

```
ping
```

Shadowsocks will send back transfer statistics:

```
stat: {"8001":11370}
```

Example Code
------------

Here's code that demonstrates how to talk to the Shadowsocks server:
```
import socket

cli = socket.socket(socket.AF_UNIX, socket.SOCK_DGRAM)
cli.bind('/tmp/client.sock')  # address of the client
cli.sendto('ping', '/var/run/shadowsocks-manager.sock')  # address of Shadowsocks manager
cli.recvfrom(1506)  # You'll receive a pong

cli.sendto(b'add: {"server_port":8001, "password":"7cd308cc059"}', '/var/run/shadowsocks-manager.sock')

cli.sendto(b'remove: {"server_port":8001}', '/var/run/shadowsocks-manager.sock')

while True:
    print(cli.recvfrom(1506))  # when data is transferred on Shadowsocks, you'll receive stat info every 10 seconds
```