# COSC 419: Topics in Computer Science
# Fall 2018 - Lab 8

In this lab, we'll explore some of the additional Flask functions that we discussed in-class, and we'll take a look at deployment using Git.

## Table of Contents
- [Creating a New Flask App](#flask)
- [Flask Project](#website)
- [Automating our Project Install](#automating)

<a name="flask"></a>
## Creating a New Flask App (1 mark)

First, we'll create a new Flask application for our lab. In ```/var/www```, create a new directory ```lab8```. Use the copy command to copy your ```passenger_wsgi.py``` and ```app.py``` files from your ```myapp``` folder to your ```lab8``` folder.

Once you've copied the files, open up your new ```app.py``` file (the one in the lab8 folder), and remove the routes from Lab 6 and Lab 7. *Make sure you do not remove the routes from the original file in the myapp folder*.

Now, open the VirtualHost configuration file you created at ```/etc/httpd/conf.d/myapp.conf```. Change the ```DocumentRoot``` and ```PassengerAppRoot``` to point to your new lab8 directory. Restart Apache using ```service httpd restart```.

If you visit your site IP address, you should now receive a 404 error, because you don't have any route for root ```/``` defined.

<a name="website"></a>
## Flask Project (10 marks, +2 bonus marks)

With your new Flask application, you must build a website. What the website does or contains is up to you (nothing crass, please), but it must implement the following guidelines:

* It must have a minimum of 5 distinct pages
* Each page should have its own template, which inherits from a master layout template
* The master layout template should include some sort of navigation system that includes links to all the pages in your site
* It should use a custom CSS stylesheet, served from the ```static``` directory
* Implement a login system:
	* Include a login page that contains a form asking for username and password, and submit that form using a ```POST``` request (This page does not count towards your 5 page minimum)
	* Have a dedicated ```POST``` route that catches this user input and checks if it is valid (You may hard-code the correct username/password into the code)
	* Use a session variable to keep track of whether a given user is logged in or not
	* If the user is logged in, include a logout link in the master layout template that clears the session variable and redirects the user to the home page
* Have a page accessible only to a logged-in user, which implements *one* of the following if the user is not logged in:
	* Use the ```flash``` and ```redirect``` modules to redirect the user to the home page and flash an error message to the user informing them they must be logged in to view the page.
	* Use the ```abort``` module to throw an HTTP 418 error, and implement a custom error page for the error (The error page does not count towards your minimum 5 page count)

One bonus mark will be awarded for each of the following:
* Going beyond the minimum requirements. This would include good styling, interesting functionality, additional pages, etc.
* Store your username/password information in an SQLite database, and use the ```sqlite3``` module to read from the database when performing a login

<a name="git-start"></a>
## Creating a Git Repo (3 marks)

Now that we've created a new Flask project, we'll use git to help us manage it. First, move to the ```/var/www/lab8``` folder, and use ```git init``` to initialize the git repository.

Then, use ```git add -A``` to add all files within the ```lab8``` directory to the current commit.

Finally, use ```git commit -m "First commit"``` to make the first commit to the repository.

Now we have a local Git repository, and our project files have been committed. The next step is to link this to a remote repository, and then push our commits to the remote repository.

Create a new public repository on GitHub. Do not initialize it with a readme file, just create an empty repo. Then, run the following command, replacing <username> with your username and <reponame> with the name of your new repository:

```git remote add origin https://github.com/<username>/<reponame>.git```

Then, use ```git push -u origin master``` to push your files to the remote repository. You will be prompted to provide your GitHub username and password for authentication.

Visit your GitHub repository and verify that your project files are now there - if so, then you've successfully committed and pushed your work to the remote repository.

<a name="automating"></a>
## Automating our Project Install (3 marks)

Now we'll automate the set-up of our project. Create a bash script called ```install.sh``` in your ```lab8``` directory. Within this bash script, you should do the following (Reference Lab 6 as required):

* Install and upgrade Pip
* Install Flask
* Install Passenger
* Create a directory ```/var/www/lab8```
* Clone the contents of the current directory (that the script is being run from) to the new lab8 directory
* Create the ```myapp.conf``` Apache configuration file in ```/etc/httpd/conf.d```. The easiest way to do this is to include a copy of ```myapp.conf``` file inside your lab8 directory, and simply copy it to the correct location.
* Restart Apache

Then, commit this script locally and push it to your remote GitHub repository. The aim of this script is so that you can clone the repository, run ```install.sh```, and have it setup Pip/Flask/Passenger and configure Apache automatically.