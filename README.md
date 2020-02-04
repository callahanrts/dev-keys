# Creating Development Certs

## Certificate Authority

You can create your own certificate authority and trust it. Doing this will
ensure that all certs signed by this authority will be trusted by your local
computer automatically.

To create a new certificate authority:

```bash
# Create a new CA
./create authority

# Enter a password when prompted. This will be used when
# signing new certificates
```

After creating your certificate authority, you will need to open keychain.app
and import the rootCA.pem file. After imported, go to the settings and select
`Always Trust`. This will ensure that all keys created by this CA will be
trusted by your computer.

**NOTE**: You will only have to create a CA once. After that, all keys will
use the same CA.

## Creating Certificates

The first step to creating a new certificate is to create a couple files in the
`config` folder.

```conf
# config/your-domain.local.csr.cnf

[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn

# Update the following block with your information
[dn]
C=US
ST=CA
L=SanFancisco
O=BigCo
OU=Engineering
emailAddress=noreply@my-domain.com
CN = your-domain.local
```

```conf
# config/your-domain.local.ext

authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
DNS.2 = your-domain.local
```

The next step is to create the certificate for your domain.

```bash
# This will create the certificate
# Make sure that your-domain.local matches the naming scheme of the files you
# created in the config/ folder
./create certificate your-domain.local
```

# How to Use Created Certificates

## Caddy

```caddy
your-domain.local:443 {
  tls /path/to/certs/your-domain.local.crt /path/to/certs/your-domain.local.key

  proxy / localhost:3000 {
    transparent
  }
}
```

## Nginx

```nginx
upstream server {
  server localhost:3000;
}

server {
  listen    443 ssl;
  server_name your-domain.local;

  ssl_certificate /path/to/certs/your-domain.local.crt;
  ssl_certificate_key /path/to/certs/your-domain.local.key;

  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://server;
    break;
  }
}
```
