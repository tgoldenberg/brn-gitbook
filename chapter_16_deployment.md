# Chapter 16: Deploying an API

As we demonstrated throughout the tutorial, using Deployd is a very easy way to access a backend API. In this chapter, we will look at how to deploy our API to a real server, so that users all over the world can access their data through the app.

#### Digital Ocean
For the purposes of instruction, we'll be using the a virtual machine hosted by Digital Ocean. There are certainly many options when it comes to hosting an API (Amazon Web Services, Microsoft Azure, Heroku, etc), but Digital Ocean provides a very minimalist interface that will make the instruction easier.

To get started, create an account on www.digitalocean.com. Then you will want to create a "droplet", which is just a name for a virtual machine. You will then be directed to a page where you can select what type of operating system and how expensive the machine is. We'll be going with the default, Ubuntu, and the $5/month option. You also have the option of adding an SSH key, though we'll be skipping this for now.

#### Accessing the Virtual Machine
Once you've created the droplet, you should receive an email from Digital Ocean with the credentials to access it. Using the IP address and the password given, you would log into the machine like this from your terminal (we'll take the example IP address 45.55.60.40:
```
ssh root@45.55.60.40
```
It will prompt you for the password, and then ask you to change the current password. After again pasting the given password, you can enter your new password to login. Now your terminal prompt should look something like this:
```
root@ubuntu-512mb-nyc3-01:~#
```

From here we can start the deployment process. The first thing we need to download is NodeJS. 

#### Installing NodeJS
```
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y build-essential nodejs
```
Once this is completed, you should be able to enter the Node console by typing `node`. Check to see that this works.

#### Installing Git

Next we need to install Git and then clone our API repository. 
```
sudo apt-get install git 

```

#### Installing MongoDB

Next we need to install MongoDB, the database that Deployd runs on.


