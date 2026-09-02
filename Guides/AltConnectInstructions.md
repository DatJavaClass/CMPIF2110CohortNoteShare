# AltConnect: Pitt VPN Without the GlobalProtect Client
# By Vic Sverdlin, Grammar De-Abominated Edition

Pitt's VPN is Palo Alto GlobalProtect, and the official way in is
Palo Alto's client. This guide is the unofficial way in: same VPN, 
credentials, no extra crap Palo Alto Software installed. Why is it crap?
Well that's because it's Palo Alto. Plus the fact that the GlobalProtect
software can be difficult to fully remove, so this route skips installing 
it in the first place.

My kicker? OpenConnect, an open source VPN client that speaks
the GlobalProtect protocol. We'll run it inside WSL2, so the tunnel lives
entirely in Linux and Windows never meets the client at all.

## What you need

- Windows with WSL2 and Ubuntu installed (any recent Ubuntu works, this
  was built on 24.04). If `wsl` in PowerShell gives you a Linux prompt,
  you are set. If not, Step 0 has you covered.
- Your Pitt username and password, with Duo ready
  on your phone.
- The password for your WSL Linux user, since installing needs `sudo`.

## Step 0: Get WSL (skip if you have it)

WSL is Microsoft's official way to run real Linux inside Windows, and
one command installs the whole thing. In PowerShell, run as
administrator:

```powershell
wsl --install ## Ubuntu included, one command
```

Restart when it asks. On first launch Ubuntu has you create a Linux
username and password. That password is your sudo password for
everything below, so pick one you will remember. From then on, WSL is
a tab choice in Windows Terminal's dropdown.

Two things can trip this. If the command just prints help text, WSL
is already half there, so run `wsl --install -d Ubuntu` instead. If
Ubuntu will not boot after the restart, your BIOS has virtualization
switched off, and Microsoft's WSL troubleshooting page walks you
through turning it on.

## Step 1: Install OpenConnect

In a WSL tab:

```bash
sudo apt update && sudo apt install -y openconnect ## one small package
```

That is the whole install. A few megabytes.

## Step 2: Connect

Still in that WSL tab, with your own username in place of `abc123`:

```bash
sudo openconnect --protocol=gp --authgroup=BYOD-GATEWAY-CL portal-palo.pitt.edu -u abc123
```

Three prompts follow, in order. First your WSL sudo password (sudo
skips this one if Step 1 was recent, it remembers you for a few
minutes). Then your Pitt password. Then Duo, so approve the push on
your phone when it asks.

Two notes on that command:

- `portal-palo.pitt.edu` is Pitt's GlobalProtect portal. If it will not
  respond, `portal2-palo.pitt.edu` is the backup.
- `--authgroup=BYOD-GATEWAY-CL` pre-answers a gateway question the
  portal would otherwise ask. If you leave it off, you get a prompt
  reading `GATEWAY: [BYOD-GATEWAY-CL|BYOD-GATEWAY-FQ]` and you type one
  of those names exactly, capitals and hyphens included. This guide was
  proven on CL. If it misbehaves, FQ is the natural one to try, though
  it has not been tested here.

## Step 3: Know you are in

Success looks like a line reporting `Connected as` followed by an IP
address, and then the command just sits there. That is not a hang. That
running command IS the tunnel. Minimize the window if you like, but do
not close it.

## Step 4: Use it

Here is the one concept this whole setup hangs on: only Linux is on the
VPN. Everything inside WSL can reach Pitt's protected resources.
Everything on the Windows side cannot, which is exactly the deal we
wanted.

> **What is actually happening:** WSL2 is a small Linux virtual machine
> with its own network stack. OpenConnect builds the tunnel inside that
> stack, so Linux programs route through Pitt while Windows programs
> keep using your normal connection. Nothing was installed on Windows,
> so there is nothing on Windows to tunnel.

So work from WSL:

- Open a second WSL tab for your actual work. The tunnel tab stays busy.
- Your Windows drives are mounted under `/mnt`, so
  `C:\Users\you\Documents` is `/mnt/c/Users/you/Documents`.

### Wiring in VS Code

- `cd` to your working folder in that second tab and run `code .` there.
  The window that opens shows a green `WSL: Ubuntu` badge in the
  bottom-left corner, which means its terminals and extensions run
  inside Linux, on the VPN.
