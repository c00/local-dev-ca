# Certificate Authority for local development

The purpose is to be able to create certs that are accepted as legitimate by the system for development purposes.

The structure is as such:

1. Create a certificate authority (CA) that allows only for the creation of certificates for names ending in `.local`.
2. Add the CA to the trusted store on the system.
3. Create certificates for local resources.

Don't forget to add the domain name to your local hosts file (`/etc/hosts` on linux).

Not all browsers automatically pick up the new certificates. e.g. on Zen browser I had to import the cert manually into the browser.

## Create CA

If you haven't created your local CA yet, do the following:

```bash
# Generate private key
openssl genrsa -out localdevca.key 4096

# Generate the certificate
openssl req -x509 -new -nodes -key localdevca.key -sha256 -days 3650 -out localdevca.crt -config ca_config.cnf

# Check details of certificate
openssl x509 -in localdevca.crt -text -noout

# Add it to your local trust store
sudo cp localdevca.crt /etc/pki/ca-trust/source/anchors/

# Update the trust store
sudo update-ca-trust extract

# Verify it works
trust list | grep "Wacky Waiving Inflatable Arm Flailing Dev Certificate Authority"
```

## Create Certificate for your service

```bash
# Generate Private key, CSR and Certificate
openssl req -newkey rsa:2048 -nodes -keyout keycloak.local.key -out keycloak.local.csr -config cert_config.cnf && openssl x509 -req -in keycloak.local.csr -CA localdevca.crt -CAkey localdevca.key -CAcreateserial -out keycloak.local.crt -days 365 -sha256 -extfile cert_config.cnf -extensions v3_req

```
