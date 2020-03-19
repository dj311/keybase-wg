# keybase-wg

This project gives Keybase teams quick and easy access to their own virtual private network. To use it, make sure
the Keybase daemon is open and logged in, then run `keybase-wg --team=<name>`. `CTRL+C` will leave the
network, and close down the program.

It requires Keybase and Wireguard to be installed. If you've got avahi then you'll also get a nice
internal naming system. I've only tested it on Ubuntu 19.10 so your mileage is bound to be pretty variable.

#### What does it do?
It'll set up a new network interface on your computer. That interface exposes an encrypted network
that only your team members have access to. On startup, you'll be assigned a static IP in the
network and it'll also give you a domain name structured as `<device>-<user>-<team>.local`. Connected
team members can be accessed via domains of the same format.

#### Why would you want it?
  1. Share a live Jupyter session with your team.
  2. Debug local instances of web servers.
  3. Play games over a LAN when you're really on a WLAN.
  4. Etcetera

#### How does it work?
Keybase provides each team with an encrypted key-value store, which this tool uses to bootstrap and synchronise
the information needed to setup the network with Wireguard.

At the beginning of each session, this tool generates a new Wireguard keypair and detects the devices external
IP address. It then shares the public key and IP with peers over the shared store. Next, it checks the shared
store for information on any peers, generates a Wireguard config, and starts up Wireguard.

It checks the shared store every 10 seconds for changes, then updates the Wireguard config if there are any.

#### Is it usable yet? Not really.
This is just a quick and dirty prototype at the moment. If you've stumbled upon this repo and want
to use it, I'd recommend reading the source code first. This will ensure it does what you were hoping it
does. It's less than 500 lines of Python so that shouldn't be too much of an issue.

With the current functionality, it's missing some pretty important details:
  1. Whenever a user is removed from the team, all Wireguard keys should to be rotated. Currently, if
     a device owned User A joins, then User A is removed from the team, this device will still be part
     of the network. Not ideal.
  2. If a devices external IP changes during the session, *newly* connected peers won't be able to
     connect to it. The fix is to check the external IP during each polling cycle.
  3. It crashes pretty regularly, and often fails to shutdown nicely. I think this is because of the way
     `avahi` is called in the background.
     
For more detail on the current status, checkout the ["`dev_docs`"](https://github.com/dj311/keybase-wg/blob/master/keybase-wg#L18).

#### How do I run it?
  1. Make sure Keybase and Wireguard are installed and working. The Keybase dameon should be running and logged in.
  2. Clone and enter this repo: `git clone https://github.com/dj311/keybase-wg.git && cd keybase-wg`
  3. Make and source a virtual env: `python3 -m venv .venv && source .venv/bin/activate`.
  4. Install the Python dependencies: `python3 -m pip install -r requirements.txt`.
  5. Run it: `./keybase-wg --team=<name>`.
  6. To stop the program and leave the network: `CTRL+C`.

I haven't tested, but tI can't think of any reason against running multiple of these at once for different teams.
