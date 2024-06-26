# Upgrade installed packages

```
sudo apt update && sudo apt upgrade -y

```

# Install Java

```
sudo apt install default-jre

```

# Install Wildfly

```
mkdir wildfly
cd wildfly
wget https://github.com/wildfly/wildfly/releases/download/31.0.1.Final/wildfly-31.0.1.Final.tar.gz
tar -xf wildfly-31.0.1.Final.tar.gz
mv wildfly-31.0.1.Final wildfly-server
rm wildfly-31.0.1.Final.tar.gz
cd ..
```

# Config Wildfly

```
sudo cp wildfly/wildfly-server/docs/contrib/scripts/init.d/wildfly.conf /etc/default/
sudo vim /etc/default/wildfly.conf
- setup as wildfly.conf.example.webp example
```

# Run Wildfly as service

```
sudo touch /etc/systemd/system/wildfly.service
sudo chmod +w /etc/systemd/system/wildfly.service
sudo vim /etc/systemd/system/wildfly.service

- add this code:
`
[Unit]
Description=WildFly Application Server
After=network.target

[Service]
Type=simple
ExecStart=/home/wildfly/wildfly-server/bin/standalone.sh -b=0.0.0.0
TimeoutStartSec=300

[Install]
WantedBy=multi-user.target
`

sudo systemctl daemon-reload
sudo systemctl enable wildfly
sudo systemctl start wildfly

```

# Enabling incoming requests

```
sudo vim wildfly/wildfly-server/standalone/configuration/standalone.xml

```

# Install Apache HTTP For https proxy

```
sudo apt install apache2 -y

```

# Generate self signed ssl certificate

```
sudo mkdir /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt

```

# Setup Apache

```
sudo bash -c 'cat > /etc/apache2/sites-available/000-default.conf << EOF
<VirtualHost *:80>
   ServerName localhost
   Redirect / https://localhost/
</VirtualHost>

<VirtualHost *:443>
   SSLEngine on
   SSLCertificateFile /etc/apache2/ssl/apache.crt
   SSLCertificateKeyFile /etc/apache2/ssl/apache.key

   ProxyRequests Off
   ProxyPreserveHost On
   ProxyPass / http://localhost:8080/
   ProxyPassReverse / http://localhost:8080/
</VirtualHost>
EOF'

```

# Setup proxy

```
sudo bash -c 'cat > /etc/apache2/mods-available/proxy.conf << EOF
<IfModule mod_proxy.c>
ProxyPass         /  http://localhost:8080/ nocanon
ProxyPassReverse  /  http://localhost:8080/
ProxyRequests     Off
AllowEncodedSlashes NoDecode

<Proxy http://localhost:8080/*>
  Order deny,allow
  Allow from all
</Proxy>
</IfModule>
EOF'

```

# Install Apache mods

```
sudo a2enmod ssl
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod rewrite
sudo a2enmod headers

```

# Enable Apache as service

```

sudo systemctl enable apache2

```

# Restart both services

```
sudo systemctl restart apache2
sudo systemctl restart wildfly

```

# Run wildfly-server if necessary

```
./wildfly/wildfly-server/bin/standalone.sh

```
