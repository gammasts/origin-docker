# Generating a self signed CA for MITM caching

**Hint!** There's a script in this folder called `generate-tls-certificate-authority.sh`
which will make these files for you. :)

EG, for use with Epic Games if you own all of the PCs and you can install a Certificate Authority (CA) on it.

```
# Generate your CA key and then certificate
openssl genrsa -out rootca.key 2048

# Just press enter to all of the questions to accept defaults
openssl req -x509 -new -nodes -key rootca.key -sha256 -days 10000 -out rootca.pem

# Generate your server private key
openssl genrsa -out client.key 2048

# Server CSR
openssl req -new -key client.key -out client.csr -subj "/C=US/ST=CA/O=Acme, Inc./CN=download.epicgames.com" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:download1.epicgames.com,DNS:download2.epicgames.com,DNS:download3.epicgames.com,DNS:download4.epicgames.com,DNS:cdn1.epicgames.com,DNS:cdn2.epicgames.com,DNS:cdn1.unrealengine.com,DNS:cdn2.unrealengine.com,DNS:cdn3.unrealengine.com,DNS:static-assets-prod.epicgames.com,DNS:epicgames-download1.akamaized.net")) -out client.csr

# Sign the server CSR
openssl x509 -req -in client.csr -CA rootca.pem  -CAkey rootca.key  -CAcreateserial -out client.pem -days 10000 -sha256

# Export your CA for use in windows
openssl pkcs12 -export -in rootca.pem -out rootca.p12 -nokeys

```

You should then mount a folder containing the files `client.pem` and `client.key` inside the container to `/etc/nginx/ssl/epic/`
