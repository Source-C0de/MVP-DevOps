The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 2 in Stratos Datacenter. They have some pre-requites to get ready that server for application deployment. Prepare the server as per requirements shared below:



1. Install and configure nginx on App Server 2.


2. On App Server 2 there is a self signed SSL certificate and key present at location /tmp/nautilus.crt and /tmp/nautilus.key. Move them to some appropriate location and deploy the same in Nginx.


3. Create an index.html file with content Welcome! under Nginx document root.


4. For final testing try to access the App Server 2 link (via hostname) from jump host using curl command. For example: curl -Ik https://<app-server-name>/.



# Solution


This is a classic **KodeKloud DevOps Engineer** task. The exam is testing whether you understand:

* Installing a web server
* Configuring SSL
* Managing Linux files
* Configuring Nginx
* Testing connectivity

I'll explain it exactly as if I were teaching a junior DevOps engineer.

---

# Step 1: SSH into App Server 2

From the jump host

```bash
ssh steve@app02
```

(or whichever username KodeKloud provides)

First verify you're on the correct server.

```bash
hostname
```

Example output

```
stapp02
```

---

# Step 2: Become root

Most configuration requires root permissions.

```bash
sudo su -
```

Why?

* Install packages
* Modify nginx configuration
* Copy certificates
* Restart services

Without root, these operations fail.

---

# Step 3: Install nginx

First update package information.

On CentOS/RHEL

```bash
yum install -y nginx
```

On Ubuntu

```bash
apt update
apt install -y nginx
```

Why?

Nginx isn't installed by default.

Verify:

```bash
nginx -v
```

Example

```
nginx version: nginx/1.24.0
```

---

# Step 4: Enable and start nginx

```bash
systemctl enable nginx
systemctl start nginx
```

Check status

```bash
systemctl status nginx
```

Expected

```
Active: active (running)
```

Why?

* start → runs now
* enable → starts after reboot

Many beginners forget **enable**.

---

# Step 5: Create SSL directory

Certificates should not remain inside `/tmp`.

Create a proper directory.

```bash
mkdir -p /etc/nginx/ssl
```

Why?

`/tmp`

* temporary
* cleaned automatically
* insecure place for certificates

Production servers usually store certificates in

```
/etc/nginx/ssl/
```

---

# Step 6: Move certificate and key

```bash
mv /tmp/nautilus.crt /etc/nginx/ssl/
mv /tmp/nautilus.key /etc/nginx/ssl/
```

Verify

```bash
ls -l /etc/nginx/ssl
```

Should show

```
nautilus.crt
nautilus.key
```

---

# Step 7: Create index.html

Document root is usually

```
/usr/share/nginx/html
```

Create file

```bash
echo "Welcome!" > /usr/share/nginx/html/index.html
```

Verify

```bash
cat /usr/share/nginx/html/index.html
```

Output

```
Welcome!
```

---

# Step 8: Configure SSL

Open nginx configuration.

On CentOS

```bash
vi /etc/nginx/nginx.conf
```

or sometimes

```bash
vi /etc/nginx/conf.d/default.conf
```

Find the HTTPS server block (or create one if missing).

Example

```nginx
server {

    listen       443 ssl;

    server_name  _;

    ssl_certificate      /etc/nginx/ssl/nautilus.crt;
    ssl_certificate_key  /etc/nginx/ssl/nautilus.key;

    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }

}
```

Why?

These directives tell nginx

```
listen 443 ssl
```

↓

Accept HTTPS traffic.

```
ssl_certificate
```

↓

Public certificate.

```
ssl_certificate_key
```

↓

Private key.

```
root
```

↓

Website location.

---

# Step 9: Test configuration

Never restart nginx before testing.

```bash
nginx -t
```

Expected

```
syntax is ok

test is successful
```

Why?

If configuration has an error,

```
systemctl restart nginx
```

may fail.

Professional DevOps engineers always run

```
nginx -t
```

first.

---

# Step 10: Restart nginx

```bash
systemctl restart nginx
```

Verify

```bash
systemctl status nginx
```

Should be

```
active (running)
```

---

# Step 11: Allow HTTPS (if firewall exists)

Sometimes KodeKloud doesn't require this, but check.

```bash
firewall-cmd --list-services
```

If https missing

```bash
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

---

# Step 12: Test locally

Run

```bash
curl -Ik https://localhost
```

Expected

```
HTTP/1.1 200 OK
```

Ignore

```
SSL certificate problem
```

because it is self-signed.

Instead use

```bash
curl -Ik https://localhost --insecure
```

or

```bash
curl -kI https://localhost
```

The `-k` (or `--insecure`) flag tells `curl` to ignore certificate verification for self-signed certificates.

---

# Step 13: Test from Jump Host

Exit App Server

```bash
exit
```

Now on Jump Host

```bash
curl -Ik https://stapp02
```

If certificate is self-signed

```bash
curl -kI https://stapp02
```

Expected

```
HTTP/1.1 200 OK
```

---

# Common mistakes in KodeKloud

❌ Forgetting to start nginx

```
systemctl start nginx
```

---

❌ Forgetting to enable nginx

```
systemctl enable nginx
```

---

❌ Leaving certificates inside `/tmp`

Move them.

---

❌ Wrong certificate path

Example wrong

```
ssl_certificate /tmp/nautilus.crt;
```

Correct

```
ssl_certificate /etc/nginx/ssl/nautilus.crt;
```

---

❌ Forgetting `ssl`

Wrong

```nginx
listen 443;
```

Correct

```nginx
listen 443 ssl;
```

---

❌ Forgetting `nginx -t`

Always test configuration first.

---

# Quick Command Summary

```bash
sudo su -

yum install -y nginx          # or apt install -y nginx

systemctl enable nginx
systemctl start nginx

mkdir -p /etc/nginx/ssl

mv /tmp/nautilus.crt /etc/nginx/ssl/
mv /tmp/nautilus.key /etc/nginx/ssl/

echo "Welcome!" > /usr/share/nginx/html/index.html

vi /etc/nginx/nginx.conf
# configure:
# listen 443 ssl;
# ssl_certificate /etc/nginx/ssl/nautilus.crt;
# ssl_certificate_key /etc/nginx/ssl/nautilus.key;

nginx -t
systemctl restart nginx

curl -kI https://localhost
```

Then, from the jump host:

```bash
curl -kI https://stapp02
```

## Interview Tip

In real production environments, SSL certificates are **not** stored in `/tmp` because that directory is temporary and may be cleaned on reboot. They are typically kept under `/etc/nginx/ssl/` (or another protected directory), and before reloading Nginx, experienced DevOps engineers always validate the configuration with `nginx -t` to avoid downtime caused by syntax errors.
