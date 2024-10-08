![header-image](http://coddingtonbear-public.s3.amazonaws.com/github/logitech-flow-kvm/mx_keys_buttons.jpg)

Quickly switch between paired devices when using a Logitech mouse and keyboard that supports connecting to multiple devices.

Do you use Logitech devices that support multiple hosts and find it a little frustrating how tedious it is to switch between hosts when you're using Linux due to Logitech's "Flow" features being unsupported there?  Good news: you can have "Flow"-like features on Linux now, too.

This utility works by monitoring one of your attached Logitech devices to see what host it is currently connected to, and then, if that host changes, instructing other connected devices to switch to the same host.

# Features

- Automatically switches all devices from one host to another when just one of your devices switches hosts.  This is particularly useful if you are using a a device like the MX Keys Mini which includes buttons that can be used for switching hosts with a single keypress.
- Securely keeps clipboards in sync when switching between hosts. Now you can copy/paste from one host to another without thinking anything about it.

# Installation

```
pip install logitech-flow-kvm
```

You can also install the in-development version with:

```

pip install https://github.com/coddingtonbear/logitech-flow-kvm/archive/master.zip

```

# Basic Use

One of your computers will serve as the "server" for managing this, and all others will be "clients".

## Server

On the computer you've decided to use as your server, get the ID of the device you'd like to serve as your "leader", and the IDs of other devices you'd like to use as "followers".  The leader device -- I recommend using your keyboard -- will be the one we watch.  The followers will just be told which host to connect to if your leader device's host changes.  Previous versions of this software used the path rather than the ID, but it was changed to prevent the values from changing.

```
> logitech-flow-kvm list-devices

┏━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━┓
┃ ID       ┃ Product ┃ Name          ┃ Path           ┃
┡━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━┩
│ 51264495 │ B366    │ MX Mechanical │ /dev/hidraw4:1 │
│ 3B776BDD │ B034    │ MX Master 3S  │ /dev/hidraw4:2 │
└──────────┴─────────┴───────────────┴────────────────┘
```

In this example, I'll be using my keyboard (`51264495`) as my "leader", and my mouse (`3B776BDD`) as a follower.

You can run a command like this:

```
> logitech-flow-kvm flow-server 1 51264495 3B776BDD
```

Note that the `1` in the above command immediately after `flow-server` indicates that the host number of your server is `1`.  This should match the host number you've paired your mouse and keyboard with your device as (i.e. when your mouse or keyboard is connected to this computer, the light for `1` is lit on the keyboard or mouse's device selector).

After running the above command, you'll receive some output indicating what hostnames the server was bound to; on my computer, this looks like this:

```
...
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:24801
 * Running on http://10.224.224.120:24801
...
```

From the above lines, you'll want to select an IP address that can be reached from your clients.  In my case, I'll be using `10.224.224.120` for connections from the clients to the server.

## Client

On the other computers you'd like to use this feature with, you can run the following command:

```
> logitech-flow-kvm flow-client 2 10.224.224.120
```

If this is your first time connecting to this server, you will be walked through a brief pairing process for establishing a secure connection between your server and client instances.  Afterward, the client will connect, gather some configuration options from the server, and will instruct your "follower" devices to change their hosts as necessary in the future.

Note that the `2` above after `flow-client` indicates the host number your devices have paired with your computer under.  See "Server" above for details.

# How to

## Finding available devices

You can get a list of available devices using the `list-devices` subcommand:

```
> logitech-flow-kvm list-devices

Finding devices... ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
┏━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━┓
┃ ID             ┃ Product ┃ Name                       ┃ Serial   ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━┩
│ /dev/hidraw4:1 │ B369    │ MX Keys Mini               │ 08F5F681 │
│ /dev/hidraw5:1 │ 4082    │ MX Master 3 Wireless Mouse │ 0F591C09 │
└────────────────┴─────────┴────────────────────────────┴──────────┘
```

## Switching which host a device is connected to

You can change the relevant device to your desired host number using the `switch-to-host` command:

```
> logitech-flow-kvm switch-to-host /dev/hidraw4:1 2
```

The above command will tell the device having the id `/dev/hidraw4:1` to connect to whichever device is paired as its #`2` device.

## Running a command when a device connects or disconnects

You can see when a device connects or disconnects from the receiver using the following example:

```
> logitech-flow-kvm watch /dev/hidraw4:1
```

If you'd like to run a command when a device connects or disconnects, use the `--on-disconnect-execute` or `--on-connect-execute` arguments.  See the "Automatically switch your mouse to a different host when your keyboard disconnects" section below for a concrete example of how you might use this.

## Automatically switch your mouse to a different host when your keyboard disconnects

Note: this isn't the recommended way of handling this sort of thing -- you probably want to follow the instructions above under "Basic Use" above.

If you have two devices:

```
> logitech-flow-kvm list-devices

┏━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━┓
┃ ID             ┃ Product ┃ Name                       ┃ Serial   ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━┩
│ /dev/hidraw4:1 │ B369    │ MX Keys Mini               │ 08F5F681 │
│ /dev/hidraw5:1 │ 4082    │ MX Master 3 Wireless Mouse │ 0F591C09 │
└────────────────┴─────────┴────────────────────────────┴──────────┘
```

You can respond run a command that will listen to when the "MX Keys Mini" device above disconnects, and when it does, ask the "MX Master 3 Wireless Mouse" to connect to a specific host:

```
> logitech-flow-kvm watch --on-disconnect-execute="logitech-flow-kvm switch-to-host /dev/hidraw5:1 2" /dev/hidraw4:1
```

# Credits

Much of what this tool does is dependent upon the work put together by the folks working on [Solaar](https://github.com/pwr-Solaar/Solaar).
