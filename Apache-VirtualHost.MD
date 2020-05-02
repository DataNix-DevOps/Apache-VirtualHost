Apache Virtual Hosts allows multiple websites to run on one Web server. With virtual hosts, you can specify the site document root (the directory which contains the website files), create a separate security policy for each site, use different SSL certificates for each site and much more.

In this doc, I'll provide a step by step instructions about how to set up Apache Virtual Hosts on a CentOS 7 server.
Prerequisites

Make sure you met the following prerequisites before continuing with this tutorial:

    You have a domain name pointing to your public server IP. We will use example.com.
    You are logged in as a user with sudo privileges.

Creating Directory Structure

DocumentRoot is the directory where the website files for a domain name are stored and served in response to requests. You can set the document root to any location you want, this example we will use the following directory structure:
```sh
/var/www/
├── example.com
│   └── public_html
├── example-1.com
│   └── public_html
├── example-2.com
│   └── public_html
```

We’re creating a separate directory for each domain we want to host on our server inside the /var/www directory. Within each of these directories, we will create a public_html directory that will store the domain website files.

Create the root directory for the domain example.com using the mkdir command:
```sh
sudo mkdir -p /var/www/example.com/public_html
```
For testing purposes we will create an index.html file inside the domain document root directory. Open your editor and create a HTML file with the following contents:
/var/www/example.com/public_html/index.html
```sh
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>Welcome to example.com</title>
  </head>
  <body>
    <h1>Success! example.com home page!</h1>
  </body>
</html>
```
Copy

All commands are executed as sudo user, so the newly created files and directories are owned by the root user. To make sure there are no permission issues, change the ownership of the domain document root directory to the apache user :

sudo chown -R apache: /var/www/example.com

Creating Virtual Host File

There are a few ways to set up a virtual host. You can either add all Virtual Host Directives in a single file or create a new configuration file for each Virtual Host Directive. Personally, I prefer the second approach because it is more maintainable.

By default, Apache is configured to load all configuration files that ends with .conf from the /etc/httpd/conf.d/ directory.

To create a virtual host for a specific website open your editor of choice and create the following basic Virtual Host configuration file:
/etc/httpd/conf.d/example.com.conf
```sh
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin webmaster@example.com
    DocumentRoot /var/www/example.com/public_html

    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
    </Directory>

    ErrorLog /var/log/httpd/example.com-error.log
    CustomLog /var/log/httpd/example.com-access.log combined
</VirtualHost>
```
Copy

    ServerName: The domain that should match for this virtual host configuration. This should be your domain name.
    ServerAlias: All other domains that should match for this virtual host as well, such as the www subdomain.
    DocumentRoot: The directory from which Apache will serve the domain files.
    Options: This directive controls which server features are available in a specific directory.
        -Indexes: Prevents directory listings.
        FollowSymLinks: This option tells your web server to follow the symbolic links.
    AllowOverride: Specifies which directives declared in the .htaccess file can override the configuration directives.
    ErrorLog, CustomLog: Specifies the location for log files.

Edit the file according to your needs and save it.

It is important that the configuration file name ends with .conf. You can name the configuration file as you like but the best practice is to use the domain name as the name of the virtual host configuration file.

Test the configuration file syntax with:
```sh
sudo apachectl configtest
```
If there are no errors you will see the following output:

Syntax OK

To activate a newly created virtual host, restart the Apache service with:
```sh
sudo systemctl restart httpd
```
