# Box-client discovery: two options

## Option 1: --no-cloud

When the Foxbox is built with the --no-cloud flag, discovery works as follows:

* Connect the box to the router using an ethernet cable. The router can just be a local switch,
it does not need to be connected to the internet.
* The box announces its hostname, which has a distinguishable format, e.g. foxbox<i>.local where <i> is a number
* The client tries to connect to https://foxbox<i>.local/ and accepts the cert on first use

Note that this requires the client to support both mDNS and self-signed certificates, so it has to be a native/Cordova app.

## Option 2: default build

When the Foxbox is built without the --no-cloud flag, discovery works as described in https://github.com/fxbox/RFC/pull/6/files.
