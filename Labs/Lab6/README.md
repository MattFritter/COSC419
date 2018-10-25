# COSC 419: Topics in Computer Science
# Fall 2018 - Lab 6

In this lab, we'll be setting up the Flask framework, a web framework that uses the Python programming language.

## Table of Contents
- [Using Pip to Install Flask](#setup)
- [Configuring Apache](#apache)
- [Hello World! in Flask](#helloworld)
- [Routing And Basics in Flask](#routing)
- [Templates in Flask](#templates)

<a name="setup"></a>
## Using Pip to Install Flask (2 marks)

Flask can be installed a number of ways, but one of the easiest is using pip. Pip is a package management utility for Python. It comes with Python 3 by default, but our servers come with Python 2.7 as standard.

To install pip, run the following command:
  ```
  sudo yum install python-pip
  ```

The default version of Pip that will be installed is quite outdated - pip 8.1.2, when the current pip release is pip 18.1. Update your pip install with the following command:
  ```
  sudo pip install --upgrade pip
  ```
  
Now, we can install Flask using pip:
  ```
  sudo pip install flask
  ```

Now that we have installed Flask, we'll need to install the module that will allow Apache to serve pages from Flask. For this, we'll need to install the Passenger module. First, we'll add a repo:
  ```
  sudo curl --fail -sSLo /etc/yum.repos.d/passenger.repo https://oss-binaries.phusionpassenger.com/yum/definitions/el-passenger.repo
  ```
  
Next, we'll go ahead and install the Passenger module:
  ```
  sudo yum install -y mod_passenger || sudo yum-config-manager --enable cr && sudo yum install -y mod_passenger
  ```

Finally, restart the Apache webserver. If you go to /etc/httpd/modules, you should notice that there is now a mod_passenger.so file present.
  ```
  sudo service httpd restart
  ```

<a name="apache"></a>
## Configuring Apache (2 marks)

Now we'll need to configure Apache to serve our Flask web application. We'll begin by creating a new folder for our Flask application:
```
sudo mkdir /var/www/myapp
```

This folder will hold our Flask application files. Now, we'll need to create a new configuration file which will tell Apache where this folder is, and how the Passenger module will interact with it.

Create a new ```.conf``` file in ```/etc/httpd/conf.d``` called ```myapp.conf```

Within this file, enter this data:

```
<VirtualHost *:80>
    DocumentRoot /var/www/myapp/
    PassengerAppRoot /var/www/myapp/
    PassengerAppType wsgi
    PassengerStartupFile passenger_wsgi.py
    
    <Directory /var/www/myapp>
      Allow from all
      Options -MultiViews
      Require all granted
    </Directory>
</VirtualHost>
```

This file does several things. First, it creates a VirtualHost listening for all traffic on port 80 (HTTP port). It specifies where Passenger module is looking for the Web Server Gateway Interface (WSGI) file, and sets some permissions for our application.

Once this is complete, we're ready to create our first Flask application.

<a name="helloworld"></a>
## Creating a Simple Hello World Application (2 marks)

Now we need to create our application. We'll focus on two main files: Our application python file and a WSGI file. We'll begin with a WSGI file.

Create a new file called ```passenger_wsgi.py``` in your ```/var/www/myapp``` folder, and write the following line into it:
```
from app import MyApp as application
```

Close and save the file. Now, we can create our application file. Create a file ```app.py``` in ```/var/www/myapp```, and write the following lines into it:
```
from flask import Flask, render_template
MyApp = Flask(__name__)

@MyApp.route("/")
def helloworld():
        return "<h1>Hello World!</h1>"

if __name__ == "__main__":
        MyApp.run()
```

Close and save the file, then restart Apache using ```sudo service httpd restart```. If everything worked properly, you should now see a page that looks like this, confirming that your Flask install is functioning:

<img src="https://i.imgur.com/eOQRXmc.png">

If you see this, congratulations! Your Flask is functioning properly.

<a name="routing"></a>
## Routing and Basics in Flask (4 marks)

In the app.py file, take notice of this section:
```
@MyApp.route("/")
def helloworld():
        return "<h1>Hello World!</h1>"
```

This defines a route and the function that handles the route, similar to the routing system in Laravel. The first line defines a route, in this case the ```/``` route, and the function defined below, ```hello()``` is executed when that route is called.

We'll define another route and function for our application. Make a route called ```/lottery```. Write a basic python function that, when called, generates a <a href="https://www.pythonforbeginners.com/random/how-to-use-the-random-module-in-python">random number</a> between 0 and 999, checks to see if the number is 777, and displays a message informing the user of their number and whether they have won or lost. Make sure you convert this message to a string and return it so that it is passed to the user. Once you're done, make sure you restart your server so your changes are available: ```sudo service httpd restart```.

Your page should look something like this:

<img src="https://i.imgur.com/omYB8WC.png">

> If you get an error page after restarting the server, double check that you don't have any syntax errors in your ```app.py``` file. In addition, double check that you are returning a string, not an integer or any other data type. Also remember that you'll need to import the random module for python using ```import```.

> Note that you can use HTML in your strings and it will render client-side.

<a name="templates"></a>
## Templates in Flask (4 marks)

So far, we've echoed HTML out directly to our user. However, this could very quickly become unsustainable and messy. Instead, we can return whole HTML files using the Flask routing system, in a similar manner to Laravel blade files.

First, create a ```templates``` folder in your ```/var/www/myapp``` folder. Then create a new ```.html``` file with a name of your choice. In this HTML file, put together a basic web page. You can see some ideas at the <a href="https://getbootstrap.com/docs/4.0/examples/">Bootstrap examples page</a>. It should have all the standard elements: ```<head>```, ```<body```, ```<title>```, and should use a couple Bootstrap components to make things pretty. The contents of the page can be whatever you want.

Once you have created your HTML page, create a new route in your Flask ```app.py``` file. Inside the function for the route, we'll return the results of the ```render_template()``` function, which we imported from the Flask Python framework. The syntax of the ```render_template()``` looks like this:
```
render_template("mytemplate.html")
```

This function will generate a string object containing the HTML stored within our template. We can then simply return the HTML to the user. Next week we'll look at how to render dynamic content into a Flask template.

>Note that the ```render_template()``` function does not require that you specify the ```templates``` folder - it will automatically assume this location

>Remember that your Content Security Policy will restrict where you can load resoucres from. To have clients properly render the Bootstrap styling, you'll need to use the ```bootstrapcdn.com``` CDN as the source for your Bootstrap files, or change your CSP.

Congratulations, you finished Lab 6! Welcome to Flask!
