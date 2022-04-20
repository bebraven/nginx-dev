# nginx-dev

## Overview
This repo is a nginx reverse proxy for the development environment. It allows
you to hit the local dev URLs without needing to put in the port and to setup
local HTTPS/SSL support. It's also a handy reference for the port numbers for each
dev application.

## SSL Support
Instead of adding support in each application and switching to boot it in SSL mode, the nginx reverse
proxy allows an easy way to run the app in HTTP mode but to be able to hit it in SSL mode. We've done
this for the [Braven Platform](https://github.com/bebraven/platform) app which runs locally
at http://platformweb:3020

### platformweb
To access https://platformweb on a Mac, simply:
1. Open up Keychain Access app
2. Click on Certificates on the left
3. Drag the `platformweb-selfsigned.crt` from the root of this repo into there
4. Double click it
5. Expand the Trust section
6. Change the setting to *"When using this certificate" : "Always Trust"*
7. Close the dialog and type your computer password to save.

After doing this, if you restart this container and hit https://platformweb (assuming you have an entry in `/etc/hosts` for platformweb), it should work with no warnings about the site being unsafe.

#### Original setup instructions for platformweb SSL support

In case you need to setup SSL support for another app in the future, here is how it
was setup for `platformweb`

1. We created a self-signed SSL cert using the following command (substitue platformweb for your
app name):
```
openssl req \
    -newkey rsa:2048 \
    -x509 \
    -nodes \
    -keyout platformweb-selfsigned.key \
    -new \
    -out platformweb-selfsigned.crt \
    -subj /CN=platformweb \
    -reqexts SAN \
    -extensions SAN \
    -config <(cat /System/Library/OpenSSL/openssl.cnf \
        <(printf '[SAN]\nsubjectAltName=DNS:platformweb')) \
    -sha256 \
    -days 3650
```

2. We checked the .crt and .key files into src control.

3. We updated docker-compose.yml to map the new .crt and .key into the docker image as a volume. See the platformweb example in there.

4. We did the Mac Keychain Access step above to let our local computers trust the self-signed .crt

5. Finally, we edited the `nginx.conf` to liston on port 443 and use the new self-signed key which you've told your local computer to trust. Just look at the platformweb config in there for an example

## Troubleshooting
Occasionally you'll get a `Bad Gateway nginx 1.17.4` error. Not sure why, but if you just restart the
service using `./docker-compose/scripts/restart.sh` it should work.


