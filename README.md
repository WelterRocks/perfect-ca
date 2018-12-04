# Perfect-CA

Copyright (c) 2018 Oliver Welter <oliver@welter.rocks>

This program comes with ABSOLUTELY NO WARRANTY, licensed under GPLv3.
This is free software, and you are welcome to redistribute it
under certain conditions; see 'LICENSE' for details.

OpenSSL based tool to build and manage the "perfect" certificate authority. It creates and manages an expert multi level public key infrastructure in seconds, without the need of expert knowledge. It comes with an offline root certificate authority, online responder (OCSP) support, easy certificate revocation list (CRL) management and automatic updates. CRL management and CA expiration prevention, can be easily automated by cron jobs. With Perfect-CA it is possible to manage your own complex PKI, without beeing a pro.

Perfect-CA hirarchie:

* OFFLINE ROOT-CA
* MASTER-CA
* NETWORK-CA
* ++ IDENTITY-CA
* ++ COMPONENT-CA

How it works:
Server, Client, Timestamp & OCSP-Signing certificates are signed by Component-CA.
Identity & Encryption certificates are signed by Identity-CA.
Identity-CA & Component-CA are signed by Network-CA.
Network-CA is signed by Master-CA.
Master-CA is signed by Offline Root-CA.
