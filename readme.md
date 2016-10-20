
#Django Deployment
----
Today you will be deploying your Courses app from the Django models chapter.

When you take your belt exam you will simply replace the courses project with your belt exam project.

We're going to use github to create a copy of our project in deployment.

## Step 1: Getting Started

Get started by activating your virtual environment. 

Once you have activated your virtual environment, cd into your project directory.  You will now save all of your installed pip modules in a .txt, so that when you deploy your project, all modules will be automatically installed:

```bash
pip freeze > requirements.txt
```

**In your text editor, open your requirements.txt file and remove pygraphviz, pydot and mysql, if present. these modules can be tricky to install and require additional installations, so we remove them now to prevent problems later.**

##Step 2: Committing 

**Important!**

We're about to initialize a new git repo. Your git repo must be initialized within the outer project folder. Double check your location before you initialize your repo.

First we'll create a `.gitignore` file.

```bash
touch .gitignore
```

Open your `.gitignore` file in your text editor and add the lines:

 ```
 *.pyc
 venv/
 ```
 
As the name implies, your gitignore file tells git to ignore any files, directories, etc. you include in the file.  In this case, we're instructing git to ignore all files with the extension "pyc"., as well as your virtual environment, if you have placed one inside of your project folder.

We know this is familiar, but here's a reminder of how to initialize a new repo:

```bash
> git init
> git add --all
> git commit -m "initial commit"
```
Create a new github repo and, back in terminal run these commands, replacing the repo url with your own.

```bash
> git remote add origin https://github.com/AnnaBNana/courses.git
> git push origin master
```

Once you login to AWS and set up a cloud server, you'll be pulling code from your GitHub repository.

## Step 3: Creating an EC2 server instance

>Note: You'll need an AWS account, which you can sign up for [here](http://aws.amazon.com). It's free for a year, so long as you don't have more than 1 (free-tier) instance up at a time!

