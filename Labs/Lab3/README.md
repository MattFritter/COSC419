# COSC 419: Topics in Computer Science
# Fall 2018 - Lab 3

In this lab, we'll be extending the SimpleChat application using new techniques we've covered in Laravel: validation, sessions, and the QueryBuilder.

## Table of Contents
- [Using Session Variables](#session-vars)
- [Validating User Input](#validating-input)
- [Using the QueryBuilder](#using-querybuilder)

<a name="session-vars"></a>
## Using Session Variables (4 marks)

Currently, our SimpleChat application randomly assigns you a new username and color if you refresh the page, or if you close the page and come back later. We will edit our join() function to allow the user to keep their username and color between pages.

In your ```ChatController``` file, modify your ```join``` function so that when the function is called, it checks to see if a session variable for username and color exist. If they don't exist, assign them randomly, otherwise, use the values stored in the session.

In order to check if your session contains a key-value pair already, remember you can use ```session()->has('key')```, which returns a boolean if 'key' exists.

To fetch your session values (if they exist), you can use the ```$var = session('key')``` syntax to get the value associated with a given key.

If the session variables *don't* exist, you'll want to randomly generate your username and and color as you did previously, and then use ```session()->put('key', 'value')``` to write the values to the user's session.

Further documentation of Laravel Session functions can be found <a href="https://laravel.com/docs/5.7/session">here</a>

<a name="validating-input"></a>
## Validating User Input (5 marks)

There are many reasons you may want to validate user input - to ensure that forms are filled out correctly, to avoid unexpected errors, and to maintain security. We'll try using the validation feature when a user creates a new chat room.

On your SimpleChat homepage, you have a form that allows the user to create a new chatroom by submitting a name. This name input is called ```roomName```.

In your create() function, you should be pulling that ```roomName``` value from your Request object. However, we'll add validation for that Request object before we do anything with it.

At the beginning of your create() function, call the ```validate([])``` function on your Request object. You want to validate the ```roomName``` input, making sure it meets the following requirements:
* Is required (must be filled out)
* Has a maximum length of 127 characters
* Is unique within the chatRoom table

The unique validation rule has a special syntax: ```unique:<tableName>,<columnName>```. In our case, the rule you want to use is ```unique:chatRoom,name```.

Once you've done that, you'll need to edit your ```main.blade``` file to report errors, as this is where users will be redirected to if the validation fails. Place an @if/@foreach block like those shown in the lecture three slides inside the jumbotron, above the HTML5 form. If validation fails, that will be where it echos your errors. Use some in-line CSS styling to make them apparent (set the font color to red, or give it background).

Once you're done, try testing the validation. Try creating two rooms with the same name, or entering a name that is longer than 127 characters. It should redirect you back to the main page and echo out the errors.

Further documentation for Laravel validation can be found <a href="https://laravel.com/docs/5.7/validation">here</a>, including a full list of the available validation rules.

<a name="using-querybuilder"></a>
## Using The QueryBuilder (3 marks)

We've learnt to use the QueryBuilder in class. Now we'll use it to create some new API routes.

In your API routes file, add three new get routes: ```getAll```, ```getUser```, and ```getRooms```. These routes shouldn't call a controller, but simply be a closure function executed in the routes file:

```Route::get('getAll', function () { //do something here });```

Then, append the ```use``` statement for the DB facade to the top of the routes file (below the PHP tag): ```use Illuminate\Support\Facades\DB;```

Once you've done that, you're ready to write your three queries:

1. The first query will be for the ```getAll``` route. This query will return the entirety of the ```chat``` database, which is all messages that have been entered. use the ```->toJson()``` QueryBuilder function after your ```->get()``` to return a JSON array, and return this value to the client.
2. The second query will be for the ```getUser``` route. This query will return only posts from the ```chat``` database that include a specific username, which will be specified in the route as a *parameter*. Like the first query, you should return it as a JSON array and then return the JSON array to the client. (hint: You'll need a where() with a 'like' comparator in it and the '%' characters to wildcard search messages for a username).
3. Your third query will be for the ```getRooms``` route. This query will return all names and IDs in the ```chatRoom``` table, but will *not* return hashes. Like the previous two, use ```->toJson()``` to convert it to a JSON array and returning that to the client.

Once you're done, you should check the routes to ensure that they are functioning properly and returning the correct data. Remember, API routes use a URL prefixed by API, i.e.: ```<yourIP>/api/getAll```.

Congratulations, you're done Lab 3!
