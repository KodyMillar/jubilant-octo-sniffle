# Nginx Tutorial
### Kody Millar

This tutorial walks users through how to configure nginx to serve a website in a Debian Server

Prerequisites:
- A Digitalocean account
- An ssh key


## Create a new regular user

To create a new regular user, you will need to use the `useradd` command like this:

```
useradd -ms /bin/bash <your-user>
```

`useradd` in this case creates a new user.  
`-m` create's the user's home directory if it doesn't already exist. This is important  
`-s /bin/bash` sets the path to the user's login shell to be /bin/bash. Your login shell will run commands everytime you log in to your regular user account. Setting the path to /bin/bash will use Bash as your shell, which creates an easier interactive experience with running commands.

Then we will need to allow the user to run administrative tasks. You will need this to be able to run important commands that make modifications to files, users, or groups by using the `sudo` command. This will be needed to install and configure nginx in the next steps.

Right now, if you try running `sudo`, you will see that it doens't work. To give yourself administrative privileges, you must add your user to the sudo group:

```
usermod -aG sudo <your-user> 
```

`usermod` will modify an existing user's account.  
`-aG sudo` will append the sudo group to the user's group list.  

Now your new regular user should be in the sudo group. You can test this out by logging into your new user account:

```
su -l <your-user>
```

You should now be logged in to your new user account. You should now see your prompt change from `root@Debian-12:~#` to something like `your-user@Debian-12:~$`.

Type in the command `groups`. This displays the groups of the current user. You should see the sudo group listed in the terminal as one of the groups.

To create a password...


## Login to user through ssh

In this step, we will configure the sshd_config file so that you can connect to your ssh server as a regular user instead of as root. This will prevent you from going into root first and having to switch to your user. More specifically, it is better to use a regular user account instead of a root account. This is because:

- One reason **
- Two reason **

First, we must copy the .ssh directory from the root home directory to your user's home directory. This directory includes the private and public ssh keys. Having this directory in your user's home directory will allow you to access the ssh server through your regular user:

```
sudo cp -r /root/.ssh/ /home/your-user
```

`cp` copies a file or directory  
`-r` is recursive, meaning that if a directory is copied, all the files inside the directory and inside its children directory will be copied as well.