1. Login to AWS Console at [https://aws.amazon.com/](https://aws.amazon.com/)
2. Once you have logged in you will see your dashboard page:
![Alt text](imgs/2_ec2.png)
3. Launch a new instance from the EC2 Dashboard, as shown below:
![Alt text](imgs/3_launch_instance.png)
4. Select *Ubuntu Server 14.04* option
![Alt text](imgs/4_ubuntu_1404.png)
5. Select *t2.micro* option and click *Review and Launch*
![Alt text](imgs/5_review_launch.png)

### Security settings

1. Click the *Edit security groups* link in the ower right corner
![Alt text](imgs/6_edit_sec_groups.png)
2. SSH option should be there already. Reset SSH source from the dropdown menu to MyIP (this has to be updated everytime you change IP addresses.)
![Alt text](imgs/7_add_rule.png)
3. Click the add a rule button twice to add HTTP and HTTPS, source set to *Anywhere*, and then click ***Click Review and Launch:***
![Alt text](imgs/8_completed_rules.png)
4. You'll be asked to create a key file. This is what will let us connect and control the server from our local machine. 

	Name your pem key whatever makes the most sense to you as shown in the next step.  Give it a generic name, not the name of your project, as we will be re-using this instance. 
	
	The key will automatically be saved to your downloads folder when you click launch instance, but you will want to move it.  See the next item for more information on this critical step. 
![Alt text](imgs/9_download_pem.png)
5. This next part is very important! Put your pem key in a file that has no chance of ***EVER***  being pushed to github. You should not send this file via email, or in any other way make it publicly available:
![Alt text](imgs/1_pem_key_folder_path.png)
6. After launching your instance, you will see a rather confusing screen with some information, as shown below.  In order to move on, scroll to the bottom of the page and confirm that you would like to view your instance.
![Alt text](imgs/10_view_instance.png)
7. Once you have several instances running, you will want to be able to identify what your different instances are for.  We have the option of naming our instance, so let's do so now by clicking on our instance's name column as shown.
![Alt text](imgs/11_name_instance.png)

##Step 4: Connecting to your remote server

Back in your terminal, cd to the folder that holds the key file you just downloaded.

```bash
> cd secretDeveloperStuff/keys
```

Now we're ready to use our .pem file to connect to the AWS instance! That means, in your AWS console, connect to your instance and use the supplied code in your terminal (PC users: use a bash terminal to do this).

Back in your AWS console click the connect button at the top of your screen here:
![Alt text](imgs/12_connect.png)

A popup will appear with instructions on how to connect.  If you are a mac user, run the chmod command, otherwise, skip this command and copy and paste the line starting with ssh into your terminal.
![Alt text](imgs/13_connect_pop.png)

If all goes well, you should be in your Ubuntu cloud server. Your terminal should show something like this in the far left of your prompt:

```bash
ubuntu@54.162.31.253:~$ #Commands you write appear here
```

## Step 5: Deploy!

Now we are going to set up our remote server for deployment. Our server is nothing more than a small allocated space on some else's larger computer (in this case, the big computer belongs to Amazon!).  That space has an installed operating system, just like your computer.  In this case we are using a distribution of linux called Ubuntu, version 14.04.

Although we have linux, our new computer is otherwise empty.  Let's change that so we can start building a server capable of providing content that the rest of the world can access. In order to do so, we have to install some key programs first.  First let's install python, python dev, pip, nginx, and git

In the terminal:

```bash
ubuntu@54.162.31.253:~$ sudo apt-get update
ubuntu@54.162.31.253:~$ sudo apt-get install python-pip python-dev nginx git
```
Now that we've installed some packages using apt-get, let's run update again to make sure apt-get knows we've done those installations.  

In addition, you'll use your newly installed pip to install the virtual environment program so that you can now build a brand new virtual environment on your new computer.

```bash
ubuntu@54.162.31.253:~$ sudo apt-get update
ubuntu@54.162.31.253:~$ sudo pip install virtualenv
```

Now you'll clone your project from GitHub. You'll type into your terminal something that looks like this:

```
ubuntu@ip-my-ip:~$ git clone https://github.com/AnnaBNana/courses.git
```
At the moment your current folder directory should looks something like this.

```bash
- ubuntu
  - repoName
    - apps
    - projectName
    - ... # other files/folders
```

Navigate into this project and run `ls` in your terminal. If you don't see `manage.py` as one of the files, *STOP*. Review the setting up GitHub/Git pieces from earlier.

If everything looks good, let's make that virtual environment, and activate it.

```bash
ubuntu@54.162.31.253:~/myRepoName$ virtualenv venv
ubuntu@54.162.31.253:~/myRepoName$ source venv/bin/activate
(venv)ubuntu@54.162.31.253:~/myRepoName$ pip install -r requirements.txt~~
(venv) ubuntu@54.162.31.253:~/myRepoName$ pip install django bcrypt django-extensions
(venv) ubuntu@54.162.31.253:~/myRepoName$ pip install gunicorn
```
## Step 8: Collect Static Files
### NOTE FOR STEP 8 and BELOW
### Anywhere you see {{repoName}} -- replace that whole thing INCLUDING the {{}} with your outer folder name.
### Anywhere you see {{projectName}} -- replace that whole thing INCLUDING the {{}} with the project folder name.

If you named your repo something different from your project, the repo name and project name may be different, but it is ok if they are the same.

Navigate into your main project directory (where `settings.py` lives). We're going to use a built-in text editor in the terminal to update the code in `settings.py`. For example:

```bash
(venv) ubuntu@54.162.31.253:~/myRepoName$ cd {{projectName}}
(venv) ubuntu@54.162.31.253:~/myRepoName/projectName$ sudo vim settings.py
```

`settings.py` is now open for editing. Use arrows to navigate around, and when you get your cursor to the desired location, press i to enter insert mode.

Add the following (this will allow you to serve static content):

```python
#Inside settings.py
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```
Once you are finished making these additions, tell the vim program to exit insert mode by pressing the escape key and then type `:wq` to save your changes and quit vim.

If you wish to exit without saving your changes type `:q!` after pressing escape.

If you wish to save changes without exiting type `:w` after escape.

Navigate back to the folder that holds `manage.py`. Make sure your virtual environment is activated!

```bash
(venv) ubuntu@54.162.31.253:~myRepoName$ python manage.py collectstatic #say yes
```
## Step 9: Gunicorn

cd back up one level by typing `cd ..` in your terminal.

Now let's test running gunicorn by entering the following:

```bash
(venv) ubuntu@54.162.31.253:~myRepoName$ gunicorn --bind 0.0.0.0:8000 projectName.wsgi:application
```

Run `ctrl-c` and `deactivate` your virtual environment.


Next, we're going to tell this gunicorn service to start a virtualenv, navigate to and start our project, *all behind the scenes*!

```bash
ubuntu@54.162.31.253:~$ sudo vim /etc/init/gunicorn.conf
```

Add the following to this empty file, updating the code that's in between curly braces {{}} (this assumes the name of your virtual environment is `venv`).  Don't forget to type i before copying and pasting the lines below, otherwise vim may cut off a few characters at the beginning!

```
description "Gunicorn application server handling our project"
start on runlevel [2345]
stop on runlevel [!2345]
respawn
setuid ubuntu
setgid www-data
chdir /home/ubuntu/{{myRepoName}}
exec venv/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/{{myRepoName}}/{{projectName}}.sock {{projectName}}.wsgi:application
```

*REMINDER: myRepoName is the name of the repo you cloned in, projectName is the name of the folder that was used when you ran the django-admin startproject command. This folder is sibling to your apps folder.*

Here's what's actually happening:
1. `runlevel`s are system configuration bytes (just use 2,3,4,5 as stated on the start, stop)
2. `respawn` -- if the project stops, restart it
3. `setuid` -- ubuntu can use this project
4. `setgid` -- establishes a group
5. `chdir` -- go to the /home/ubuntu/{yourProject} #This needs to be updated to have your project's name
6. `exec venv/bin/gunicorn`... -- execute `gunicorn` in your `virtualenv` where you pip installed it. Futhermore, bind some workers to it and activate the `wsgi` file in your main project folder. *Look at these names carefully!*

To turn on or off this process:

`sudo service gunicorn start`

`sudo service gunicorn stop`

It's time to start your process!  In your terminal, enter the command below:

```bash
ubuntu@54.162.31.253:~$ sudo service gunicorn start
```
You should see a message confirming that your process has started. 

If you see instead a message that your job failed to start, check to make sure your `.conf` file is correct, with no typos, and all pieces that needed to be changed done correctly.

## Step 10: Nginx
---

One final file to edit. From your terminal:

```bash
ubuntu@54.162.31.253:~$ sudo vim /etc/nginx/sites-available/{{projectName}}
```

Add this to the following, editing what's inside curly brackets {{}}:

```
server {
    listen 80;
    server_name {{yourEC2.public.ip.here}}; # This should be just the digits from AWS public ip!
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/{{myRepoName}};
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/{{myRepoName}}/{{projectName}}.sock;
    }
}
```
Remove the # This should be just the digits from AWS public ip, and should look something like this: `server_name 54.162.31.253;`

Escape and `:wq` to save and exit.


Now in terminal, run the following (taking note of the space after {{projectName}}):

```bash
ubuntu@54.162.31.253:~$ sudo ln -s /etc/nginx/sites-available/{{projectName}} /etc/nginx/sites-enabled
ubuntu@54.162.31.253:~$ sudo nginx -t
```

Take a careful look at everything that's in that file. Compare these names to the `gunicorn` names, and what you actually are using!

##Finally:

```bash
ubuntu@54.162.31.253:~$ sudo service nginx restart
```
If you get an *OK*, your app is deployed! Go to the public domain and you app should be there. If you see anything other than your app, review your server file for errors.

## Step 12.01: Adding a MySQL server (optional)

SQLite is great for testing, but it's not really efficient in the context of real-world use.  This may be a bit too much to go into here, but let's look at a quick summary of why that is, and what we'll use instead.

Although the developers of SQLite have done much to improve its performance, particularly in version 3, it suffers from some lack efficient write concurrency. If your site has a lot of traffic a queue begins to form, waiting for write acess to the database.  Before long, your response speed will slow to a crawl.  This happens only on high-traffic sites, however. 

MySQL databases, on the other hand, are incredibly fast, and very good at performing multiple operations concurrently.  In addition, MySQL can store an incredibly large amount of data, and so is said to scale well.

This might never be a consideration for small to medium sized projects, but is key information in the real world. Very soon you may be working for a company that handles a large volume of requests, and it is important to know why depending on a SQLite database alone is not a practical solution for enterprise or large startups.  

If you'd like to learn how to add a MySQL database to the app we just deployed, read on.  It's not as hard as you might think, thanks to Django migrations!

First, we'll need to install everything necessary to run MySQL from our deployment machine.

```bash
>sudo apt-get install libmysqlclient-dev
```
Let's install mysql-server, then we'll create a MySQL database and database user for our Django application. After running the command below, set your mysql password for root as root.

```bash
>sudo apt-get install mysql-server
```

Then let's just make sure that we were able to install it correctly. First we're going to switch over to root user.

```bash
>sudo su
```

The su command stand for "switch user". If we don't specify a user, Linux/Unix defaults to root user, which is the administrative user. This command provides access to everything in your system, and therefore can be dangerous if you aren't familiar with the effect of the commands you're using. Be careful not to type any commands other than the following until we exit root user a little later.

The command below will open up MySQL console. You have not interacted with MySQL in the command line before because we used MySQL workbench to interact instead. We may not have a GUI available, but it's pretty easy to interact with MySQL in the command line.

```bash
mysql -u root -p
```
You'll see the mysql prompt where we can set up our database.

Let's create a the database for our project. You can call it whatever you want but we recommend giving the name of your project. Note, every command must end with a semi-colon. Make sure to check for them if you have any errors or your commands do not run. Now to create a database on our mysql server.

CREATE DATABASE {{projectName}};

Exit the MySQL prompt by typing `exit;`

**Important!!! After exiting the MySQL promt you MUST type the command `exit` again!**

That's right, we're typing exit twice.  The first time is to exit the MySQL prompt, the second time is to deactivate the root user.  As we warned above, this is a critical step and can result in some problems with installations if we skip it.

Now that we have MySQL all set up, we are ready to change some lines in our `settings.py` document and we can start working with our MySQL database!

If you're in your outer project directory, you must cd into the directory containing your `settings.py` file. If you have followed instructions, you will type:

```bash 
>cd {{projectName}}
>sudo vim settings.py
```
Change your the databases section in settings.py to look like below:

```bash
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '{{projectName}}',
        'USER': 'root',
        'PASSWORD': 'root',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```
remember how to exit vim, by pressing `esc`, `:wq`

We're almost done! Now the only thing left to do is to make migrations!

```bash
(venv) ubuntu@54.162.31.253:~myRepoName$ python manage.py makemigrations
(venv) ubuntu@54.162.31.253:~myRepoName$ python manage.py migrate
```
Now just need to restart nginx `sudo service nginx restart`

Now visit your site! You should be finished at this point, with a fully functioning site.  Your old data will not show up, but you should be able to perform all operations as you did previously.

## Step 12.02: Reconnecting

Remember how we said that we would have to change our security settings every time our IP changes?  The bad news is that this will be every time we try to reconnect.  The good news is that it's easy to change those settings, if you know where to look.

1.  In your aws console, with your instance selected, scroll down to view some options.  Next to security groups, you will see launch-wizard.  Click it!
![Alt text](imgs/14_security_groups.png)
2. Now you just have to update the IP connected to the instance.  In the next window you will see something like this at the bottom of your screen. Click the inbound tab, and then select edit.
![Alt text](imgs/15_edit_groups.png)
3. Now, all that is left to do is let AWS automatically change our IP to the new one. Do this by selecting the dropdown in the SSH row, under source, and select MyIP (it is already selected, but doing so again will refresh your IP to the current one). Once this is done, click save.  Your're ready to SSH into your instance again!
![Alt text](imgs/16_update_security_groups.png)