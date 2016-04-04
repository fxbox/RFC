
# Discovery using the cloud and leap-of-faith
## NUPNP + plex + LE
 * (less-secure option when QR codes cannot be used)
 * con: you need to trust not only the Box and the client, but also the bridge/DNS-server
 * pro: the user does not have to click the (confusing) "Allow" button in the getUserMedia dialog
 * con: lack of QR-code means trusting that no attacker is on local network during pairing phase

## Flow:
1. At build time, a serial number and a deploy token is put onto the box, so that it can request two services:
 * setting up a DNS entry,
 * signing a CSR.
2. Still at build time, the serial number is put into a string of the form "https://<serial-no>.knilxof-local.org/". This string is printed onto a sticker.
3. The sticker is put onto the physical Box. The Box is then packaged in such a way that you can't see the serial number sticker in the shop without opening the packaging.
4. The user unpacks the Box, and plugs it into their router using an ethernet cable.
5. The Box obtains a local IP address using DHCP, and generates two CSRs.
6. The Box connects to the Bridge on e.g. https://register.knilfox.org and posts the following data:
 * its serial number,
 * the deploy token,
 * its CSR,
 * its local IP address.
7. The Bridge registers two LetsEncrypt certs (one for <serial-no>.knilxof-local.org and one for <serial-no>.knilxof-remote.org), telling the Bridge to update DNS with the ACME DNS challenge strings (currently in staging, see https://community.letsencrypt.org/t/dns-challenge-is-in-staging/8322 ).
8. The Bridge adds an A record for <serial-no>.knilfox-local.org, pointing to the Box's local IP address; <serial-no>.knilxof-remote.org points to the public end of the tunnel.
9. The user can use the normal browser on their client to open our web app.
10. The Box posts its long URL "https://<serial-no>.knilxof-local.org/" to https://discover.knilxof.org/I-am-a-Box.
11. The Bridge adds an entry to its database linking the outgoing IP address of the Box to its long URL.
12. The user types https://discover.knilxof.org/I-am-a-client.html into their browser's address bar, and the Bridge does a database lookup;
 * If there is exactly one Box registered from that same outgoing IP address, the user is asked to check that the <serial-no> matches the one on the sticker, and if they click 'Yes', they are redirected to https://<serial-no>.knilxof-local.org/
 * If there are no Boxes registered for that same outgoing IP address, an error is displayed.
 * If there are  >1 Boxes registered for that same outgoing IP address, the user is offered a choice, and told to select the serial-no that matches the one on the sticker on their Box, after which the user is redirected to https://<serial-no>.knilxof.org/ for the serial-no they selected.
13. The Box serves a page or cross-origin API (accessible only on its local IP address) that requests the creation of a user on the Box.
14. User creation:
 * If the Box is still in setup mode, it accepts this request and gives the app a token with admin access,
 *  If an admin user already exists, this admin user is asked whether they want to allow the new user creation request.
15. If the Box gets assigned a new local IP address through DHCP, it posts this new IP address to the Bridge and we wait for DNS propagation.
16. When the user leaves the local WiFi range, the app automatically switches from <serial-no>.knilfox-local.org to <serial-no>.knilfox-remote.org if activated and allowed (see also https://github.com/fxbox/RFC/pull/2).


