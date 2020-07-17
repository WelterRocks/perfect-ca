# Perfect-CA

Copyright (c) 2018 - 2020 Oliver Welter <oliver@welter.rocks>

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

--

## How to setup

After downloading and login as root, copy the file named *pca* to /usr/bin and chmod it to 755.
Than, create a pki store, copy your *pcarc* file to it and edit *pcarc* to your needs.

 cp pca /usr/bin
 chmod 755 /usr/bin/pca
 mkdir -p /etc/pki
 cp etc-samples/pki/pcarc /etc/pki

Because *joe* is my favorite editor, I will use joe in my examples. Feel
free to use nano, vi, or something else that fit your needs.

 joe /etc/pki/pcarc

After you set /etc/pki/pcarc (for extended validation to work, do not
change the suggested encryption settings to higher values) to your needed
values, you can start the Perfect-CA setup routine. Be sure /etc/pki/ca is
unused and empty for this example, because pca will probably overwrite
things in there. To setup Perfect-CA, simply do

 /usr/bin/pca --setup

and wait for the command to finish. If anything went wrong, the pca tool
will let you know and you first have to fix this. If things went well, you
can backup your newly created Offline-Root-CA by copying /etc/pki/ca/root-ca/private/root-ca.key
to a secure location and execute shred (for example) on the origin file and
delete it securly. If very high security is not your topic or
Offline-Root-CA is not needed, you can leave the key, where it is.

In any case, the pca tool will tell you, what to do next. And next, you
should create OCSP signing certificates and your online responder services.

 pca --create-certificate --skip-pkcs12 --ocspsign --root-ca --filename ocsp-root-ca
 pca --create-ocsp-systemd-service --root-ca > /lib/systemd/system/pca-root-ca-ocsp.service
 
 pca --create-certificate --skip-pkcs12 --ocspsign --master-ca --filename ocsp-master-ca
 pca --create-ocsp-systemd-service --master-ca > /lib/systemd/system/pca-master-ca-ocsp.service
 
 pca --create-certificate --skip-pkcs12 --ocspsign --network-ca --filename ocsp-network-ca
 pca --create-ocsp-systemd-service --network-ca > /lib/systemd/system/pca-network-ca-ocsp.service
 
 pca --create-certificate --skip-pkcs12 --ocspsign --component-ca --filename ocsp-component-ca
 pca --create-ocsp-systemd-service --component-ca > /lib/systemd/system/pca-component-ca-ocsp.service
 
 pca --create-certificate --skip-pkcs12 --ocspsign --identity-ca --filename ocsp-identity-ca
 pca --create-ocsp-systemd-service --identity-ca > /lib/systemd/system/pca-identity-ca-ocsp.service
 
After all, reload your systemd:

 systemctl daemon-reload

After daemon reload is done, you can enable and start your online-responders, with:

 systemctl enable pca-root-ca-ocsp
 systemctl start pca-root-ca-ocsp
 systemctl enable pca-master-ca-ocsp
 systemctl start pca-master-ca-ocsp
 systemctl enable pca-network-ca-ocsp
 systemctl start pca-network-ca-ocsp
 systemctl enable pca-component-ca-ocsp
 systemctl start pca-component-ca-ocsp
 systemctl enable pca-identity-ca-ocsp
 systemctl start pca-identity-ca-ocsp

Than, create and install the CA maintainance crontab, using this command:

 pca --create-crontab > /etc/cron.d/perfect-ca

At least, you can create your first EV certificate, using this command
(change your-hostname to your systems FQDN):

 pca --create-certificate --ev --export-san "DNS:your-hostname" --filename your-hostname

Or without extended validation (EV) and without generating an additional pkcs12 file
in place:

 pca --create-certificate --server --skip-pkcs12 --filename your-hostname

And if you want to create your first identity certificate (e.g. to access
TLS-Client-Cert protected websites or your XMPP host):

 pca --create-certificate --identity --filename your-name

If you want to check the validity of your newly generated certificate, do:

 pca --validate --filename your-hostname.pem

You can also validate the certificate using openssl command. This is nearly
the same check, except that openssl is using the system certificates to
verify the validity of your given certificate. Also you can test, if your
system certificate store has successfully hashed your new Perfect-CA.

 openssl verify /etc/pki/ca/certs/your-hostname.pem

If the result is *not OK*, you should rehash your system certificate store,
using

 c_rehash

and try the above openssl command again. After rehashing (what pca cronjob
should do from time to time), your certificates should by verified as valid
on your Perfect-CA host.

Your /etc/pki path is your new Perfect-CA store, which is backed up and
encrypted to /var/lib/backup by default. See your perfect-ca crontab for
details on how this works and your pcarc file for the key and backup
rotation time. By default, backups are held for 7 days. It is highly
recommended, that you change the backup key in your pcarc file. If you have
installed makepasswd utility, you can create a secure key, using this:

 makepasswd --chars=32

Your Multi-Level-Offline-Root-PKI should be ready to work at this point.

--

## TODO

Help and samples.


 