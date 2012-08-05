Depo
====
A shell script for deploying node.js applications to a remote server using ssh, git and virtualhosts.

Sample usage
------------

Connect to your server and forward port 80 to port 8080 by running: (DepoProxy is listening to port 8080)
```bash
$ sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

Go back to your client where depo.sh is installed.   
Now create a virtualhost for the ``beta branch`` of the ``example`` repository by running:
```bash
$ depo create beta.example.org https://github.com/john-doe/example beta    
```
A node server will now be running (using ``forever``) and the ``beta branch`` will now be accessible from ``beta.example.org``. Make some modifications, push them to the remote branch, and update the server using:

```bash
$ depo update beta.example.org    
```

Create another site

```bash
$ depo create www.example.org https://github.com/john-doe/example
```

The master branch will now be accessible from ``www.example.org``, while ``beta.example.org`` will still point to the beta branch.  
   
To see all the virtual hosts and repositories that are active on the server, run:

```bash
$ depo list
johndoe@example.orgs password: 

	Configured virtual hosts
	========================
	app.use(express.vhost('beta.exaple.org', require('/home/johndoe/public_html/beta.example.org').app));
	app.use(express.vhost('www.example.org', require('/home/johndoe/public_html/www.example.org').app));

	Folders and Git repositories
	============================
	                   Folder                      Repo
	        beta.example.org/ https://github.com/john-doe/example @ beta
	         www.example.org/ https://github.com/john-doe/example @ master
```

Installation
------------
#### On your server
Put the folder ``DepoProxy`` somewhere on your server, enter the folder and run ``npm install`` to install its dependencies (currently express). Make sure that your SSH user can access this folder. You also need to install ``forever`` from https://github.com/nodejitsu/forever/

#### On your client
Put `depo.sh` on a client from where you would like to manage the deployments:

```bash
$ wget https://raw.github.com/charlie-rudenstal/depo/master/depo.sh
$ chmod +x depo.sh
```

Then open depo.sh in your favorite editor and configure the variables
* __REMOTE\_HOST__  
  Change to your remote server host (will connect using SSH)

* __REMOTE\_PATH\_TO\_PROXY\_SERVER__  
  Will look for server.js (and create a server.js.tmp during the process). Must end with /

* __REMOTE\_PATH\_TO\_PUBLIC\_HTML__  
  Folders for each site will be created at this path. DepoProxy and look for a start script such as server.js.

For easier access when you are working on your projects, copy it to /usr/bin without the .sh extension and mark it as executable 

```bash
$ sudo cp depo.sh /usr/local/bin/depo
$ sudo chmod +x /usr/local/bin/depo
```

Requirements
------------

depo.sh will launch DepoProxy using forever (https://github.com/nodejitsu/forever/). Install forever on your server using 

```bash
$ npm install forever -g
```

It's not necessary to use forever, you can modify depo.sh to use `node` or something else if you'd like. 

For DepoProxy to work with your current Node.js Express applications
you need to export the app to the proxy. 

```js
exports.app = app
``` 

And if you want to run it as a standalone application as well you can check for the existence of DepoProxy by using ``module.parent``

```js
if (module.parent) {
	exports.app = app;
} else {
	app.listen(80);
}
```

Usage
-----
- __depo create__ [nameOfVirtualHost] [repository] [optional branch]  
  Setup a new virtualhost on the server and checkout the specified repository using git

	```bash
	$ depo create beta.example.org https://github.com/john-doe/example beta    
	```

- __depo update__ [nameOfVirtualHost]  
  Update the repository for the selected virtualhost on the server. (Reset to HEAD) 
  This will remove possible local modifications in the server repository, it is not a merge.

	```bash
	$ depo update beta.example.org
	```

- __depo list__  
  Show a list of all virtual hosts, folders, associated git repository urls and current branches on the remote server. 

	```bash
	$ depo list
	```

- __depo restart__  
  Restart the proxy server (using forever). Will start it if not already running.
	
	```bash
	$ depo restart
	```

How it works
-----------------------

#### depo.sh

depo.sh will access your server using SSH. It will modify the file ``server.js`` located at
the server path you specify for DepoProxy (``__REMOTE_PATH_TO_PROXY_SERVER__``). It will also create new folders in the path ``_REMOTE_PATH_TO_PUBLIC_HTML`` and checkout a clone of the
git repository you supply. 

When running the ``list`` command it will return the virtual hosts defined in ``server.js``, traverse the directory ``_REMOTE_PATH_TO_PUBLIC_HTML`` and retrieve the current repository url and branch for each directory inside, using git.  

#### Virtualhosts with DepoProxy

When adding new virtual hosts using ``depo create`` ``depo.sh`` will connect to your server and re-generate  ``server.js`` at ``__REMOTE_PATH_TO_PROXY_SERVER__``. When a new virtualhost is added it will: 

 1.  Remove the last line of server.js (app.listen(8080)), 
 2.  Append a new ``app.use()`` statement with the virtualhost
 3.  Append ``app.listen(8080)`` again

Virtual hosts that depo.sh generates looks something like this:

```js
app.use(express.vhost('beta.example.org', require('/home/johndoe/public_html/beta.example.org').app));
```

You may modify server.js as you see fit, but be sure to end it with app.listen() and make sure that there are no empty newlines after it.
  

Author  
Charlie Rudenstål  
<charlie4@gmail.com>