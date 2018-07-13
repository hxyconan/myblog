---
layout: default
title:  "HAProxy configuration for pass-through https request to backend"
date:   2018-07-13 12:30:00 +1100
categories: blog
---

# HAProxy configuration for pass through HTTPS request to corresponding backend

Last time, we present [how to generate self-sign SSL certificate using OpenSSL](/blog/2017/10/30/create-self-sign-ssl-certificate.html). A typical web services network typology is that you hosting multiple backend services after a load balancer. Install proper SSL certificate on service will allow you to establish trust channel between server to your client. There is two places you can install SSL certificate:

## 1. Install SSL certificate on load balancer level
Here, HAProxy will decrypted HTTPS requests, SSL cert must be installed in Load Balancer level and responsible for encrypting and decrypting all traffics. All out-going requests from Load Balance to backends will be unencrypted HTTP.
```
client <--> HTTPS <--> HAProxy <--> HTTP <--> backend server
```
Those backend servers usually in private subnet which will be safe to use HTTP with no encryption.

However, with such manner, you have to including all certificates in your LB  which make it hard to maintain when you have multiply servers at backend and keep adding new ones. Restart LB always tricky since it might outage all connected backend services. Ref to: 


## 2. Install SSL certificate on backend level and HAProxy transparently pass through all HTTPS request
When HAProxy passing though HTTPS traffic it simply sends the raw TCP stream through to the backend which has SSL certificate and handles encryption and decryption. The tricky thing is, since the HAProxy doesn’t have a certificate then it’s not going to decrypt the traffic, means it’s never going to see the Host header. Instead it needs to be told to wait for the SSL hello so it can sniff the SNI request (which including a request domain) during the handshaking stage, then decides which backend server it routing to. 

Following is the codes to routing HTTPS request based on SNI in request. No decryption step in LB level:
```
#---------------------------------------------------------------------
# Proxys to the webserver backend port 443
#---------------------------------------------------------------------
frontend main_ssl
    bind :443
    mode tcp
    option tcplog

    # Wait for a client hello for at most 5 seconds
    tcp-request inspect-delay 5s
    tcp-request content accept if { req_ssl_hello_type 1 }

    use_backend aaa_ssl if { req_ssl_sni -m end .aaa.domain.com }
    use_backend bbb_ssl if { req_ssl_sni -m end .bbb.domain.com }

    default_backend static

backend aaa_ssl
    mode tcp
    balance roundrobin
    server aaa_ssl_server x.x.x.x:443 check

backend bbb_ssl
    mode tcp
    balance roundrobin
    server bbb_ssl_server x.x.x.x:443 check
```

Following is a example of SSL certificate installed in backend server:
```
[ req ]
default_bits        = 2048
distinguished_name  = req_distinguished_name
default_md          = sha256
prompt              = yes
x509_extensions     = v3_req

[ req_distinguished_name ]
countryName           = Country Name (2 letter code)
countryName_default   = { countryName_default }
stateOrProvinceName   = State or Province Name (full name)
stateOrProvinceName_default = { stateOrProvinceName_default }
localityName          = Locality Name (eg, city)
localityName_default  = { localityName_default }
organizationName          = Organization Name (eg, company)
organizationName_default  = { organizationName_default }
commonName            = Common Name (eg, YOUR name)
commonName_default    = { commonName_default }
commonName_max        = 64

[ v3_req ]
basicConstraints      = CA:FALSE
keyUsage              = digitalSignature, nonRepudiation, keyCertSign
subjectAltName          = @alt_names

[alt_names]
DNS.1   = *.aaa.domain.com
```

For supporting wildcard SNI, the ACL rule in haproxy.cfg should be exactly like:

```
use_backend aaa_ssl if { req_ssl_sni -m end .aaa.domain.com }
```


# References
- https://www.haproxy.com/blog/enhanced-ssl-load-balancing-with-server-name-indication-sni-tls-extension/
- https://stuff-things.net/2016/11/30/haproxy-sni/
- <b>How to support wildcard sni:</b> https://stackoverflow.com/questions/24839318/haproxy-reverse-proxy-sni-wildcard

