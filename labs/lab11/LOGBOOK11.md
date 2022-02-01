# Public-Key Infrastructure (PKI) Lab

## Logbook Week 11

### Lab Environment Setup

```shell
# cd path/to/Labsetup
dcbuild
dcup

# spawn a new shell
docker ps -qa
docksh 06

# spawn a new shell
sudo nano /etc/hosts

# input the content below
10.9.0.80   www.bank32.com
10.9.0.80   www.m05g02.com
```

### Task 1: Becoming a Certificate Authority (CA)

In this task, we want to become a CA, so that we can issue a certificate for others. To achieve that we first need to copy the OpenSSL configuration file into our Labsetup folder.

```shell
# cd path/to/Labsetup
cp /usr/lib/ssl/openssl.cnf ./openssl.cnf
```

Next up, under `[CA Default]` inside the `openssl.cnf` file the following content has to be present

```env
dir = ./demoCA
certs = $dir/certs
crl_dir = $dir/crl
database = $dir/index.txt
unique_subject = no
new_certs_dir = $dir/newcerts
serial = $dir/serial
```

Then we need to create the `demoCA` folder and add the necessary files into it to match the specifications in the config file.

```shell
mkdir demoCA
cd demoCA
mkdir certs crl newcerts
touch index.txt
echo "1000" > serial
cd .. # return to Labsestup
```

After that we are going to start generating the self-signed CA certificate, using the command given so we are not prompted for additional information.

```shell
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -keyout ca.key -out ca.crt -subj "/CN=www.modelCA.com/O=Model CA LTD./C=US" -passout pass:dees
```

We are not prompted any further because the command above already contains the mandatory information according to the following content in our `openssl.cnf` file:

```env
# For the CA policy
[ policy_match ]
countryName = match
stateOrProvinceName = match
organizationName = match
organizationalUnitName = optional
commonName = supplied
emailAddress = optional
```

The commands below allow us to look at the decoded content of the X509 certificate and the RSA key as plain text

```shell
openssl x509 -in ca.crt -text -noout
openssl rsa  -in ca.key -text -noout
```

### Questions

**What part of the certificate indicates this is a CA’s certificate?**

CA:TRUE

