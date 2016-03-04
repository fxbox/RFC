
# Discover using the cloud, but otherwise still secure
## QR + plex + LE
  * (box will temporarily set up the bridge and update DNS during LE reg/renew)
  * con: you need to trust not only the Box and the client, but also the bridge/DNS-server
  * pro: no need for Cordova; just use a standard QR-code reader and mobile browser
  * con: scanning a QR code is cumbersome on laptops and TVs

## Flow:
1. At build time, a serial number and a deploy token is put onto the box, so that it can request two services:
 * setting up a DNS entry,
 * signing a CSR.
2. Still at build time, the serial number is put into a string of the form "https://<serial-no>.knilxof-local.org/". This string is put into a QR code and both the QR code and the string are printed onto a sticker.
3. The sticker is put onto the physical Box. The Box is then packaged in such a way that you can't see the QR code sticker in the shop without opening the packaging.
4. The user unpacks the Box, and plugs it into their router using an ethernet cable.
5. The Box obtains a local IP address using DHCP, and generates a CSR.
6. The Box connects to the Bridge on e.g. https://register.knilfox.org and posts the following data:
 * its serial number,
 * the deploy token,
 * its CSR,
 * its local IP address.
7. The Box registers two LetsEncrypt certs (one for <serial-no>.knilfox-local.org and one for <serial-no>.knilfox-remote.org), telling the Bridge to update DNS with the ACME DNS challenge for each (currently in staging, see https://community.letsencrypt.org/t/dns-challenge-is-in-staging/8322 ).
8. The Bridge adds an A record for <serial-no>.knilfox-local.org, pointing to the Box's local IP address; <serial-no>.knilfox-remote.org points to the public end of the tunnel on the Bridge.
9. The user opens the client app with their normal browser, and is prompted to allow getUserMedia for the QR code scan.
10. The user either scans the QR code on the Box's sticker, or types the <serial-no> into the client app.
11. The Box serves a html interface or cross-origin API end-point (accessible only on its local IP address) that allows the creation of a user on the Box.
12. User creation:
 * If the Box is still in setup mode, it accepts this request and gives the app a token with admin access,
 * If an admin user already exists, this admin user is asked whether they want to allow the new user creation request.
13. If the Box gets assigned a new local IP address through DHCP, it posts this new IP address to the Bridge and we wait for DNS propagation.
14. When the user leaves the local WiFi range, the app automatically switches from <serial-no>.knilfox-local.org to <serial-no>.knilfox-remote.org if activated and allowed (see also https://github.com/fxbox/RFC/pull/2).
