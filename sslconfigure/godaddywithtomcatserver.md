# GoDaddy Domain and SSL Configuration for Tomcat on Linux

## 1. Buy a Domain from GoDaddy

1. Go to [https://www.godaddy.com/](https://www.godaddy.com/).
2. Search for your desired domain (e.g., `example.com`) in the search bar.
3. If available, click **Add to Cart** and complete the purchase.
4. Note your **domain login credentials** for DNS management.

## 2. Buy an SSL Certificate from GoDaddy

1. In GoDaddy, navigate to **SSL Certificates**.
2. Choose a certificate type (e.g., **Standard SSL** for single domain or **Wildcard SSL** for multiple subdomains).
3. Complete the purchase.
4. After purchase, you’ll need to **request the certificate** (CSR) from your server.

## 3. Generate CSR on Linux (Tomcat Server)

```bash
# Navigate to Tomcat directory
cd /opt/tomcat

# Generate a keystore and CSR
keytool -genkey -alias tomcat -keyalg RSA -keystore keystore.jks -keysize 2048

# Generate CSR from keystore
keytool -certreq -alias tomcat -file tomcat.csr -keystore keystore.jks
```

* Provide your domain info (CN = your domain, e.g., `example.com`).
* The CSR file (`tomcat.csr`) will be uploaded to GoDaddy.

## 4. Submit CSR to GoDaddy

1. Log in to your GoDaddy account.
2. Navigate to **SSL Certificates → Manage → Set up**.
3. Select **Other** for the server type.
4. Upload the `tomcat.csr` file.
5. GoDaddy will issue the SSL certificate files (`.crt` / `.pem`).

## 5. Import SSL Certificate into Tomcat Keystore

1. Download the **primary certificate** and **intermediate/root certificates** from GoDaddy.
2. Import the certificates into your keystore:

```bash
# Import root and intermediate certificates first
keytool -import -trustcacerts -alias godaddyroot -file gdroot.crt -keystore keystore.jks
keytool -import -trustcacerts -alias godaddyintermediate -file gdintermediate.crt -keystore keystore.jks

# Import your domain certificate
keytool -import -trustcacerts -alias tomcat -file example_com.crt -keystore keystore.jks
```

## 6. Configure Tomcat SSL Connector

Edit `server.xml`:

```bash
sudo nano /opt/tomcat/conf/server.xml
```

Add or update SSL Connector:

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="200" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="/opt/tomcat/keystore.jks"
                     certificateKeystorePassword="your_keystore_password"
                     type="RSA" />
    </SSLHostConfig>
</Connector>
```

Restart Tomcat:

```bash
sudo systemctl restart tomcat
```

## 7. Configure Domain DNS

1. In GoDaddy, go to **Domain → Manage DNS**.
2. Add an **A Record**:

   * Host: `@` (or `www` for subdomain)
   * Points to: your server’s public IP
   * TTL: 1 hour
3. Wait for DNS propagation (\~30 mins–2 hours).

## 8. Verify SSL

* Open browser: `https://example.com:8443`
* Or test using `curl`:

```bash
curl -Ik https://example.com:8443
```

✅ Your Tomcat server is now configured with GoDaddy SSL and accessible via your domain.
