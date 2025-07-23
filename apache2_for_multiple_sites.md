Below is a quick, repeatable way to run several sites on one Apache2 server, each on its own port, on Ubuntu 22.04.

---

## 1. Add the extra Listen ports

```bash
sudo nano /etc/apache2/ports.conf
```

Add one `Listen` per port (keep the existing 80/443 if you still use them):

```apache
Listen 8080
Listen 9090
# Listen 8443   # example for an extra HTTPS port
```

Save.

---

## 2. Create a vhost file per site/port

Example: site1 on port 8080, site2 on 9090.

```bash
sudo nano /etc/apache2/sites-available/site1-8080.conf
```

```apache
<VirtualHost *:8080>
    ServerName site1.example.com
    DocumentRoot /var/www/site1

    <Directory /var/www/site1>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/site1-error.log
    CustomLog ${APACHE_LOG_DIR}/site1-access.log combined
</VirtualHost>
```

```bash
sudo nano /etc/apache2/sites-available/site2-9090.conf
```

```apache
<VirtualHost *:9090>
    ServerName site2.example.com
    DocumentRoot /var/www/site2

    <Directory /var/www/site2>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/site2-error.log
    CustomLog ${APACHE_LOG_DIR}/site2-access.log combined
</VirtualHost>
```

(Adjust paths, names, etc.)

---

## 3. Create the document roots

```bash
sudo mkdir -p /var/www/site1 /var/www/site2
echo "Site1 on 8080" | sudo tee /var/www/site1/index.html
echo "Site2 on 9090" | sudo tee /var/www/site2/index.html
sudo chown -R www-data:www-data /var/www/site1 /var/www/site2
```

---

## 4. Enable the sites and reload

```bash
sudo a2ensite site1-8080.conf site2-9090.conf
# (optional) disable the default if you don't need it:
sudo a2dissite 000-default.conf

sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

## 5. Open the firewall (if UFW is on)

```bash
sudo ufw allow 8080/tcp
sudo ufw allow 9090/tcp
# sudo ufw allow 8443/tcp   # if you add a custom SSL port
```

---

## 6. Test

On the server:

```bash
curl http://localhost:8080
curl http://localhost:9090
```

From your browser:
`http://your-server-ip:8080` and `http://your-server-ip:9090`

---

### Optional: HTTPS on a non‑443 port

1. Enable SSL:

   ```bash
   sudo a2enmod ssl
   ```
2. Create a vhost like:

   ```apache
   <VirtualHost *:8443>
       ServerName secure.example.com
       DocumentRoot /var/www/secure
       SSLEngine On
       SSLCertificateFile /etc/letsencrypt/live/secure.example.com/fullchain.pem
       SSLCertificateKeyFile /etc/letsencrypt/live/secure.example.com/privkey.pem
   </VirtualHost>
   ```
3. Add `Listen 8443`, open firewall, reload.

(Let’s Encrypt’s `certbot` can still issue certs over 80/443; you can then serve them on any port.)

---

### Common gotchas

* Forgot to add `Listen` lines for the new ports.
* UFW/iptables or a cloud provider’s security group blocking the port.
* DNS pointing to the wrong host/IP (or not pointing at all).
* SELinux/AppArmor rules (rare on Ubuntu defaults) blocking custom dirs.

---

Need this scripted or templated (Ansible, bash loop)? Let me know.
