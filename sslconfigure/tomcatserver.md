# Setup Tomcat Server with SSL on Domain

## 1. Prerequisites

* A server running **Ubuntu/Debian** or **RHEL/CentOS/AlmaLinux**.
* Java (OpenJDK) installed.
* A valid **domain name** pointing to the server’s **public IP address** (configured in DNS A record).
* `sudo` or root access to the server.

## 2. Install Java

```bash
# Ubuntu / Debian
sudo apt update
sudo apt install openjdk-17-jdk -y

# RHEL / CentOS / AlmaLinux
sudo yum install java-17-openjdk-devel -y
```

Verify installation:

```bash
java -version
```

## 3. Download and Install Tomcat

```bash
# Navigate to /opt
cd /opt

# Download Tomcat (example: Tomcat 10)
wget https://downloads.apache.org/tomcat/tomcat-10/v10.1.14/bin/apache-tomcat-10.1.14.tar.gz

# Extract
sudo tar xvf apache-tomcat-10.1.14.tar.gz
sudo mv apache-tomcat-10.1.14 tomcat
```

## 4. Configure Tomcat as a Service

```bash
sudo nano /etc/systemd/system/tomcat.service
```

Add the following:

```ini
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
User=root
Group=root
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload systemd and start Tomcat:

```bash
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl enable tomcat
```

## 5. Configure Tomcat Connector for SSL

### Generate Keystore

```bash
sudo keytool -genkey -alias tomcat -keyalg RSA -keystore /opt/tomcat/keystore.jks -keysize 2048
```

* Enter domain info and password.

### Edit Tomcat server.xml

```bash
sudo nano /opt/tomcat/conf/server.xml
```

Uncomment and configure SSL Connector:

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="200" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="/opt/tomcat/keystore.jks"
                     type="RSA"/>
    </SSLHostConfig>
</Connector>
```

Restart Tomcat:

```bash
sudo systemctl restart tomcat
```

## 6. Obtain SSL Certificate using Certbot

* For Tomcat, use **Apache as reverse proxy** or **Certbot standalone**.

### Using Apache Reverse Proxy (Recommended)

* Configure Apache as reverse proxy for Tomcat (port 8080).
* Follow the previous Certbot Apache guide to install SSL.
* Requests to `https://example.com` will be forwarded to Tomcat.

### Using Certbot Standalone

```bash
sudo certbot certonly --standalone -d example.com -d www.example.com
```

* Update Tomcat connector to use `fullchain.pem` and `privkey.pem` in server.xml.

## 7. Verify SSL

* Open browser: `https://example.com:8443` or via Apache reverse proxy `https://example.com`

## 8. Auto Renewal

* Certbot renews automatically.
* For standalone, reload Tomcat after renewal:

```bash
sudo certbot renew --dry-run
sudo systemctl restart tomcat
```

## 9. SSL File Locations

* Certificates (Certbot): `/etc/letsencrypt/live/example.com/fullchain.pem`
* Private Key: `/etc/letsencrypt/live/example.com/privkey.pem`
* Keystore (Tomcat native SSL): `/opt/tomcat/keystore.jks`
