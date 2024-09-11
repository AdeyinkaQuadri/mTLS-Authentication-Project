# mTLS-Authentication-Project
# OVERVIEW
This project demomstrates the setup of Mutual Transport Layer Security (mTLS) authentication using Apache. In mTLS, both the client and the server authenticate each other by exchanging certificates, ensuring a secure, encrypted communication channel berween them. This project was implemented on Ubuntu(server) and Kali Linux(client).

# PROJECT COMPONENTS
Certificates: Self signed certificates for server and client.

Apache Configuration: Configuring Apache to handle mTLS connection

Client Setup: Setting up a client to authenticate and communicate with thw server using mTLS.

# PREREQUISITES
Ensure the following are in plaxe before starting:
Apache Web Server installed and running on your server(Ubuntu).

Openssl for generating the necessary certificates.

SSH access between the client(kali) and server(Ubuntu).

A basic understanding of SSL/TLS and Apache configuration.

# SETUP
# 1. Generate Certificates
   
   Root Certificate Authority (CA): This is used to sign both the server and client certificates.
   # Generate a private key for the Certificate Authority(CA)
    openssl genrsa -aes256 -out ca.key 2048
   # Create the Root CA's self signed Cerificate 
    openssl req -x509 -new -key ca.key -out ca.crt
   
   
   Server Certificate:
   # Generate a server private key 
    openssl genrsa -aes256 -out server.key 2048
   # Generate a server certificate signing request (CSR)
    openssl req -new -key server.key -out server.csr
   # Create a v3 extension file for the server’s certificate
    authorityKeyIdentifier=keyid,issuer
    basicConstraints=CA:FALSE
    keyUsage = digitalSignature, keyEncipherment
    extendedKeyUsage = serverAuth
    subjectAltName = @alt_names

    [alt_names]
    DNS.1 = localhost
    IP.1 = 127.0.0.1 
  The extension file (server_v3.ext) is used to define the roles of the server and add 
   SAN(Subject Alternative Name).
   
   # Generate the Server Certificate by signing  with the Root CA
    openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -passin pass:your_ca_password -CAcreateserial - out 
   server.crt -days 30 -sha256 -extfile server_v3.ext.
   # Generate the Client's Private Key
    openssl genrsa -aes256 -out client.key 2048
   # Generate a client certificate signing request (CSR)
    openssl req -new -key client.key -out client.csr
   # Create a v3 extension file for the Client’s certificate
    authorityKeyIdentifier=keyid,issuer
    basicConstraints=CA:FALSE
    keyUsage = digitalSignature, keyEncipherment
    extendedKeyUsage = clientAuth


  The extension file for the client (client_v3.ext) is used to specify client authentication usage.

   # Genrate the client certificate by signing with the Root CA
    openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -passin pass:your_ca_password -CAcreateserial - out client.crt -days 30 -sha256 -extfile client_v3.ext 
    
 
#   2. Apache Configuration for mTLS.
 # Install Apache:
    sudo apt update
    sudo apt install apache2
  # Check if it's running:
     sudo systemctl starus apache2
     
 # Create directory for your certificates under the Apache Configuration files
    sudo mkdir -p /etc/apache2/ssl
 # Copy the certificates to /etc/apache2/ssl/:
    sudo cp /home/Ubuntu/Documents/mTLS_Certs/server.crt /etc/apache2/ssl/
    sudo cp /home/Ubuntu/Documents/mTLS_Certs/server.key /etc/apache2/ssl/
    sudo cp /home/Ubuntu/Documents/mTLS_Certs/ca.crt /etc/apache2/ssl/
 # Configure Apache by editing the SSL configuration file (default-ssl.conf):
     sudo nano /etc/apache2/sites-available/default-ssl.conf
 # Add the following SSL Directives:
      <IfModule mod_ssl.c>
              <VirtualHost _default_:443>
                      ServerName yourip
                      ServerAdmin webmaster@localhost

                      DocumentRoot /var/www/html

                      ErrorLog ${APACHE_LOG_DIR}/error.log
                      CustomLog ${APACHE_LOG_DIR}/error.log combined 

                      SSLEngine on

                      SSLCertificateFile       /etc/apache2/ssl/server.crt
                      SSLCertificateKeyFile   /etc/apache2/ssl/server.key
                      SSLCACertificateFile    /etc/apache2/ssl/ca.crt

                      SSLVerifyClient require
                      SSLVerifyDepth 1
                      SSLProtocol all TLSv1.2 -TLSv1.3
                      SSLCipherSuite HIGH:!aNULL:!MD5
              </VirtualHost>
       <?IfModule>      

# Enable the SSL module and the site:
    sudo a2enmod ssl 
    sudo a2ensite default-ssl

# Restart Apache to apply changes:
    sudo systemctl restart apache2

#  3. Client Configuration

 # Copying the client.crt,client.key and ca.crt to the client machine(kali)
    scp /home/Ubuntu/Douments/mTLS_Certs/client.crt kali@<kali-ip-address>:/home/kali/Documents/Clients_Certs/
    scp /home/Ubuntu/Douments/mTLS_Certs/client.key kali@<kali-ip-address>:/home/kali/Documents/Clients_Certs/
    scp /home/Ubuntu/Douments/mTLS_Certs/ca.crt kali@<kali-ip-address>:/home/kali/Documents/Clients_Certs/ 
  # Combine the client key and client certificate into one file usually in (.p12 format) using:
      openssl pkcs12 -export -in client.crt -inkey client.key -out client.p12

# Add the CA certificate and client certificate to the Browser's trusted Authorities
     Steps:

    Open Chrome.
    Go to chrome://settings/ in the address bar and hit Enter.
    Scroll down and click "Privacy and Security".
    Click on "Security".
    Scroll down and click on "Manage certificates".
    Under the Authorities tab, click "Import".
    Select your CA certificate file (typically .crt or .pem), and when prompted, ensure that the certificate is trusted for website authentication
    Under the your certificates tab, click Import.
    Navigate to to your client and private key (usually in .p12 format)
    Follow the prompts to import certificates and enter the passphrase if required.

    Open Firefox
    Open Firefox.
    Click on the menu icon (three horizontal lines in the top-right corner), and go to Settings.
    Scroll down to the Privacy & Security section.
    Find the Certificates section and click on View Certificates.
    In the Certificate Manager window, go to the Authorities tab.
    Click Import and navigate to your CA certificate (ca.crt).
    Check the option to trust this CA to identify websites and click OK.
    Go back to the Certificate Manager and this time, go to the Your Certificates tab.
    Click Import.
    Select your client certificate (in .p12 format) and follow the prompts to import it.

#  4. Testing mTLS Authentication
 Open your browser and navigate to https:// server-ip

 You'll be prompted to select the client certificate to  Establish a secured connection.



#  5. Issues Encountered

   Invalid Certificate Errors: This can happen if the certificate’s Subject Alternative Name (SAN) is not properly set. Ensure SAN is configured when generating certificates.
   
Connection Refused: Ensure that the firewall allows traffic on port 443 and that Apache is running.


   
   
   



