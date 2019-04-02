sa-jumpbox-sslh
===============

[![Build Status](https://travis-ci.com/softasap/sa-jumpbox-sslh.svg?branch=master)](https://travis-ci.com/softasap/sa-jumpbox-sslh)

SSLH – A SSL/SSH MULTIPLEXER
sslh accepts connections on specified ports, and forwards them further based on tests performed on the first data packet sent by the remote client.

Probes for HTTP, TLS/SSL (including SNI and ALPN), SSH, OpenVPN, tinc, XMPP are implemented, and any other protocol that can be tested using a regular expression, can be recognised. A typical use case is to allow serving several services on port 443 (e.g. to connect to SSH from inside a corporate firewall, which almost never block port 443) while still serving HTTPS on that port.

Hence sslh acts as a protocol demultiplexer, or a switchboard. With the SNI and ALPN probe, it makes a good front-end to a virtual host farm hosted behind a single IP address.

sslh has the bells and whistles expected from a mature daemon: privilege and capabilities dropping, inetd support, systemd support, transparent proxying, chroot, logging, IPv4 and IPv6, a fork-based and a select-based model, and more.


Simple

```YAML
  roles:
    - {
        role: "sa-jumpbox-sslh"
      }
```

Advanced:

```YAML
  roles:
    - {
        role: "sa-jumpbox-sslh"
      }
```

Portions of notes from  https://www.rutschle.net/tech/sslh/README.html

CONFIGURATION
If you use the scripts provided, sslh will get its configuration from /etc/sslh.cfg. Please refer to example.cfg for an overview of all the settings.

A good scheme is to use the external name of the machine in listen, and bind httpd to localhost:443 (instead of all binding to all interfaces): that way, HTTPS connections coming from inside your network don’t need to go through sslh, and sslh is only there as a frontal for connections coming from the internet.

Note that ‘external name’ in this context refers to the actual IP address of the machine as seen from your network, i.e. that that is not 127.0.0.1 in the output of ifconfig(8).



OPENVPN SUPPORT
OpenVPN clients connecting to OpenVPN running with -port-share reportedly take more than one second between the time the TCP connexion is established and the time they send the first data packet. This results in sslh with default settings timing out and assuming an SSH connexion. To support OpenVPN connexions reliably, it is necessary to increase sslh’s timeout to 5 seconds.

Instead of using OpenVPN’s port sharing, it is more reliable to use sslh’s --openvpn option to get sslh to do the port sharing.


USING PROXYTUNNEL WITH SSLH
If you are connecting through a proxy that checks that the outgoing connection really is SSL and rejects SSH, you can encapsulate all your traffic in SSL using proxytunnel (this should work with corkscrew as well). On the server side you receive the traffic with stunnel to decapsulate SSL, then pipe through sslh to switch HTTP on one side and SSL on the other.

In that case, you end up with something like this:

ssh -> proxytunnel -e ----[ssh/ssl]---> stunnel ---[ssh]---> sslh --> sshd
Web browser -------------[http/ssl]---> stunnel ---[http]--> sslh --> httpd
Configuration goes like this on the server side, using stunnel3:

stunnel -f -p mycert.pem  -d thelonious:443 -l /usr/local/sbin/sslh -- \
	sslh -i  --http localhost:80 --ssh localhost:22
stunnel options:
-f for foreground/debugging
-p for specifying the key and certificate
-d for specifying which interface and port we’re listening to for incoming connexions
-l summons sslh in inetd mode.
sslh options:
-i for inetd mode
--http to forward HTTP connexions to port 80, and SSH connexions to port 22.





Copyright and license
---------------------

Code is dual licensed under the [BSD 3 clause] (https://opensource.org/licenses/BSD-3-Clause) and the [MIT License] (http://opensource.org/licenses/MIT). Choose the one that suits you best.

Reach us:

Subscribe for roles updates at [FB] (https://www.facebook.com/SoftAsap/)

Join gitter discussion channel at [Gitter](https://gitter.im/softasap)

Discover other roles at  http://www.softasap.com/roles/registry_generated.html

visit our blog at http://www.softasap.com/blog/archive.html
