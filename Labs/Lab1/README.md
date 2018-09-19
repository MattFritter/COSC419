# DATA 419: Topics in Computer Science
# Fall 2018 - Lab 1

In this lab, we'll be installing our web server software stack. This includes the Apache Web server, PHP 7.2, the Laravel PHP framework, and an SQLite database. We will also perform basic configuration of Laravel, the web server, and permissions.

You should receive login information for your personal web server either in person at the lab, or via email. This server has been set up with a standard CentOS 7 image, and these instructions have been optimized for installation in CentOS 7.5 x64.

For grading, marks will be deducted for errors in configuration or software errors that impede the function of the web server, framework, or database. Larger errors will result in more marks being deducted.

## Table of Contents
- [First Steps](#first-steps)
- [Installing Apache](#installing-apache)
- [Installing Repos and PHP 7.2](#installing-repos-and-PHP)
- [Installing Laravel](#installing-laravel)
- [Configuring SELinux](#selinux)
- [Configuring Laravel](#configuring-laravel)
- [Bonus Mark](#bonus)

## First Steps

The first step in preparing for our install is to make sure our system is up-to-date. Run the following command:

```yum update```

This will update all packages. It may take a while. If it appears to hang at the final cleanup stage of the package updates, use <kbd>CTRL</kbd>+<kbd>C</kbd> to kill the process, then run:

```yum-complete-transaction --cleanup-only```

Once updates are complete, we will install the git version control software, and the nano text editor:

```yum install nano git```
<a name="installing-apache"></a>
## Installing Apache (2 marks)

First, run:

```yum install httpd```

In CentOS, the Apache web server is commonly referred to as httpd. Users of Ubuntu may be more familiar with the package name apache2.

We will then enable the Apache service. This allows the web server to automatically start following a server reboot:

```systemctl enable httpd.service```

Finally, we restart Apache:

```systemctl restart httpd.service```

If you visit your server IP in a web browser, you should now see a page that looks similar to the following:

<img src="https://i.imgur.com/dDkiewK.png" width="100%">
<a name="installing-repos-and-PHP"></a>

## Installing Repos and PHP (2 marks)

In order to install PHP 7.2, we'll need to install two additional package repositories, as CentOS 7 does not by default support PHP 7. These repositories are known as EPEL and Remi. First, we will need to install the latest RPMs themselves:

```yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm```<br/>
```yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm```

By default, the PHP 7.2 repo that exists within the Remi repositories is not enabled. To enable it, we'll edit it's repo file using Nano:

```nano /etc/yum.repos.d/remi-php72.repo```

Under the listing for remi-php72, change ``enabled=0`` to ``enabled=1``.

Then use <kbd>CTRL</kbd>+<kbd>X</kbd> to exit, and <kbd>Y</kbd> to save the changes.

With the repo enabled, installation of PHP packages will default to using the 7.2 versions from the Remi repo. You can verify the PHP 7.2 repo has been enabled using the following command:

```yum repoinfo remi-php72 list```

Next, we'll install all the PHP components that we will need for the Laravel framework:

```yum install php php-common php-xml php-pdo php-zip php-cli php-mbstring```

You can verify that the correct version of PHP has been installed by running ```php -v```, which should return PHP 7.2.9 (or later).
<a name="installing-laravel"></a>
## Installing Laravel (3 marks)

First, we'll need to install the unzip utility:

```yum install unzip```

Then, we'll navigate to the /var/www folder:

```cd /var/www```

Laravel is commonly installed using Composer, a dependency manager for PHP packages and projects. To install composer, we'll use curl to download the installer:

```curl -s https://getcomposer.org/installer | php```

Than, we'll move the Composer install file to the /usr/local/bin, allowing us to use it via shell:

```mv composer.phar /usr/local/bin/composer```

A quick check using ```composer -V``` should return a version of 1.7.2 or later.

Now, we can create a Laravel project. Run the following command:

```composer create-project --prefer-dist laravel/laravel cosc419```

This command may take a while as Composer downloads all the dependencies required for the Laravel framework. Once it is complete, navigate into the created cosc419 directory:

```cd cosc419```

While we're here, we'll download an SQLite database file from this GitHub using curl. This database will be used in Lab 2 when we are building a web application, but will be configured in this lab:

```curl -L https://github.com/MattFritter/COSC419_Lab1/raw/master/cosc419_db.db > storage/cosc419.db```

In order for Laravel to work properly, it must have write access to the Bootstrap cache, the storage folder, and their respective contents. In order to do this, we'll run two commands to change the user group of the folders and the permissions associated with them:

```chgrp -R apache storage bootstrap/cache```<br/>
```chmod -R ug+rwx storage bootstrap/cache```

Now, the last thing to do is change our Apache configuration to point to our new Laravel project:

```nano /etc/httpd/conf/httpd.conf```

Several changes need to be made in the httpd.conf file. The DocumentRoot line needs to be changed from /var/www/html to /var/www/cosc419/public. You will then need to find the Directory group that refers to the old DocumentRoot (/var/www/html), and similarly change the directory to the new DocumentRoot. You will also need to change the AllowOverride value in this Directory module from ```None``` to ```All```. This enables .htaccess files to override Apache configuration files. Laravel uses an .htaccess file to rewrite URLs as part of it's routing scheme.

As before, exit and save your changes. Then, restart the Apache web server:

```systemctl restart httpd.service```
<a name="selinux"></a>
## SELinux (2 marks)

If you try visiting your web page now, you may notice that it is currently producing an error due to not being able to write to a file. Despite the fact we changed the permissions, SELinux continues to block write access to the necessary files.

Security-Enhanced Linux (SELinux) is a kernel security module that acts as an access control. SELinux uses *contexts* to determine if a given file can be read, written, or executed by a process or user. This is divorced from the normal chmod/chown permissions system.

In order to make Laravel work, we'll need to change the contexts of several folders in the Laravel project. While some users suggest disabling SELinux entirely, it does provide extra security should someone inadvertently get chmod permissions they shouldn't, and is worth keeping. SELinux can also provide security at the network layer, including network interfaces and sockets.

To add the exceptions, we run the following three commands:

```semanage fcontext -a -t httpd_sys_content_t "/var/www/cosc419(/.*)?"```<br/>
```semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/cosc419/storage(/.*)?"```<br/>
```semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/cosc419/bootstrap/cache(/.*)?"```

Note that you must use full, absolute addresses for folders when using the semanage fcontext command.

To update the SELinux contexts using these new permissions, we then run the following command:

```restorecon -R -v /var/www/cosc419```

Now when we visit our site IP, we should see a page similar to the following:

<img src="https://i.imgur.com/fBUhsrG.png" width="100%">
<a name="configuring-laravel"></a>

## Configuring Laravel (1 mark)

We are now almost finished performing our basic web server setup. The final step is to configure the Laravel environment.

Run the following command:

```nano .env```

This will open the default environment file for the Laravel framework. In this file, a number of settings are defined.

First, change the APP_NAME from ```Laravel``` to ```"Your Name"``` - Note, you'll need to encapsulate strings with spaces using quotation marks.

Then, change the APP_URL to ```http://<Whatever Your Server IP Is>```

Lastly, we'll configure the SQLite database. Change the DB_CONNECTION from ```mysql``` to ```sqlite```. Then, change the DB_DATABASE location to ```/var/www/cosc419/storage/cosc419.db```. This also needs to a full, absolute file path - relative file paths will not work. We can then remove the lines that concern the DB_HOST, DB_PORT, DB_USERNAME, and DB_PASSWORD. These are not used when dealing with an SQLite database.

Congratulations! You should now have a working, freshly installed web server running Laravel. At this point, you're finished, and we'll begin developing a web application using Laravel in the next lab.
<a name="bonus"></a>
## Bonus Mark (1 mark)

Create a Bash script that performs these tasks automatically. This could come in handy if you want to quickly deploy this stack in the future, or if your server had a non-recoverable failure.