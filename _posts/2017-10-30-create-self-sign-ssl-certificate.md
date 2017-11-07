---
layout: default
title:  "How to generate self-sign SSL certificate using OpenSSL"
date:   2017-10-30 17:00:00 +1100
categories: blog
---

# How to generate self-sign SSL certificate using OpenSSL

There are couple of ways to create your self-sign ssl certificate but most of Google searched results do not provide clear step-by-step manner about how to create the ssl certificate properly, especially for the certificate including CN and SANs/alt-names. Subject Alternative Names(SANs) are a X509 Version 3 (RFC 2459) extension to allow an SSL certificate to specify multiple names that the certificate should match. 

## Solution 1: 

#### Check OpenSSL installed properly
In my case:
```
# openssl version
OpenSSL 0.9.8zh 14 Jan 2016
```

#### Prepare the .conf file
For including SANs into a certificate, you need to create a .conf file including all configurations such as CN (Common Name) and SANs. For example:
```
# vim example-cert.conf

[ req ]
distinguished_name  = req_distinguished_name
req_extensions      = v3_req

[ req_distinguished_name ]
countryName           = Country Name (2 letter code)
countryName_default   = AU
stateOrProvinceName   = State or Province Name (full name)
stateOrProvinceName_default = New South Wales
localityName          = Locality Name (eg, city)
localityName_default  = Sydney
organizationName          = Organization Name (eg, company)
organizationName_default  = Example Group Limited 
commonName            = Common Name (eg, YOUR name)
commonName_default    = www.example.com
commonName_max        = 64

[ v3_req ]
basicConstraints      = CA:FALSE
keyUsage              = digitalSignature, nonRepudiation, keyCertSign
subjectAltName          = @alt_names

[alt_names]
DNS.1   = example.com
DNS.2   = www.othername.com
DNS.3   = othername.com
```

#### Generate the private key
```
# openssl genrsa -out example-private-key.pem 2048
```

#### Generate the CSR
```
# openssl req -new -out example-cert.csr -key example-private-key.pem -config example-cert.conf
```
The private key file which generated before will be used there. No extra value need to be input since you have defined everything in the example-cert.conf file already. 

Then, you can decode to check the new csr
```
# openssl req -in example-cert.csr -noout -text
Subject: C=AU, ST=New South Wales, L=Sydney, O=Example Group Limited, CN=www.example.com
...
X509v3 Subject Alternative Name: 
    DNS:example.com, DNS:www.othername.com, DNS:othername.com
```

Here, the new "example-cert.csr" includes all CN and SANs will be signed by a public CA. Or, you can self-sign it.

#### Self-sign the new CSR with sha256 algorithm
```
# openssl x509 -req -sha256 -days 3650 -in example-cert.csr -signkey example-private-key.pem -out example-cert.crt -extensions v3_req -extfile example-cert.conf
```
We also give 10 years as expired date for the new self-signed certificate. 

Then, you can check the new certificate file as: 
```
# openssl x509 -in example-cert.crt -text -noout
Issuer: C=AU, ST=New South Wales, L=Sydney, O=Example Group Limited, CN=www.example.com
...
X509v3 Subject Alternative Name: 
    DNS:example.com, DNS:www.othername.com, DNS:othername.com
```


## Solution 2:
So far, you have your certificate sign request which you can provide to CA or you can self-sign it. Another solution which will generate the private key and self-signed CRT(certificate) file in one command. No intermediate CSR will be created.

#### Prepare the .conf file
Create a .conf file including all configurations such as CN (Common Name) and SANs.
```
# vim example-cert.conf

[ req ]
default_bits        = 2048
distinguished_name  = req_distinguished_name
default_md          = sha256
prompt              = yes
x509_extensions     = v3_req

[ req_distinguished_name ]
countryName           = Country Name (2 letter code)
countryName_default   = AU
stateOrProvinceName   = State or Province Name (full name)
stateOrProvinceName_default = New South Wales
localityName          = Locality Name (eg, city)
localityName_default  = Sydney
organizationName          = Organization Name (eg, company)
organizationName_default  = Example Group Limited 
commonName            = Common Name (eg, YOUR name)
commonName_default    = www.example.com
commonName_max        = 64

[ v3_req ]
basicConstraints      = CA:FALSE
keyUsage              = digitalSignature, nonRepudiation, keyCertSign
subjectAltName          = @alt_names

[alt_names]
DNS.1   = example.com
DNS.2   = www.othername.com
DNS.3   = othername.com
```

The most important difference is the "x509_extensions = v3_req" line in the "req" section.


#### Generate the private key, and a self-signed certificate file without CSR
```
# openssl req -x509 -sha256 -nodes -days 3560 -newkey rsa:2048 -config example-cert.conf -keyout example-private-key.pem -out example-cert.crt
```

You can check your new csr as above, it should shows right CN, SANs and sha256 algorithm
```
# openssl x509 -in example-cert.crt -text -noout
```


## Verify your certificate via GUI tool:
You can verify your CSR, CRT or live site SSL at https://www.sslshopper.com  in browser.

## Verify your live site certificate via OpenSSL command line tool
```
# openssl s_client -connect 23.46.42.28:443 -servername www.paypal.com | openssl x509 -fingerprint -text
```
or without Host IP 
```
# openssl s_client -connect www.paypal.com:443 | openssl x509 -fingerprint -text
```


## Reference:
- https://geekflare.com/san-ssl-certificate/
- http://apetec.com/support/generatesan-csr.htm
- https://geekflare.com/joomla-security-vulnerability-scanner/
