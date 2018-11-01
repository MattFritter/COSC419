# COSC 419: Topics in Computer Science
# Fall 2018 - Lab 7

In this lab, we'll be extending our Flask server and learning about the various functionalities that Flask provides to develop our webpages.

## Table of Contents
- [Passing Data to Templates](#templates)
- [Variable Routes](#routes)
- [Defining Route Methods](#methods)
- [Handling Request Data](#requests)

<a name="templates"></a>
## Passing Data to Templates (4 marks)

Much like how you can pass data from your controller to your blade files in Laravel, Flask allows you to pass data from your application to your template files.

First, create a new template in your templates folder called ```example.html```. In this file, provide the following HTML:

 ```
 <html>
  <body>
    {% if rand %}
      <h1>The random number is: <strong>{{ rand }}</h1>
    {% else %}
      <h1>No random number generated.</h1>
    {% endif %}
  </body>
</html>
```

In this case, rand is the name of a variable that we will pass from our application to our template. The ```if```, ```else```, and ```endif``` clauses handle what happens if ```rand``` doesn't exist. ```{% %}``` is used to enclose logic, while ```{{ }}``` is used to echo a variable.

Now, we'll need to create a route in our application that we can use to pass our data to the template. Create a new route for ```/random```. The function for the route should generate a random integer (any length), and then pass that integer to your template using the following code:

```
return render_template('example.html', rand=variableName)
```
Where ```variableName``` is whatever the name of the variable is that has the random integer assigned to it within your function. Once you're done, make sure you restart your server using ```service httpd restart``` to make sure your application changes are reflected. Then, you should see something like this when you go to /random:

<img src="https://i.imgur.com/rgHCHR6.png">

Now, we'll try passing some HTML markup to our template. Go back into your ```example.html``` template and add a second ```if/else/endif``` clause that will simply print a variable ```data```.

Like Laravel, HTML and script is automatically escaped when it is echoed into your template, for security reasons. However, we can echo markup that we explicitly state is safe by using the *Markup* module. First, we'll import markup in our application:

```
from flask import Markup
```

Now we can use the ```Markup()``` function. This function takes a string as an input and returns a string that is specially formatted. Any HTML or script that goes through the Markup() function can then be rendered properly using the same method as above. Note that you should only call the ```Markup()``` function on strings you know are safe, *not user generated content*.

Run this string through the ```Markup()``` function:

```
<p>It was the <strong>best</strong> of times, it was the <strong>worst</strong> of times.</p>
```

Then, we can pass it to our ```example.html``` template just as we did previously, using a comma to separate the second variable:
```
return render_template('example.html', rand=variableName, data=anotherVariableName)
```
As previously, replace ```anotherVariableName``` with whatever variable you are using to store the string from the ```Markup()``` function. Restart your server, and then try visiting the page. It should now look something like this:

<img src="https://i.imgur.com/FMK85pN.png">

<a name="routes"></a>
## Variable Routes (2 marks)

Oftentimes, we may want to use a dynamic route - a route that has a dynamic portion of the URL that we can use as a variable. We did this previously in Laravel, now we'll see how easy it is to do in Flask.

Open up your application file and define a new route called ```routes/<var>```, with a route declaration and function that looks like this:
```
@MyApp.route('routes/<var>')
def routesFunc(var=None):
```
In this case, the route will accept ```routes/<Any string here>```, and then pass that string into the function with the name ```var```. By default, the value will be ```None```, which will cover the case of the user not providing the string. In this case, just make the function convert the string to uppercase and then return it to the user.

You can check for multiple values within the route URL by simply adding additional variables and passing them to the function:
```
@MyApp.route('routes/<var>/<var2>')
def routesFunc(var=None, var2=None):
```
Add a second variable to your route and append it to the first variable before returning the string. It should produce a result like this: visiting the page ```routes/foo/bar``` should return ```FOObar```. This is a simple and effective way of writing a single function to handle multiple potential pages and determining which page the user is viewing. For example, you might have a page ```/userprofiles/<user>```, where the value of ```<user>``` is used to query a database that populates the specified user profile.

<a name="methods"></a>
## Defining Route Methods (3 marks)

You may want to define whether a specific route should use the ```GET```, ```POST```, or both methods, and what should be done different for a ```GET``` versus ```POST``` request. This can be easily done using the request module in the Flask framework. First, we'll import the module:
```
from flask import request
```

Then, we can simply define our required method(s) within route declaration:

```
@MyApp.route('/random', methods=['GET'])
```

If you attempt to make a request that isn't of the specified type, the server will return a 405 Method Not Allowed. You may define more than one method:
```
@MyApp.route('/random', methods=['GET', 'POST'])
```
If you define multiple methods, you can use the ```request.method``` variable to check which method a particular request is using. Request.method will be a string that represents the request type, such as ```GET``` or ```POST```.

For this section, define a route ```/reqType``` that accepts both ```GET``` and ```POST``` requests. Write a function for the route that returns either ```This is a GET request!``` or ```This is a POST request!```. Use an ```if/else``` statement to print the two strings, do *not* simply append the value of ```request.method``` to the string. You can test to see if it's working by using <a href="https://apitester.com">ApiTester</a> to send it some POST requests. The response body returned should contain your string.

<a name="requests"></a>
## Handling Request Data (3 marks)

The ```request``` module is also used to read data from an incoming request. Now we'll learn how to do that.

First, create a template ```input.html``` in your ```templates``` folder, and copy the following HTML into it:
```
<html>
        <body>
                <form action="/submit" method="POST">
                        <input type="text" name="input">
                        <input type="submit" value="Submit">
                </form>
        </body>
</html>
```
Then, create a new route for ```/form```. The function for this route should just return the rendering of the ```input.html``` template. Save and restart Apache. When you visit the ```/form``` page, you should see a text entry box and a submit button.

Now, we'll make the route that will handle the form data. As you can see from the form ```action``` field, this route is going to be ```/submit```. Create the route and make it accept *only* ```POST``` requests. Then in the function read the request value using ```request.form['variableName']```. In this case, our variable name will be ```input```, which is the name given to the text input field. Return the length of the variable. Don't forget to cast the integer result to a string using ```str()``` before you try to return it.

Now, you should be able to go to the ```/form``` page, submit some text, and receive back from the server the length of the text sent. For example, entering ```Halloween``` should return a value of 9.

You can also fetch the query parameters from a ```GET``` request using the ```request``` object. Specifically, you can use ```request.args.get('variableName')``` to get a specific parameter.

Create one more route called ```/getTest```, and make sure that it accepts *only* ```GET``` requests. Have it fetch a parameter called ```key``` and return it to the user. For example, you should be able to visit ```/getTest?key=123456```, and the server will return ```123456```.

As you can see, handling GET and POST request data is very simple in Flask. In fact, many of the functions that we made use of in the Laravel framework can be easily done in Flask using a handful of modules like ```request``` and ```Markup```. This is what makes Flask excellent for small projects: there's minimal code required to perform these simple tasks.

Have a happy Halloween!
