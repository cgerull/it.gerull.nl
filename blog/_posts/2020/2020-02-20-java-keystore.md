---
layout: post
title: Java keystore
description: "Get and add a root certificate to the java keystore."
tag: Java SSL
category: DevOps 
date: 2020-01-10 14:13:17
---


## Get Certificate with openssl

The easiest way to get the certificate is with openssl.

```bash
echo -n | \
openssl s_client -connect www.private.local:443 | \
sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > private-local.crt

```

Alternative you can use your Firefox or Chrome web browser to get the certificate. Click on the lock
in the URL and follow the browser specific instruction to save the certificate.


## Import the certificate into the keystore

Run the following command from the CLI.

```bash
keytool -import -trustcacerts -keystore $JAVA_HOME/jre/lib/security/cacerts \
   -storepass changeit -noprompt -alias private-local -file private-local.crt
```

## Tools

On Windows you can find an openssl installer packages on [openssl.org](https://wiki.openssl.org/index.php/Binaries).
But if you already installed `git` it's accessible via your `git-bash` shell. Same goes for `MobaXterm`, here you can access it
from the local terminal.

On Linux and MacOS it should be already installed.

