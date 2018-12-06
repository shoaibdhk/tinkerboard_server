# How to Create a cloud server With NextCloud and LAMP Stack Web Server 

### Before Reading
I am not an expert. This is not finished yet. There are something I still couldn't figure out. I am just trying by myself. I have searched a lot but didn't found a single information how to do it. I didn't found much resource for Tinkerboard, but there is a lot of resource for Raspberry Pi. I chose Tinkerboard because of its extended RAM and CPU. I will update if I discover anything.

## Choosing OS
I have chosen [Armbian Stretch](https://www.armbian.com/tinkerboard/) which is a very lightweight OS and a __Debian based__ linux system. Since I am going to use a headless server, I strongly recommend this OS. I always had issue with TinkerOS. I don't why everytime I tried to update my distro it gives me issues. I like its `armbian config` where you can just simply type `sudo armbian-config`and you can easily configure from the terminal. For more infor you can visit [Armbian Doc](https://docs.armbian.com/) to get started.

## Samba
[Samba](https://www.samba.org) is a file sharing system when you are within the same wifi/network. You can create a network on you system. Installation is very easy. Here is the [full and easy instruction](https://tutorials.ubuntu.com/tutorial/install-and-configure-samba#0). Even though it is a Ubuntu link but it works since it uses debian system. I saw some videos where people says **WORKGROUP** has to be same as current Desktop's **WORKGROUP**, you don't need to do that. Simply add you network location with the `ip_address` Something I just want to add:
- Start `$ sudo /etc/init.d/smbd start`
- Stop `$ sudo /etc/init.d/smbd stop`
- Restart `$ sudo /etc/init.d/smbd restart`
- Configure `sudo vi /etc/samba/smb.conf` go to the bottom of the line and add the following:
```
         [harddrive] (your choice)
         comment = Hard Disk Western Digital (your choice)
         path = /media/mycloud (the path you want to share)
         readonly = no
         browsable = yes (if you want to read and write)
```

## Webmin (Optional)
Then I actually wanted to instal Webmin. Webmin is a dashboard where you can configurey things for your server such as file manager, editing samba file sharing system, monitoring your system and make your life very easier. The installation process are [here](http://www.webmin.com/deb.html) and see **Using the Webmin APT repository**
But I have had an issue when installing its dependency `apt-get-versions`. When I tried to install I got this error:
```
** initializing cache. This may take a while **
Error: No information about packages! (Maybe no deb entries?)
dpkg: error processing package apt-show-versions (--configure):
 subprocess installed post-installation script returned error exit status 255
Errors were encountered while processing:
 apt-show-versions
E: Sub-process /usr/bin/dpkg returned an error code (1)

```
Then I found the solution from [this link](https://askubuntu.com/questions/916199/install-apt-show-versions-inside-an-ubuntu-docker-container)
All you just have to do the following.(#):

```
# rm /etc/apt/apt.conf.d/docker-gzip-indexes
# apt-get purge apt-show-versions
# rm /var/lib/apt/lists/*lz4
# apt-get -o Acquire::GzipIndexes=false update

```
After that I tried to install `apt-show-versions` by the following command:
```
# apt-get install apt-show-versions
```
Then All you just have to type `your_ip:10000` and sign in with your username and password that has **root previliges**.

## PHPMyAdmin
Installing LAMP Stack is easy, but activating PHPMyAdmin I found some difficulties. It keep saying missing. So I just want to keep all my solutions here.
- First install some dependencies: 
```               
                  $ sudo apt-get install php-mbstring php7.0-mbstring php-gettext libapache2-mod-php7.0
```
- restart `apache2` and `mysql`
  - `sudo /etc/init.d/apache2 restart`
  - `sudo /etc/init.d/mysql restart`
- Then link in `/var/www` like this: `$ sudo ln -s /usr/share/phpmyadmin /var/www/`

For reference [1](https://stackoverflow.com/questions/10769148/extension-mysqli-is-missing-phpmyadmin-doesnt-work) [2](https://askubuntu.com/questions/387062/how-to-solve-the-phpmyadmin-not-found-issue-after-upgrading-php-and-apache)
## NextCloud

### NextCloudPi
I find installing NextCloudPi easier that installing NextCloud manually. When I run the following command:
```
# curl -sSL https://raw.githubusercontent.com/nextcloud/nextcloudpi/master/install.sh | bash
```
This will install LAMP stack and automatically configure the process but I don't know if it right or not because I also want to use this server as a web server too. So its tempting. I am still gonna use this.
I found a better instruction in [youtube](https://www.youtube.com/watch?v=oIYg2O_ilbw), this works for me.

### Installing without NextCloudPi
Installing without NextCloudPi is pretty hard but still I wanted to do that because I can't browse web after installing NextCloudPi. That is the problem. Here are some instructions:

- I tried to installed with snap package, which you can install via `sudo apt install snapd`
- Here is the [documentation](https://docs.nextcloud.com/server/14/admin_manual/installation/source_installation.html#) for that
- **Very Importnat**: You have to create a `/etc/apache2/sites-available/nextcloud.conf` file with these lines in it:
```
Alias /nextcloud "/var/www/nextcloud/"

<Directory /var/www/nextcloud/>
  Options +FollowSymlinks
  AllowOverride All

 <IfModule mod_dav.c>
  Dav off
 </IfModule>

 SetEnv HOME /var/www/nextcloud
 SetEnv HTTP_HOME /var/www/nextcloud

</Directory>
```

### Some Notes

- When I tried to format my hard disk for keeping the nextcloud data I could not do it via browser I actually had to it via `ncp-config`

### Docker
I don't know much about docker but I couldn't install via `armbian-config`. I tried to install via 

```
# curl -sSL https://get.docker.com | sh
```

## Let's Encrypt
It provides free SSL Certificate for your domain [Certbot's website](https://certbot.eff.org/lets-encrypt/debianstretch-apache) explains a lot about this. This website is very usefull how to encrypt the website.
## Linking with Domain
Now here I got stuck, I didn't found any useful information how should I link my nextcloud outside of the network. I have realized that I have open port 80 and 443 in my router and also reserve a fix internal IP address and I configure that in my router. 
