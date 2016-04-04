# Discovery

Clients can discover the Box in three ways:

* [using a QR code to a mDNS host](https://github.com/fxbox/RFC/pull/4)
* [using a QR code to a public DNS host](https://github.com/fxbox/RFC/pull/5)
* [using nupnp](https://github.com/fxbox/RFC/pull/6)

The Box can be configured at build-time to:
* create a tunnel for remote access or not,
* create a public DNS host or not,
* announce itself via nupnp server or not.

All three actions involve the Box connecting to a given Bridge, hard-coded at build-time.

The Bridge is open-source software, and a manufacturer of the Box will probably want to host an instance of it.

It provides the following services:

* DNS server
* API for requesting DNS changes
* SNI-level tunnel (public interface)
* SNI-level tunnel (backend interface)
* nupnp (see https://github.com/fxbox/registration_server)

It is possible to use the Box without using an instance of the Bridge, but then remote (tunnel) access will not be possible,
and only the first of the three discovery methods (using mDNS) will work.
