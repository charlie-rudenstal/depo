Depo
====

A shellscript for deploying node.js applications on a remote server using ssh, git and virtualhosts. When doing updates with this script the server copy of the repository will be reset to HEAD.

Installation
--------------------------
On your client(s) (depo.sh)

#### On your server

Put the folder ``DepoProxy`` somewhere on your server, enter it, and the run

	$ npm install

to install its dependencies (currently express).
Make sure that your SSH user can access this folder.

You will also need to install forever from https://github.com/nodejitsu/forever/

#### On your client

Place depo.sh on a client from where you would like to manage the deployments:

	wget https://raw.github.com/charlie-rudenstal/depo/master/depo.sh
	chmod +x depo.sh

Then open depo.sh in your favorite editor and configure the variables
* __REMOTE\_HOST__
  Change to your remote server host (will connect using SSH)

* __REMOTE\_PATH\_TO\_PROXY\_SERVER__
  Will look for server.js (and create a server.js.tmp during the process). Must end with /

* __REMOTE\_PATH\_TO\_PUBLIC\_HTML__
  Folderes for each site will be created in here. DepoProxy and look for a start script such as server.js.

For easier access when you are working on your projects:
Copy it to /usr/bin without the .sh extension and mark it as executable 
	
	$ cp depo.sh /usr/bin/depo
	$ chmod +x /usr/bin/depo

depo.sh has the following options:

- Setup a new virtualhost and checkout the specified repository using git
	
		$ depo create [nameOfVirtualHost] [repository] [optional branch]    

- Update the repository for the selected virtualhost on the server. (Rest to HEAD) 
  This will remove local modifications on the server repository, it is not a merge.

		$ depo update [nameOfVirtualHost]

- Show a list of all virtual hosts, folders, associated git repository urls and current branches on the remote server. 

		$ depo list

- Restart the proxy server (using forever). Will start it if not already running.
	
		$ depo restart

Author: Charlie Rudenstål <charlie4@gmail.com>


On your server (DepoProxy)
-------------------------



server.js will be re-generated by depo.sh

When a new virtualhost is added (using depo create) it will: 
1. Remove the last line (app.listen()), 
2. Append a new app.use() statement
3. Append app.listen() again *

Virtual hosts that depo.sh generates looks something like this:
	app.use(express.vhost('beta.example.org', require('/home/cr/beta.example.org').app));

You may modify server.js on your own risk, but be 
sure to leave no newlines on the end of the file.

