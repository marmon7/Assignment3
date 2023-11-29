# How to Create a Sudo User and Configure a Basic nginx Web Server on a Digitial Ocean Droplet
## Part 1: Creating a Sudo User
So you just created new debian droplet on Digital Ocean. It's best practice to have a sudo enabled user rather
than working directly with the root user and disable remote SSH access to the root user. This is so your web 
server can be protected from hackers looking to take control as the root has all permissions.

1. Create a User

The first thing you should do with this new droplet is create a new user with sudo command permissions.
Creating a user can be done by typing the command `useradd` but don't execute it quite yet because we are going 
to add options for a home directory and default shell.

options:
-m : creates a home directory for that user based on the skeleton directory predefined by the system
-s : specifies the default shell foe that user (we will be using bash as the default)

The final command cammand to execute becomes:
```useradd -ms /bin/bash <username>```

Congratulations you now have a user but you aren't finished here. 

2. Give User a Password
   
The new user needs a password
add a password with the command:
```passwrd <username>```
This command will prompt you to enter the password twice but won't display the typed password. 
This is for security reasons.

3. Grant the user sudo command access

To have access to the sudo command the user must be made a part of the sudo group.
A group is collection of users in a linux OS.
The sudo group is the name of the administrator group for debian.
a user can be made a part of a group with the `usermod` command 
but we will be modifying the options to fit our needs.
options:
-a : append to group (adds user to group)
-G : specifies a group name

The final command cammand to execute becomes:
```usermod -aG sudo <username>```

4. Enable ssh for the new user

The `ssh` command won't work automatically for your created user.
This is a simple fix to enable this.
Using the root user type the command:
```cp -r /root/.ssh /home/<username>/```
This will copy the .ssh directory and all its contents with option -r from the root user home directory to the
home directory of the new user.

Now you must give the newly copied .ssh directory ownership under the new user using `chown`
the syntax for chown is `chown [options] <username>:<groupname> <file>`.

In our case the groupname and username will be the same.
We will be using the -R option to give the directory and its contents the same ownership.
The final command will have the structure:
```chown -R <username>:<username> /home/<username>/.ssh/``` 

You are now ready to test the new user.

5. Test your new user

Now that your user is all configured it's good to test if it's working before the next step
exit the droplet with `exit`. Attempt to ssh into the droplet with your new user with this command on your host:

```ssh -i <sshkey-path> <username>@<droplet-ip-address>```
basically replace root with the username

if the droplet logs you in and the bash shell loads up this is what you want
otherwise check your `ssh` command for any typos
if nothing else something went wrong with your user and you should restart from the beginning with step 1

Attempt to use the sudo command to check your sudo permission
```sudo <anything>```
If sudo is working it will ask for your password this is good.
If not there is something wrong with your `usermod` command and you should restart from step 3.

Congratulations your new user now has sudo command access

6. Close access to the Remote Access to the root user

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


