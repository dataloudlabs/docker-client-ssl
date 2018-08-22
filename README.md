
This repo is an example of how to create a docker environment with nginx serving as reverse proxy to nodejs app.

The Nginx server is configured to use ssl... 
  
  ...delivering its content (through https://) 
  
  ...and to authenticate its clients.


# Creating the keys and certificates 

**for both the server and an example client**

Taken from [here](https://www.djouxtech.net/posts/nginx-client-certificate-authentication/) and [here](http://nategood.com/client-side-certificate-authentication-in-ngi)

You can run these commands inside the /auth folder. Then, copy the files that nginx needs into docker/web/auth.

#### Create the CA Key and Certificate for signing Client Certs
    openssl genrsa -des3 -out ca.key 4096
    openssl req -new -x509 -days 365 -key ca.key -out ca.crt

#### Create the Server Key, CSR, and Certificate
    openssl genrsa -des3 -out server.key 1024
    openssl req -new -key server.key -out server.csr

#### We're self signing our own server cert here.  This is a no-no in production.
    openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt

#### Create the Client Key and CSR
    openssl genrsa -des3 -out client.key 1024
    openssl req -new -key client.key -out client.csr

#### Sign the client certificate with our CA cert.  Unlike signing our own server cert, this is what we want to do.
    openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt

#### Create diffie hellman key for the server
    openssl dhparam -out dhparam.pem 2048

#### Bundle the client certificate and key into p12 file 
    openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12

You'll need to give these to nginx (place them in docker/web/auth, the dockerfile will do the rest):
- dhparam.pem
- ca.key 
- server.crt
- server.key

You'll need to give these to your clients:
- client.key + client.csr
- (or) client.p12


> if you want to use curl or some dev library, the certificate+key are enough. If you want to import the certificate into 
your keychain/firefos/client software, you'll need the p12 file.
- you can remove the -des3 from the commands above if you don't want to use passphrases in your files.


After [configuring nginx](_blank), your client should be able to acess the service. Anyone else (or the client without the certificates) should get a **400 - No required SSL certificate was sent** error.

# In order to run the containers

**(you need to be inside the /docker directory)**

#### You'll need to build the containers first (also, run this ever time you make ANY changes inside the /docker directory)
    docker-compose build --pull;

#### Run the containers
    # Interactively
    docker-compose up;
    # Daemon
    docker-compose up -d;

#### Stop the containers
    docker-compose down


§# Test

In order to test the configuration, in your client, you can use curl...

    # Authenticated
    curl -v -s -k --key client.key --cert client.crt https://example.com
    
    # Not Authenticated
    curl -v -s -k https://example.com

... or import the p12 file into your system/browser and then navigate to your url.