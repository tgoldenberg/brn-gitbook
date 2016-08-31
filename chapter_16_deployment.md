# Chapter 16: Deploying an API

As we demonstrated throughout the tutorial, using Deployd is a very easy way to access a backend API. In this chapter, we will look at how to deploy our API to a real server, so that users all over the world can access their data through the app.

#### Digital Ocean
For the purposes of instruction, we'll be using the a virtual machine hosted by Digital Ocean. There are certainly many options when it comes to hosting an API (Amazon Web Services, Microsoft Azure, Heroku, etc), but Digital Ocean provides a very simple interface that will make the instruction easier.

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
git clone git@github.com:buildreactnative/assemblies_api.git
```

#### Installing MongoDB

Next we need to install MongoDB, the database that Deployd runs on. There are good instructions on the [mongo documentation](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/), but here they are.

1. 	Import the public key used by the package management system.
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
```
2. 	Create a list file for MongoDB.
```
echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
```
3. 	Reload local package database and install MongoDB packages.
```
sudo apt-get update
sudo apt-get install -y mongodb-org
```

### Configuring MongoDB

You'll notice that the API that we cloned has some extra files --- namely, a `package.json` file and a `server.js` init file. In `server.js`, we establish a connection to an external MongoDB database and then load Deployd. This means we need to set up an external database. For ease of instruction, we'll be using www.mlab.com, though this is by no means the only way.

After creating an account on mlab.com, add a new database under the service's free plan. Then add the necessary collections, such as `messages`, `conversations`, `users`, `notifications`, `groups`, `events`, etc. 

Next under the tab of `users`, add yourself as a database user. To do this, it will ask for a username and password. It is this username and password that you'll use in the `server.js` file, along with the `host` information and `port` number (these are also shown in mlab).

Once you've gotten the `server.js` file configured through Vim or Nano (these are text editors available on Ubuntu), we can get to loading our server. Make sure to test it out before this, by simply initializing it with `node server.js`. 

#### Launching the Server

To launch our server as a background job, we'll be using the NPM package `forever`. Install the package with `npm install -g forever`. Then we can launch the server with `forever start --uid "app" server.js`. To stop the server, we can run `forever stop "app"`. To list all processes on **forever**, run `forever list`.

### Deployd Remote Dashboard

You should now be able to access the Deployd dashboard remotely, once the server is running. It will, however, ask you for a keycode. This is easily obtained through the terminal in your virtual machine. Simply type `dpd keygen` to create a secret key, then `dpd showkey` to reveal the key. By copy/pasting the key into the dashboard, you should now have access just as you did locally.

### Modifying our code

While you won't get all the data created in development, everything else should work smoothly now. You only have to replace the `API` variable in the React Native code to reflect the new endpoint.

```javascript
/* export API = 'http://localhost:2403'; */
export API = 'http://YOUR_IP_ADDRESS_HERE';
```

### Summary

In this chapter, we covered quite a bit --- how to set up a virtual machine with Digital Ocean, install necessary dependencies for a Deployd server, setup an external MongoDB instance, and hook up the new server to our React Native app. We hope you found this helpful!