- The first time, VS Code will show your extensions as disabled. That is
  normal. Extensions install per side, so click `Install in WSL` on the
  ones you need (your database client, for instance). One time step.

### Wiring in Microsoft Access

Access is a Windows program, and Windows is not on the VPN, so Access
cannot reach the course server on its own. The workaround is a relay: a
small Linux tool listens inside WSL and forwards everything it hears
through the tunnel. Windows can always talk to WSL on `localhost`, so
Access talks to the relay and the relay talks to Pitt.

In a WSL tab (a third one, the relay needs to stay running too):

```bash
sudo apt install -y socat ## the relay tool
socat TCP-LISTEN:13306,fork,reuseaddr TCP:COURSE-SERVER:3306 ## relay on, leave tab open
```

Swap `COURSE-SERVER:3306` for the host and port your course materials
give you. The `13306` is the local door the relay listens on, picked
high so it cannot collide with a MySQL you may already run on Windows.

First grab the Windows ODBC driver for whatever the course server is
(MySQL's Connector/ODBC for a MySQL server), which is a normal
Windows install, not a Palo Alto one, so your machine survives. Then
point Access at the relay: External Data, New Data Source, From Other
Sources, ODBC Database. Access asks you to pick a data source, so
create a new one with that driver, and its dialog is where server
`127.0.0.1` and port `13306` go, plus the same database name,
username, and password the course gave you.

`Ctrl+C` in the relay tab shuts the relay down. It holds no login of
its own, it just forwards.

## Step 5: Disconnect

`Ctrl+C` in the tunnel tab. OpenConnect signs off with the gateway and
exits clean. Closing the tab also kills the tunnel and restores your
networking, but it skips the sign-off, so the gateway holds your
session open a while longer. Prefer `Ctrl+C`.

## Quick reference

The whole routine, once the install is behind you:

```bash
sudo openconnect --protocol=gp --authgroup=BYOD-GATEWAY-CL portal-palo.pitt.edu -u abc123 ## connect
```

Approve Duo, watch for `Connected as`, leave it running, work from a
second WSL tab. `Ctrl+C` in the tunnel tab when you are done.

## When something goes sideways

**"Temporary failure in name resolution."** WSL's built-in DNS relay
occasionally stops relaying, and this is what it looks like. The fix is
to hand Linux real DNS servers and tell WSL to stop meddling:

```bash
printf '\n[network]\ngenerateResolvConf=false\n' | sudo tee -a /etc/wsl.conf ## WSL: hands off DNS
sudo rm /etc/resolv.conf ## drop the generated file
printf 'nameserver 1.1.1.1\nnameserver 8.8.8.8\n' | sudo tee /etc/resolv.conf
```

Then restart WSL: from PowerShell run `wsl --shutdown`, wait a few
seconds, and open a new WSL tab. If names still fail after the restart,
run the last two lines once more (Ubuntu's own resolver sometimes
re-links the file during that first restart) and you are done for good.

**The portal rejects a password you know is right.** Try it again
before assuming the worst. Ours bounced once and accepted the identical
password on the next attempt.

**The gateway prompt repeats itself.** It wants the name typed exactly:
`BYOD-GATEWAY-CL`, all caps, both hyphens. Enter on an empty line just
asks again.

**Disconnected after about an hour.** The portal asks clients for
periodic health check-ins (HIP reports). Most sessions never notice. If
yours drops on the hour, reconnect with this added to the command:
`--csd-wrapper=/usr/libexec/openconnect/hipreport.sh`

**Forgot your WSL sudo password.** From PowerShell:
`wsl -u root passwd yourlinuxusername`, set a new one, carry on.

## What it remembers about you

Nothing. OpenConnect stores no credentials, no history, no config. No
service runs when you are not connected, and nothing starts at boot.
Your shell history keeps the command line itself, which is convenient,
since the up arrow is your reconnect button. The day you want it gone:

```bash
sudo apt remove --autoremove openconnect ## no survivors
```

## The one button version

Mine starts with a double click: terminal opens, tunnel comes up, Duo
pings my phone, and VS Code lands on the right folder by itself. That
part is a personal script rather than a walkthrough, so if you want it,
reach out and we'll see about getting you setup.

Remember, You're awesome. Stay Awesome.

-Vic Sverdlin
