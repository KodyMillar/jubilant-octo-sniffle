# Nginx Tutorial
### Kody Millar

This tutorial walks users through how to configure nginx to serve a website in a Debian Server

Prerequisites:
- A Digitalocean Debian 12 droplet
- An ssh key to connect to that droplet

## Connect to the Debian 12 Server

First, you must connect to your Debian 12 server using your ssh key. It should look something like this:

```
ssh -i C:\Users\your-user\.ssh\do-key root@165.232.135.242
```

`-i` will specifiy the file to read as the private key.

The ip address in the example should be replaced with your droplet's public ip address. You should now be logged into your droplet server as the root user.


## Create a new regular user

To create a new regular user, you will need to use the `useradd` command like this:

```
useradd -ms /bin/bash <your-user>
```

`useradd` in this case creates a new user.  
`-m` will create your home directory and will copy shell configuration files into your home directory.  
`-s /bin/bash` sets the path to the user's login shell to be /bin/bash. Your login shell will run commands everytime you log in to your regular user account. Setting the path to /bin/bash will use Bash as your shell, which creates an easier interactive experience with running commands.

Now we must set your new user's password:

```
passwd <your-user>
```

It will prompt you to enter your new password. You may be confused to see that the terminal is not displaying what you type. Don't worry, that's actually done on purpose so nobody else can see your password. If you're wondering why they don't use asterisks, it's because other people can see the length of your password with asterisks even though they can't see the actual characters.

Now we will need to allow the user to run administrative tasks. You will need this to be able to run important commands that make modifications to files, users, or groups by using the `sudo` command. This will be needed to install and configure nginx in the next steps.

Right now, if you try running `sudo`, you will see that it doesn't work. To give yourself administrative privileges, you must add your user to the sudo group:

```
usermod -aG sudo <your-user> 
```

`usermod` will modify an existing user's account.  
`-aG sudo` will append the sudo group to your user's group list.  

Now your new regular user should be in the sudo group. You can test this out by logging into your new user account:

```
su -l <your-user>
```

You should now be logged in to your new user account. You should now see your prompt change from `root@Debian-12:~#` to something like `your-user@Debian-12:~$`.

Type in the command `groups`. This displays the groups of the current user. You should see the sudo group listed in the terminal as one of the groups.


## Login to user through ssh

In this step, we will configure the sshd_config file so that you can connect to your ssh server as a regular user instead of as root. This will prevent you from going into root first and having to switch to your user. More specifically, it is better to use a regular user account instead of the root account. This is because:

- It is easier for a hacker to hack the root account because they already know the username (root). 
- It gives everybody accountability as users will be able to know who did what.

First, we must copy the .ssh directory from the root home directory to your user's home directory. This directory includes the private and public ssh keys. Having this directory in your user's home directory will allow you to access the ssh server through your regular user:

```
sudo cp -r /root/.ssh/ /home/your-user
```

`cp` copies a file or directory.  
`-r` is recursive, meaning that if a directory is copied, all the files inside the directory and inside its children directory will be copied as well.  
`/root/.ssh/` is the directory you are copying. Make sure to put this before the destination.  
`/home/your-user` this is the destination directory that you are copying the .ssh directory to.

To put this into full effect, go into your home directory with `cd /home/your-user` and change the owner of the .ssh directory and its files to your user and your user group:

```
sudo chown -R your-user: .ssh
```

`chown` will change the owner of the directory.  
`-R` will recursively change the owner, meaning the owners of the directory and all of its files inside will be changed.  
`your-user:` will change the owner of the directory to be both your user and your user group. Having the colon `:` after your user name will change the group owner to the group of the same name as the user name provided. When you created a user, a group of the same name was automatically created as your user group.

Now disconnect from your remote server. Do this by typing `exit` to log out of your regular user account and `exit` again to disconnect from the server.

To test if you can connect to the server through your regular user, replace `root` with your user name:

```
ssh -i C:\path\to\your\key your-user@server-ip-address
```

You should now be connected to your server as your regular user and not the root user.

Now that you can connect directly to your regular user, we will now disable connecting to the server as the root user by configuring the sshd_config file. 

First, open the sshd_config file in vim:

```
sudo vim /etc/ssh/sshd_config
```

We use `sudo` for this command because we will modify the file and change ssh settings.

Now search for the `PermitRootLogin yes` line. Search for it faster by using a slash followed by the search pattern: `/PermitRootLogin`. Press `i` on your keyboard to go into insert mode. Change the "yes" to "no". Press the `Esc` key to exit insert mode. save and quit the file using `:wq` and restart the ssh service:

```
sudo systemctl restart ssh.service
```

Now you will no longer be able to connect to the ssh server as root. 


## Install Nginx

Before we can configure Nginx, we must first install it onto the server:

```
sudo apt install nginx
```

`apt` is a package manager that can install, update, upgrade, and delete packages and dependencies.

We must now check if the Nginx service is enabled and activated. We can check the status of the service by entering:

```
systemctl status nginx.service
```

If it is disabled, you can enable the service by typing:

```
sudo systemctl enable nginx.service
```


## Configure Nginx to Serve a Sample Website

Now we are going to configure Nginx so that it will serve a sample website of our making. First, go into the `/var/www` directory using the `cd` command. The `/var/www` directory is one of the default directories that hold the documents that are served by Nginx.

Once you are inside the `/var/www` directory, create a new directory called my-site:

```
sudo mkdir my-site
```

Go into the new my-site directory you created and create a new file called index.html:

```
sudo touch index.html
```

This file is going to be the sample website that will be served by Nginx. Once you have created this file, go into the file using `sudo vim` and enter the the following code into the index.html file:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```

Make sure you save your file before exiting vim. Next, we are going to add a service block file. This will be used to tell Nginx how it should handle traffic for our sample website domain. We will create this file in the `/etc/nginx/sites-available` directory. Go into this directory and create a new file called `my-site.conf` using `sudo touch`. Open this new file in vim and enter the following server block configuration into the file:

```
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	
	root /var/www/my-site/;
	
	index index.html index.htm;
	
	server_name _;
	
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}
```

You can notice that the path to our `my-site` directory is listed. This is used to specify the website documents that the server will serve. `listen 80 default_server` tells the server to listen on port 80 with ipv4. `listen [::]:80 default_server` is the same, but with ipv6.

Before we can enable the website, we must first disable the default website. In the `/etc/nginx/sites-enabled` directory, there is a symbolic link that is currently pointing to a file called `default` in the `/etc/nginx/sites-available` directory. To disable it, simply remove the symbolic link:

```
sudo unlink /etc/nginx/sites-enabled/default
```

To enable our sample website, we will create a symbolic link in the `/etc/nginx/sites-enabled` directory that points to the `my-site.conf` server block file:

```
sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx/sites-enabled/my-site.conf@
```

`ln` creates a link.  
`-s` will specify the link type as a symbolic link.  

Now that the symbolic link is in the sites-enabled directory, the site has been enabled. The reason we are making a symbolic link in the sites-enabled directory instead of simply putting the my-site.conf file in the directory is because editing a single file in sites-enabled could break the server. Having two different configuration files will allow you to try writing new configurations and features in the symlink in sites-enabled without breaking the server. They will also allow you to have two servers running on one physical server.

We can now test our Nginx configuration by entering the following command:

```
sudo nginx -t
```

`-t` will test the nginx configuration file.

It should display a success message like the following:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Lastly, restart the service so that the changes will be updated:

```
sudo systemctl restart nginx.service
```

Now we are ready to run our sample website! In order to run it, use your server's public ip address. Using the `curl` command will transfer the sample website data from the server:

```
curl <server-ip-address>
```

You should now see your html code from `index.html` displayed in the terminal!
