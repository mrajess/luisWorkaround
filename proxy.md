# Creating Proxy Server and Hosting PAC File to Bypass Cognitive Service Endpoint for LUIS.AI

## Step 1 - Deploy Ubuntu 20.04 VM in a subnet without Service Endpoint for Cognitive Services enabled
Make sure to document the private IP address of the VM as this will be needed in later configuration steps


## Step 2 - Install Squid

SSH into the VM

Udpate the VM:
>sudo apt update
>
>sudo apt upgrade

Install Squid
>sudo apt install squid


## Step 3 - Configure Squid

Edit squid config file
>sudo nano /etc/squid/squid.conf

Change the Squid ACL list to allow traffic
> Ctrl+W
>
> Type in "http_access deny all" and hit enter
>
> Change to "http_access allow all"
>
> Ctrl+O
>
> Hit "Enter" key
>
> Ctrl+X

Enable Squid service to run at boot
>sudo systemctl enable squid.service

Start Squid service
>sudo systemctl start squid.service


## Step 4 - Install Apache

Install Apache
> sudo apt install apache2


## Step 5 - Configure Apache

Adjust firewall
>sudo ufw allow 'Apache'

Begin creating new Virtual Host by creating a directory
>sudo mkdir /var/www/ipAddressOfYourVMHere
>>Example: sudo mkdir /var/www/10.0.0.4

Assign ownership of the directory
>sudo chown -R $USER:$USER /var/www/ipAddressOfYourVMHere

Ensure permissions are set correctly
>sudo chmod -R 755 /var/www/ipAddressOfYourVMHere

Create our proxy pac file
>sudo nano /var/www/ipAddressOfYourVMHere/pac.js

Add the following code, after modifying the IP address to that of the server hosting squid, to the file and save
```
function FindProxyForURL(url, host) {

    // use proxy for specific domains
    if (shExpMatch(host, "*.luis.ai"))
        return "PROXY ipAddressOfYourVMHere:3128";

    // by default use no proxy
    return "DIRECT";
}
```
>Ctrl+O
>
>Hit "Enter" key
>
>Ctrl+X

Create new Virtual Hosts file

>sudo nano /etc/apache2/sites-available/ipAddressOfYourVMHere.conf

Add the following code, after modifying the IP address to that of the server hosting squid, to the file and save

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName ipAddressOfYourVMHere
    ServerAlias ipAddressOfYourVMHere
    DocumentRoot /var/www/ipAddressOfYourVMHere
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the file we just created
>sudo a2ensite ipAddressOfYourVMHere.conf

Disable default site created by Apache
>sudo a2dissite 000-default.conf

Restart Apache for changes to take effect
>sudo systemctl restart apache2

Configure Apache to start at boot
>sudo systemctl enable apache2


## Step 6 - Configure Windows Client to use Squid Proxy

Go to start menu and search for, and open, "Control Panel"

Open "Internet Options"

Select the "Connections" tab

Select "LAN settings"

Check the box "Use automatic configuration script" and enter the following:
>http://ipAddressOfYourVMHere/proxy.js

Select "OK"

Select "OK" again
