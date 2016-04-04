

# Discovery, local and secure:
## QR + mDNS + HSS

  - pro: privacy (not leaking user data to the cloud)
  - pro: security (you only need to trust the Box and the client)
  - con: requires Cordova
  - con: scanning a QR code is cumbersome on laptops and TVs
  
## Flow:

1. At build time, a self-signed cert is generated, using the method from https://github.com/coolaj86/nodejs-self-signed-certificate-example#but-wheres-the-magic.
2. Still at build time, the fingerprint of the signing cert (not the https server's own TLS cert) is determined, and put into a string of the form "https://<fingerprint>.self-signed". This string is put into a QR code and both the QR code and the string are printed onto a sticker.
3. The https server's TLS cert (public + private key) and the signing cert (public key only) are put into the Box's https server.
4. The sticker is put onto the physical Box. The Box is then packaged in such a way that you can't see the QR code sticker in the shop without opening the packaging.
5. The user unpacks the Box, and plugs it into their router using an ethernet cable.
6. The Box obtains an IP address via DHCP and announces its "https://<fingerprint>.self-signed" service over mDNS, pointing to its local IP address.
7. The user installs either a browser or an app onto their client, that supports:
 * reading a QR code,
 * resolving the <fingerprint>.self-signed domain name to a local IP address using mDNS,
 * connecting to https://<fingerprint>.self-signed/ and checking the fingerprint of the https server's signing cert against the <fingerprint> part of the URL.
8. If the fingerprint matches, the app requests the creation of a user on the Box (this could be either a html interface or a rest API call, UX for that to be determined, but accessible only over the local network),
9. User creation:
 * If the Box is still in setup mode, it accepts this request and gives the app a token with admin access,
 * If an admin user already exists, this admin user is asked whether they want to allow the new user creation request.
10. If the Box gets assigned a new local IP address through DHCP, it announces this new IP address using mDNS.
11. When selecting this discovery mechanism at build-time, it is not possible to use the tunnel for remote access.

