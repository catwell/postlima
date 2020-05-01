# Lima root FAQ

## Why do I need a Lima Ultra [updated after June 2017](howto-root-ultra.md#prerequisites)?

The Lima firmware was updated over the air, and firmware updates were cryptographically signed. A firmware update without a valid signature would not run. Only a handful of people held private keys that could sign firmwares; they had to correspond to public keys stored on the device.

The `.pkg` file in this repository is technically similar to a firmware update, except it is run through an alternative channel using a USB drive instead of an update server. It still has to be signed, so I signed it with my own private key. However, my public key was only added to the Lima by *another* firmware update shipped around June 2017. Lima devices that came out of the factory only included a single public key, and only the corresponding private key could be used to sign the initial firmware update they applied.

I do not have that private key in my possession, and this is why you need a Lima Ultra updated after June 2017 to root your Lima device.

## Can I root a Lima Original device?

Not using this repository. There may be ways to do it that involve soldering and so on, but Lima Original did not have the alternative "update over USB" feature.

It would be a lot of work for little benefit though. The Lima Original hardware has a 133 MHz CPU, 32 MB of RAM and 8 MB of Flash. There isn't much useful software you can run satisfactorily on a device like this. Moreover the CPU has an exotic architecture and you need a complicated toolchain to compile software for it; you cannot easily run things like Alpine with precompiled packages.
