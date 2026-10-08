[![hackmd-github-sync-badge](https://hackmd.io/YIWaeRG_SriNtTCLaNXx-w/badge)](https://hackmd.io/@hayat01sh1da/lamp-on-wsl)

<img src="https://hackmd.io/_uploads/rk2etFHu0.jpg" alt="LAMP" />

## 1. Environment

* Ubuntu 20.04.5 LTS
* MySQL 5.7.27

## 2. Apache Server

### 2-1. Install Apache Server


```bash
$ sudo apt install apache2
```

### 2-2. Check Version

Check if Apache2 is installed.

```bash
$ sudo apachectl -v
Server version: Apache/2.4.29 (Ubuntu)
Server built:   2019-09-16T12:58:48
```

### 2-3. Check Behaviour

Start Apache server.

```bash
$ sudo service apache2 start
 * Starting Apache httpd web server apache2
```

***

Open `127.0.0.1` in a browser to show `/var/www/html/index.html`.

<img src="https://user-images.githubusercontent.com/37478830/194862892-eaf9d13b-70a4-468f-ba38-86a363617559.png" alt="13_Apache Top Page" />

## 3. Install PHP

Install a package which enables dialog with MySQL in Apache environment(refer to[How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04)).

```bash
$ sudo apt install php libapache2-mod-php php-mysql
```

## 4. Show Web Page in Apache Server

### 4-1. Change Document Root

Change the default page when accessing `127.0.0.1`.  
Edit the following file with vim.

```bash
$ sudo vim /etc/apache2/sites-available/000-default.conf
```

Edit `DocumentRoot` in Line 12.

```vim
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html/{your directory}/

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

### 4-2. Restart Apache Server

Restart Apache server to reflect change of the document root.

```bash
$ sudo service apache2 restart
 * Restarting Apache httpd web server apache2
 [Fri Nov 08 16:13:12.193332 2019] [core:warn] [pid 13636] (92)Protocol not available: AH00076: Failed to enable APR_TCP_DEFER_ACCEPT
```

### 4-3. Show Web Page

Access `127.0.0.1` and make sure that the page assigned to the document root is shown.
