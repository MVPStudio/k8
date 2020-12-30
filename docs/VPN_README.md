# VPNs Plural

We have 2 VPNs. The first is an [OpenVpn](https://openvpn.net/) created by Continu for us. This gives us direct access
to all the machines in the cluster and we can ssh to individual nodes, etc. This is nice but it has a few downsides:

* Adding additional people to the VPN requires talking to Continu to set up certs and things.
* It provides a lot of access including the ability to send packets to individual nodes!

What we really want is the above VPN for MVP Studio admins but for regular users who just need to run `kubectl` we want
a VPN that limits their access to just the master server and we'd like to be able to add and remove new users easily and
without help from Continu. To this end we also have a simple [WireGuard](https://www.wireguard.com/) VPN.

# The WireGuard VPN

To access the [WireGuard](https://www.wireguard.com/) VPN users must first [install
WireGuard](https://www.wireguard.com/install/). 

Setting up WireGuard for a user consists of the following steps

1. Select a unique IP address for the user
2. Have the user generate a public/private key pair
3. Install the user on the server
4. Have the user create a client configuration
5. Have the user test their connection to the server

How the user does some of these steps depends on whether they are using the command-line or GUI version of WireGuard. 

## 1. Select a unique IP address for the user

NOTE: Steps 1 and 2 can take place in parallel. 

Figure out what the next free IP address is by looking at [the server config file](../ansible/templates/wg0.conf). Note 
that each user has a different number for the `AllowedIPs` line between `100.65.0.` and `/32`. Look at the number for 
the last user in the list, and increment that number to select the user's IP address. Send the user this IP address so 
they can create their configuration. 

## 2. Have the user generate a public/private key pair

How they do this depends on whether they are using the command-line or GUI version of WireGuard. Once this is done, have 
the user send their public key to you so you can install them on the server. 

### Command-line

To generate keys with the command-line, see [the WireGuard docs](https://www.wireguard.com/quickstart/#key-generation). 

### GUI (Mac or Windows)

To generate keys with the GUI, create a new tunnel by clicking on the `+` on the lower left of the GUI and select `Add 
Empty Tunnel...`. You will see something like the following. Give the tunnel a name (e.g. MVP), and the save. 

![image](https://user-images.githubusercontent.com/311063/85208559-61b9a500-b2e6-11ea-9c2f-cea6515cb8ff.png)

## 3. Install the user on the server

NOTE: Steps 3 and 4 can take place in parallel. 

To install a user on the server, edit [the server config file](../ansible/templates/wg0.conf) adding a new `[Peer]` 
section adding the user's public key and the unique IP address they were assigned in step 1 above. Be sure to add a 
comment indicating what actual human the public key belongs to. Finally, to apply the changed config use Ansible:

```
ansible-playbook -i inventory_wg.yml -l master --tags=wg_config --ask-vault-pass playbook.yml
```

You can find the password for the Ansible vault in the `continu-k8.txt` file in our secrets bucket
(`passwords.mvpstudio.org` bucket on GCE).

Note that `inventory_wg.yml` is the same as `inventory.yml` but the IP addresses are the addresses that show up in the
WireGuard tunnel. If you are on the OpenVpn tunnel use `inventory.yml` instead.

Finally, create a pull request to check in the modified `wg0.conf` file.

The user can now access the master server at 100.64.0.1 on the WireGuard network.

## 4. Have the user create a client configuration

The client configuration consists of the following text, with the users unique number replacing the `XXX` after 
`Address`, and with their actual private key replacing `<private key here>`. 

```
[Interface]
Address = 100.65.0.XXX/10
PrivateKey = <private key here>
ListenPort = 51820

[Peer]
Endpoint = 50.201.1.196:51820
PublicKey = zEEB7K7+iY4OOZTYffQI7s0xsC2bq6aO+B6RTs6BzVo=
AllowedIPs = 100.64.0.1/32
PersistentKeepalive = 25
```

How the users create the configuration depends on whether they are using the command-line or GUI version of WireGuard. 

### Command-line

To create the configuration for command-line users, create a text file with the above contents. 

### GUI (Mac or Windows)

To create the configuration with the GUI, bring up the tunnel editor with the tunnel created in step 2. Then edit the 
lower text area so it has the above contents. 

## 5. Have the user test their connection to the server

The user can test their WireGuard connection with the following instructions:

### Command-line

* Activate your connection with the command `wg-quick up /path/to/config.conf` where `/path/to/config.conf` is the 
client configuration file you created in step 4. 
* Test the connection to the master via `ping 100.64.0.1` - you should get timely responses.
* Load the page https://100.64.0.1:8443 in your browser - it should load. You might get a warning about the certificate 
not being up to date. You can ignore it. 
* When done, deactivate the connection with the command `wg-quick down /path/to/config.conf`. 

### GUI

* Select the tunnel for MVP from the list on the left, and then press the `Activate` button. 
* Test the connection to the master via `ping 100.64.0.1` - you should get timely responses.
* Load the page https://100.64.0.1:8443 in your browser - it should load. You might get a warning about the certificate 
not being up to date. You can ignore it. 
* When done, break down the connection by pressing the `Deactivate` button. 

# The OpenVpn VPN

To connect to the "full VPN" that gives you access to not only the master server but also all the minions you will need
Continu to set up a username, password, and a variety of certificates and keys. They will general provide all this
information to you on a thumb drive or similar. Once you have the drive you'll need to configure some
[OpenVpn](https://openvpn.net/) compatible client.

## Linux

On Linux the easiest setup is probably to use the [Network
Manager](https://wiki.archlinux.org/index.php/NetworkManager). That package provides several ways to configure a
connection. The `nm-connection-editor` command will bring up a GUI to let you edit a configuration and most desktop
provide tray applets to configure and connect to various networks, including VPNs. On Debian Buster an `apt-get install
network-manager-openvpn-gnome` was sufficient to install the OpenVpn extension to the network manager and update the
tray applets to enable quick, GUI connections to the VPN.

However, you must first set up the VPN connection. The easiest way to do this is probably to first copy and edit the
following `.ovpn` file:

```
# Change the following 4 paths to point to the appropriate files
ca '/home/oliver/Documents/MVPStudio/vpn/certs/ca.crt'
cert '/home/oliver/Documents/MVPStudio/vpn/certs/cert.crt'
key '/home/oliver/Documents/MVPStudio/vpn/certs/key.key'
tls-auth '/home/oliver/Documents/MVPStudio/vpn/certs/ta.key' 1
client
remote '50.201.1.196'
auth-user-pass
cipher AES-128-CBC
dev tun
dev-type tun
proto tcp
port 443
nobind
auth-nocache
auth SHA256
script-security 2
persist-key
persist-tun
user nm-openvpn
group nm-openvpn
```

You must edit the first few paths to point to the certificates and keys provided to you by Continu. Once that's done you
can import this into the Network Manager via:

```
nmcli connection import type openvpn file <your .ovpn file>
```

If you then open any Network Manager gui you should see your new VPN connection.  Since authorization to Continu is a
mix of keys/certificates and username/password you must then also provide your username and password. If using a GUI
like `nm-connection-editor` or a tray applet you simply open the configuration and add your username and password.

## OSX

[Tunnelblick](https://tunnelblick.net/) seems to work well on OSX.

TODO: can someone who's set up the VPN via Tunnelblick provide some details??
