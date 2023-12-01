# How to Create a Sudo User and Configure nginx on a DigitalOcean Droplet
## Part 1: Creating a Sudo User
So you just created new debian droplet on Digital Ocean. It's best practice to have a sudo enabled user rather
than working directly with the root user and disable remote SSH access to the root user. This is so your web 
server can be protected from hackers looking to take control as the root has all permissions.

1. **Create a User**

The first thing you should do with this new droplet is create a new user with `sudo` command permissions.  
Creating a user can be done by typing the command `useradd` but don't execute it quite yet because we are going to add options for a home directory and default shell.

options:  
-m : creates a home directory for the user based on a skeleton directory in the OS  
-s : specifies the default shell for that user (we will be using bash as the default)

The final command to execute becomes: 

```useradd -ms /bin/bash <username>```  

Congratulations you now have a user but you aren't finished here. 

2. **Give User a Password**
   
The new user needs a password
add a password with the command: ```passwd <username>```  
This command will prompt you to enter the password twice but won't display the typed password. This is for security reasons.

3. **Grant the user sudo command access**

To have access to the sudo command the user must be made a part of the sudo group.
A group is collection of users in a linux OS.
The sudo group is the name of the administrator group for debian.
a user can be made a part of a group with the `usermod` command 
We will be using these options to fit our needs.  
options:  
-a : append to group (adds user to group)  
-G : specifies a group name

The final command to execute becomes:
```usermod -aG sudo <username>```

4. **Enable ssh for the new user**

The `ssh` command won't work automatically for your created user.
This is a simple fix to enable this.
Using the root user type the command:

```cp -r /root/.ssh /home/<username>/```

This will copy the .ssh directory and all its contents with option -r from the root user home directory to the home directory of the new user.

Now you must give the newly copied .ssh directory ownership under the new user using `chown`
the syntax for chown is `chown [options] <username>:<groupname> <file>`.

In our case the groupname and username will be the same.
We will be using the -R option to give the directory and its contents the same ownership.
The final command will have the structure:

```chown -R <username>:<username> /home/<username>/.ssh/``` 

You are now ready to test the new user.

5. **Test your new user**

Now that your user is all configured it's good to test if it's working before the next step
exit the droplet with `exit`. Attempt to ssh into the droplet with your new user with this command on your host:

```ssh -i <sshkey-path> <username>@<droplet-ip-address>```

basically replace root with the username in your original ssh command

if the droplet logs you in and the bash shell loads up this is what you want
otherwise check your `ssh` command for any typos  

if nothing else something went wrong with your user and you should restart from the beginning with step 1

Attempt to use the sudo command to check your sudo permission

```sudo <anything>```

If sudo is working it will ask for your password this is good.
If not there is something wrong with your `usermod` command and you should restart from step 3.

Congratulations your new user now has sudo command access

6. **Close access to the Remote Access to the root user**

As your new user you need to find a edit the sshd_config file.
This file is located in the /etc/ssh/ directory.
edit the file with vim (sudo is required)
```sudo vim /etc/ssh/sshd_config```

in vim look for `PermitRootLogin yes`

change `yes` to `no` using insert mode

restart ssh with:
```systemctl restart ssh.service```
it will prompt your password

now attempt to login your droplet with root user
if it gives you permission denied you are done with part 1. Well done.

## Part 2: Configure a Basic nginx Web Server
Now that you have created your user it is time to configure a nginx server to serve a sample website.
1. **Install nginx**

On your droplet installing nginx is simple just use `sudo apt install` using your freshly made sudo user.

```sudo apt install nginx```

You have now installed nginx
You can verify that nginx is installed correctly by typing:

```curl <your-ip-address>```

if you see the default html for nginx you were successful.

2. **Create a Sample html Document**

Create a new directory in `/var/www/` called `my-site` (remember to sudo):

```sudo mkdir /var/www/my-site/```

In bash create a new file called `index.html` inside the my-site directory with `sudo vim`:

```sudo vim /var/www/my-site/index.html```

Put the following code into this file and save:
```html
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
You are now ready to configure your sample website.

3. **Configure nginx to Serve the Sample Website**

To tell nginx to serve our newly created website my-site we must build a server block configuration.
Inside `/etc/nginx/sites-available/` create a new file called `my-site.conf`

```sudo vim /etc/nginx/sites-available/my-site.conf```

Copy this code into the newly created file:
```
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	
	root /var/www/my-site;
	
	index index.html index.htm index.nginx-debian.html;
	
	server_name _;
	
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}
```
This server block will configure nginx to display our my-site website on port 80 with the document root set to `/var/www/my-site/` and serve the `index.html` file inside.

4. **Enable the New Server Block**

Now that you have created a server block configuration we now want to enable it so that nginx can serve your website contents.

To achieve this we must create a symbolic link to our server block configuration file inside the `/etc/nginx/sites-enabled/` directory.
To create the link requires the command for our my-site.conf file:

```sudo ln -s /etc/nginx/sites-available/my-site.conf/ /etc/nginx/sites-enabled/my-site.conf/```

We use -s to specify a symbolic link

5. **Disable the nginx Default Server**

nginx can only run one default server at a time so we must disable the symbolic link for the nginx default server with the `unlink` command
This is the full command to disable the default configuration:

```sudo unlink /etc/nginx/sites-enabled/default```

6. **Test Your New Website**

Test your server configuration with:

```sudo nginx -t```

If there is no errors your server is ready.
Otherwise check steps 1-5 and ensure your symbolic link is correct.

Restart the nginx service with:

```sudo systemctl restart nginx.service```

You can now access your new site using the same `curl` command from step 1

```curl <your-ip-address>```

If you now see your new website the configuration is complete. Congratulations!

You now have made a new user and configured your first website.