![](https://i.imgur.com/fbXqF4E.png)

**What part of the certificate indicates this is a self-signed certificate?**

X509v3 Subject Key Identifier = X509v3 Authority Key Identifier

![](https://i.imgur.com/AtEDIgu.png)

**In the RSA algorithm, we have a public exponent, a private exponent `d`, a modulus `n`, and two secret numbers `p` and `q`, such that `n=pq`. Please identify the values for these elements in your certificate and key files.**

The logs show the values in the following order: n (modulus), public exponent (publicExponent), d (privateExponent), p (prime1), q (prime2).

```
RSA Private-Key: (4096 bit, 2 primes)
modulus:
    00:d6:5b:c5:aa:a6:21:26:d8:53:be:96:ee:2a:43:
    f8:25:be:ed:8e:76:bb:4e:77:24:63:5f:7b:d9:8c:
    b9:b9:38:b8:68:17:15:ba:a6:72:28:c1:be:75:94:
    2e:6e:e7:20:8e:ff:20:ce:7b:32:3a:e1:23:4e:58:
    34:d8:4d:82:33:75:8a:40:bd:c8:5a:aa:9f:5f:c4:
    d0:4c:4a:25:75:ca:97:31:1a:82:8c:59:ec:46:6e:
    ba:28:e1:31:33:36:a5:86:82:28:87:ed:a9:d9:3e:
    95:6f:7f:b3:0e:e1:35:63:86:53:d6:a9:c8:a4:5c:
    16:1b:76:7a:8d:91:bc:fb:d0:b5:5e:24:fd:b4:63:
    77:02:b6:0a:d2:38:02:b2:e0:7e:64:02:1c:a2:e9:
    ac:88:e7:83:94:c6:d6:9f:79:51:39:47:b7:70:93:
    01:ac:4f:e8:26:af:21:50:86:d3:ff:1a:9d:29:48:
    9b:de:e7:8d:03:d1:b1:60:b6:93:88:36:ca:c0:17:
    ca:bc:bd:c2:ff:22:81:02:20:19:f4:4c:83:b1:d5:
    b9:00:da:9b:92:5d:ba:a9:53:df:15:dd:c7:00:63:
    17:3b:31:c5:d6:1a:df:92:b5:a3:1c:8b:da:a0:4e:
    0d:42:bc:3e:62:b8:55:e0:89:df:82:4f:21:37:eb:
    0d:42:c0:f6:98:2f:6b:c6:c4:7f:d9:7e:06:18:e1:
    01:b3:b8:db:ca:a7:3a:b4:38:7f:cf:02:27:8b:7d:
    d1:6c:e8:10:14:37:6a:1e:45:1d:b1:00:ac:f5:e1:
    4b:4d:81:f2:19:75:d6:25:f6:31:46:c1:36:46:a6:
    86:43:3d:37:a6:b2:9f:23:aa:3a:6d:37:83:48:73:
    83:87:bc:a6:e7:39:9c:c9:3c:0f:ae:e8:1e:eb:72:
    83:47:03:52:22:86:b9:57:f0:7c:5f:28:9f:eb:08:
    92:9d:db:38:4c:df:2f:1a:b9:ff:b2:dd:57:8f:e9:
    86:7f:5c:e8:69:0d:a9:f6:16:77:94:ce:97:99:c0:
    99:45:de:f2:ab:98:65:44:e8:84:a0:aa:d9:9e:aa:
    15:f8:fc:2f:86:89:28:d3:a1:99:35:e1:61:30:f9:
    05:64:c5:74:b0:b8:45:4e:db:29:75:92:dc:d0:ad:
    2a:95:e1:83:25:2c:be:84:73:e0:2a:d1:4e:b6:88:
    18:42:6a:cf:64:2d:3e:90:61:ef:d6:f8:ac:c9:3d:
    d5:ea:53:61:1c:4f:2b:63:35:42:06:02:92:29:da:
    0b:35:c7:0f:b8:12:eb:a3:c9:70:92:78:a4:50:f4:
    0f:5c:57:b8:fd:6e:a7:4d:0c:c6:99:97:e5:70:f3:
    c2:ed:89
publicExponent: 65537 (0x10001)
privateExponent:
    00:94:25:cd:11:49:cb:f3:ba:e2:f5:ff:fe:0e:7b:
    f7:4e:af:0c:23:bf:ef:68:25:73:a2:b2:65:38:4f:
    c8:34:38:fd:4a:03:5a:63:2b:92:0e:95:08:7a:de:
    b4:d0:b5:30:8d:63:ca:5a:aa:4e:66:df:1e:b5:90:
    c4:c5:11:9c:80:d0:25:82:e5:27:49:72:4e:bf:b3:
    98:7a:81:6c:2e:62:9b:e7:b5:f8:af:e3:9e:26:77:
    74:75:b8:5d:76:95:b8:04:a4:84:3a:9d:89:1b:b9:
    e3:31:b2:42:20:70:89:a3:85:3d:00:49:4b:80:3c:
    9c:92:d2:69:94:da:3a:90:97:08:22:4e:d2:81:0f:
    95:3a:ec:71:c2:24:2f:c9:4c:da:4d:68:20:3e:7f:
    dd:5c:a9:15:09:87:fa:e1:30:c9:70:1b:1e:ae:d1:
    0b:00:fa:20:ea:4b:73:6c:e2:22:36:57:40:73:3d:
    d9:6c:4e:ff:e8:b9:ce:2b:97:43:93:8a:ba:c9:d4:
    27:ac:16:42:64:6e:86:56:df:b4:d0:60:e9:4b:c8:
    f8:19:9c:fc:94:45:ef:32:03:e3:54:8b:78:73:ce:
    08:4a:42:f6:06:29:80:87:36:bd:ca:86:e5:cc:90:
    e5:1d:50:58:95:85:ec:e4:48:a5:8e:bb:fd:ab:55:
    ea:4d:0a:9a:20:c6:07:b8:74:c5:36:13:18:bf:6e:
    bc:65:85:b9:25:53:89:57:d8:b1:19:f4:d7:90:c7:
    3c:86:2f:66:f5:0b:e3:70:8d:0a:e9:f3:68:6b:4c:
    38:3a:14:47:89:05:56:7f:65:a0:be:9c:8f:bb:10:
    1b:cc:df:9e:91:a3:d8:c3:fd:f8:0b:ad:fe:78:3c:
    75:6e:bf:fe:f6:96:93:c9:f0:bf:b4:de:69:45:37:
    d6:b1:e5:b2:10:f0:b4:ca:f5:84:ec:69:44:9c:bc:
    7b:7f:2c:92:60:b9:1a:23:43:eb:89:cf:d8:5d:fa:
    12:e6:c5:cc:8e:27:a2:35:59:f7:dd:fb:e5:02:aa:
    a5:82:ad:6a:4d:43:d4:3b:1a:4e:04:ad:da:12:2a:
    b0:f1:9e:19:65:13:26:69:fb:ef:b5:ce:98:d7:f1:
    0e:12:68:07:dc:bd:27:74:19:10:c9:c3:54:67:3c:
    dc:74:2e:4d:e8:2a:5e:26:b2:c3:07:89:44:09:14:
    2e:ef:95:33:4e:f8:2c:c9:a0:ec:e5:bb:b8:fe:80:
    cc:10:f9:16:c4:30:b5:03:27:1e:c5:0f:3c:2d:40:
    90:aa:56:39:05:eb:99:f4:e6:0c:89:64:7a:f4:22:
    28:ad:9b:a5:17:51:e7:48:82:ac:8a:ac:5c:cf:58:
    a6:60:01
prime1:
    00:ec:9c:e8:78:57:f9:0c:77:a9:e3:48:fa:76:5b:
    fe:7b:79:0e:a4:38:d1:bc:69:4e:12:ff:17:ad:fe:
    93:cd:24:5c:3a:43:ba:64:c2:df:a5:40:d9:cb:c2:
    68:fe:fb:e2:87:4a:6e:e8:83:09:44:78:8b:8e:4f:
    f2:5e:bc:a0:67:1c:30:9e:ef:de:67:d3:15:69:55:
    39:b2:4c:f4:62:e1:eb:0e:50:1b:e5:ed:0f:47:07:
    4b:16:2c:95:f4:7e:7d:6e:0f:7b:53:be:d7:da:c1:
    fb:25:ba:6c:3c:4e:2e:e5:ef:6b:36:5b:33:d4:f7:
    2e:96:71:27:38:24:09:af:60:ce:12:5b:35:ce:9c:
    98:4f:62:d1:d0:6d:39:cc:69:23:d8:76:7b:e3:bf:
    16:1e:23:47:1a:f7:34:ff:2a:50:57:be:ca:15:49:
    62:4c:92:cf:a9:83:fe:3d:87:9a:99:5e:34:34:2f:
    22:9e:60:2a:ce:10:b1:a5:ea:4a:71:a2:28:35:df:
    0a:84:92:26:46:6e:57:ff:1c:6e:cd:2a:f1:75:ad:
    83:e5:f1:18:78:4b:32:ad:49:3d:0e:a9:2c:6c:1f:
    67:3e:59:bd:a2:00:9f:13:fd:1c:05:75:29:90:96:
    f8:d9:d7:81:00:ef:14:6f:7b:75:12:7a:81:c5:87:
    c3:89
prime2:
    00:e7:ec:10:7f:9b:30:1d:e4:be:4d:af:c1:2c:65:
    c5:81:65:ca:b6:dc:3f:01:d4:a1:cb:87:de:f7:ef:
    13:c5:df:f8:ad:2c:04:5b:01:60:22:9b:4f:97:ad:
    f4:dd:90:9e:6c:f5:d5:07:de:9e:a0:35:fe:d4:1f:
    ae:0c:f4:43:6c:9b:6c:5d:6f:c8:9e:8c:9d:02:42:
    3a:11:d3:2b:28:54:b3:38:82:fc:c4:76:97:0a:43:
    37:9e:c3:f5:6e:f0:c7:e3:e0:8d:58:19:c6:7f:8d:
    66:da:b2:09:ac:24:98:81:be:6d:ff:aa:00:13:3e:
    5d:03:71:69:cd:be:75:ca:23:6c:f8:e0:37:0c:4a:
    4e:4c:20:e3:35:36:80:ed:76:06:c6:92:06:40:2a:
    b5:6e:56:33:e5:b9:73:d8:df:e7:46:d5:f7:a5:f2:
    73:d0:ea:71:72:bf:a2:85:64:41:6e:c5:98:bf:17:
    46:4d:60:a3:65:58:b5:ce:07:a2:1c:2b:5f:6b:82:
    a3:80:2b:c9:9e:fd:55:d8:7f:27:b0:58:c9:27:e0:
    50:f7:53:06:45:ce:40:43:f4:df:6f:ae:5d:73:b8:
    72:ac:5d:f7:11:a9:1a:35:75:ba:04:e0:e7:95:39:
    be:5c:b6:0d:08:ba:18:63:1e:19:a7:29:b0:8f:55:
    5a:01
```

### Task 2: Generating a Certificate Request for Your Web Server

The following command is used to generate a request using the new server key.

```shell
openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.m05g02.com/O=M05g02 Inc./C=US" -addext "subjectAltName = DNS:www.m05g02.com, DNS:www.m05g02.net, DNS:www.m05g02.pt" -passout pass:dees
```

The `-addext` flag uses the Subject Alternative Name (SAN) extension to connect different urls, that are pointing to the same web server, to the same certificate.

```shell
openssl req -in server.csr -text -noout
```

![](https://i.imgur.com/UPqFfPe.png)

### Task 3: Generating a Certificate for your server

To form a certificate, the CSR file needs to have the CA's signature, so we'll use the one we defined in **Task 1**. (openssl.cnf)

```shell
openssl ca -config openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```

![](https://i.imgur.com/ql90DBe.png)

For security reasons, the default setting in openssl.cnf doesn't allow the `openssl ca` command to copy the extension field from the request to the final certificate.

For this task, we need to enable it. We can achieve that by uncommenting the following line in our copy of the openssl.cnf file:

```env
copy_extensions = copy
```

The command below shows that the aliased URLs were added:

```shell
openssl x509 -in server.crt -text -noout
```

![](https://i.imgur.com/TKZgJUi.png)

### Task 4: Deploying Certificate in an Apache-based HTTPS Website

For this task we explain our approach in the steps below:

1. Create a new directory `task4` inside `Labsetup`.
2. Inside it, there is a `Dockerfile`, the `m05g02_apache_ssl.cnf` file, the two `index.html` previously provided files (_index_ and _index_red_) and a folder named `certs`.
3. The certs folder has the certificates `ca.crt`, `server.crt` and `server.key` files.
4. The dockerfile content is as seen below

```dockerfile
FROM handsonsecurity/seed-server:apache-php

ARG WWWDIR=/var/www/m05g02

COPY ./index.html ./index_red.html $WWWDIR/
COPY ./m05g02_apache_ssl.conf /etc/apache2/sites-available
COPY ./certs/server.crt ./certs/server.key /certs/

RUN chmod 400 /certs/server.key \
    && chmod 644 $WWWDIR/index.html \
    && chmod 644 $WWWDIR/index_red.html \
    && a2ensite m05g02_apache_ssl

CMD tail -f /dev/null
```

5. The m05g02_apache_ssl.cnf file contains the text below

```xml
<VirtualHost *:443>
    DocumentRoot /var/www/m05g02
    ServerName www.m05g02.com
    ServerAlias www.m05g02.pt
    ServerAlias www.m05g02.org
    DirectoryIndex index.html
    SSLEngine On
    SSLCertificateFile /certs/server.crt
    SSLCertificateKeyFile /certs/server.key
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot /var/www/m05g02
    ServerName www.m05g02.com
    DirectoryIndex index_red.html
</VirtualHost>
```

6. We changed the docker-compose to have `build: ./task4`
7. Now we can run the container inside the `task4` folder and access it using `dockps` and then `docksh <id>`
8. Inside the container we can run `service apache start`, for example, and insert the password (`dees`)

![](https://i.imgur.com/L3AqCR0.png)

9. Entering https://www.m05g02.com on firefox will give us a security warning

![](https://i.imgur.com/PWDEc8B.png)

10. This happens because Firefox only has highly secure certificates in its bank of certificates, which means a self-signed certificate is a reason to launch a warning in a modern browser.

11. To avoid this warning we can either:
    - Create an exception in the browser and allow the domain to be accessed
    - Manage to tell the browser to trust our CA

In the image below we will allow the browser to create an exception, and proceed to the website, where we accept the risk and proceed.

![](https://i.imgur.com/xtV6NiZ.png)

---

The alternative is to tell the browser that it should trust our CA which will safely connect us to our domain without any warnings. In firefox we can do that by going to [about:preferences#privacy](about:preferences#privacy) and looking for the certificates tab where we can let the browser trust our CA.

![](https://i.imgur.com/1Veydyi.png)

![](https://i.imgur.com/DoIj5rh.png)

![](https://i.imgur.com/7IHEmHq.png)

### Task 5: Launching a Man-In-The-Middle Attack

Tasks 5 focus on showing how public-key encryption based on Certification Authority signature is capable to handle MITM attacks as long as the certification keys are not compromised. Task 5 compresses two steps. In the first one, it is intended to set up a website we want to try to intercept communication. In the second step, the attacker is required to change the routing table of the victim's host machine to divert the HTTP request towards the malicious site set up in the first step.

If the public key certification wasn't efficient to detect MITM, it would be possible for Alice to get a valid HTTPS validated connection with the exploiter and the browser certification management wouldn't complain about the potential trap Alice was being a victim of.

For this task, we will try to fool Alice when she accesses the domain, www.slbenfica.com. We will repeat the process done in previous tasks and generate a new apache configuration in the Docker container, namely a new Virtual Host with a server name www.slbenfica.com.

The second step is focused on the victims. Somehow the attacker should be able to change the DNS routing table.  
By inserting in /etc/hosts:
**10.9.0.80 www.slbenfica.com**
Alice browser will resolve the IP 10.9.0.80 towards

Even with the DNS route exploitation, when Alice visits www.slbenfica.com, the browser will display the following warning:

![](https://i.imgur.com/l1oOT5C.png)

This happens because the certificate that we are using to sign www.slbenfica.com was emitted for www.m05g02.com. Modern websites can detect this kind of mismatches in certificates and alert Alice.

If we assume the risk and continue. We can access the malicious clone of www.slbenfica.com. But we were warned.

![](https://i.imgur.com/TzQeYmN.png)

### Task 6: Launching a Man-In-The-Middle Attack with a Compromised CA

In Task 5 it was clear that Public Key certificates, even the self-signed implement the necessary countermeasures to detect mismatches that an attacker can't overthrow while performing a man in the middle attack.

Nevertheless MITM still happen, meaning they are still possible. Task 6 is intended to perform one. For that, it is necessary to assume a strong burden. **The private key used by the CA to sign a request is stolen**.

The intent is to show that a private key is a single point of failure, whose failure is impossible to detect and therefore is a clear sign why CA infrastructure must be highly protected against attacks of all sorts.

Before describing how it is possible to effectively perform a MITM to Alice when she visits slbenfica.com, **we must remember that Alice trusted previously in the CA that will see their keys stolen**

Our intuition is the following:
With the CA keys stolen, we can issue valid certificates.
After issuing them we need to install them on the apache server
Alice will not notice it is being fooled when she accesses slbenfica.com, because it previously trusted that CA

To perform this attack in isolation, a simulation of it is being done in the attacker's computer, we created a copy of the CA keys and added them to the docker container.

![](https://i.imgur.com/xFt7Ggw.png)

Then we repeated the commands to turn docker a CA (Task 1) and generated a certificate request/reply (Tasks 2 & 3) but this time, specifying slbenfica.com as the entity requiring certification (command bellow)

```shell
openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.slbenfica.com/O=Benfica Inc./C=US" -passout pass:dees
```

The result of this series of commands was the issue of the certificate and a server public key that we then injected in the new entry in the apache2 server, after an apache2 restart the result in the Alice browser is that we are secured, Alice is now suffering a MITM attack.

![](https://i.imgur.com/yNVLTyH.png)
