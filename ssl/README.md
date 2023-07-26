# Generate the server key
openssl genrsa -out rootCA.key 2048
# Generate the root CA cert
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem -config rootCA.conf
# Generate a server key request
openssl genrsa -out server-ssl.key 2048
# Generate CSR
openssl req -new -key server-ssl.key -out server-ssl.csr -config csr.conf


# Generate the self-signed root CA certificate, this is needed for cleaner install.
# openssl req -x509 -sha256 -new -nodes -key rootCAKey.pem -days 3650 -out rootCACert.pem

# Signed by Root CA
openssl x509 -req -days 365 -extensions req_ext -extfile csr.conf -CA rootCA.pem -CAkey rootCA.key -in server-ssl.csr -out server-ssl.crt


scp server-ssl.crt mabe@192.168.1.110:/home/mabe/mirror
scp server-ssl.key mabe@192.168.1.110:/home/mabe/mirror

# Configure your latptop to recognize the CA
# $ sudo cp rootCA.pem /etc/pki/ca-trust/source/anchors/
# $ sudo update-ca-trust extract
# $ trust list 
