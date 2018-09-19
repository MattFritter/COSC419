# DATA 419: Topics in Computer Science
# Fall 2018 - Lab 2

In this lab, we'll be building a basic chat application that uses AJAX calls, the Laravel framework, and an SQLite database to allow people to anonymously chat. A basic scaffold is available through GitHub to start with which you will fill out.

## Table of Contents
- [Setting up the Project](#project-setup)
- [Creating our Controller](#creating-our-controller)
- [Controller Functions](#controller-functions)
- [Creating Routes](#creating-routes)
- [Bonus Mark](#bonus-marks)

<a name="project-setup"></a>
## Project Setup (2 marks)

In your ```/var/www``` folder, run the following command to clone the project:

```git clone https://github.com/MattFritter/SimpleChat.git```

This will download the basic controllers, views, etc. First, lets move the old Laravel install to a new folder:

```mv cosc419 cosc419_lab1```

Now, we'll rename the newly created SimpleChat project directory to cosc419, so we don't need to redo our Apache configuration:

```mv SimpleChat cosc419```

Now, we copy our old environment file into our new directory:

```cp cosc419_lab1/.env cosc419/.env```

Now, we can use Composer to reinstall the Laravel dependencies for our new Project:

```cd cosc419```<br/>
```composer install```

Composer should re-download and reinstall the dependencies, creating a vendor directory in your project directory. Lastly, we'll need to fix the permissions in the project folders and redo our SELinux configuration. Run the following commands:

```chgrp -R apache storage bootstrap/cache```<br/>
```chmod -R ug+rwx storage bootstrap/cache```<br/>
```semanage fcontext -a -t httpd_sys_content_t "/var/www/cosc419(/.*)?"```<br/>
```semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/cosc419/storage(/.*)?"```<br/>
```semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/cosc419/bootstrap/cache(/.*)?"```<br/>
```restorecon -R -v /var/www/cosc419```

Our new project should be ready to work on. To confirm, visit your server IP in a web browser. You should see a page that looks similar to the following:

<img src="https://i.imgur.com/ToW97TC.jpg" width="100%">

<a name="creating-our-controller"></a>
## Creating our Controller (2 marks)

To begin building our backend, we'll need to create a new controller in ```app/Http/Controllers```. This can be done manually as shown in the notes, or by using the following command while in the root of the Laravel project:

```php artisan make:controller ChatController``` *If you get an error saying could not open input file, make sure you're in the correct directory: ```cd /var/www/cosc419```

We'll need several facades and packages for this lab. In your new Controller file, below the Namespace declaration but before the class declaration, include the following lines:

```use Illuminate\Http\Request;```
```use Illuminate\Support\Facades\DB;```

The Request package allows us to use the Request object that Laravel uses to pass POST data from the router to the Controller. The DB facade allows us to use Laravel's Query Builder, which is how we're going to interface with out database in this lab.

Now, we are ready to begin writing our functions.

<a name="controller-functions"></a>
## Controller Functions (8 marks, 2 marks each)

# The ```create``` Function

We will require four controller functions for our chat system. The first function we'll need to create is the one that will insert a database record when we create a new chat room, and then redirect the user to this room. This function will be called 'create'.

First, create the public function. As an input parameter, make sure it takes a Request object ```(Request $request)```. Within the function, we want to check that the $request object has a variable called 'roomName' - this is the name of the new room to be created.

If it does have a room name, we'll copy that to a variable, then use the PHP ```md5(str)``` function to create a relatively unique hash for this room that we'll use as an identifier. To do this, just run ```md5()``` on a combination of the room name and the ```time()``` function, which is the current time since the UNIX epoch. You could also use something like the ```rand()``` function instead of ```time()```. The goal here is to make sure that rooms with the same name have different identifiers.

Now, we'll store this data in our database using the Query Builder. The files include a new database file, which includes a table called chatroom. This table takes two text inputs: 'name' and 'hash'. The code below performs the insertion.

```DB::table('chatroom')->insert(['name' => $name, 'hash' => $hash]);```

Finally, we'll redirect the user to a new page: the page URI should be '/chat/$hash', where $hash is the newly generated identifier we made. Note that in PHP, the string concatenation operator is ```.``` (period).

# The ```join``` Function

When a user goes to the page at '/chat/$hash', they are essentially joining in on the conversation. Before they can do that though, we need to pass them some data. This function should take a parameter, $room. $room is the $hash from the URI, which uniquely identifies the chatroom.

First, we'll query our DB to get the name of the chatroom that they are joining from the unique identifier:

	```$roomData = DB::table('chatroom')->where('hash', '=', $room)->first();```
	
Now, we do something a little fun. We want our users to be anonymous, but able to tell each other apart. So, let's randomly assign them a color and a username when they join. You can use two small arrays, one for colors, and one for usernames, then randomly pick a value from each. Here's some example code:

	```$animals = array("Dog", "Cat");```<br\>
	```$user = $animals[array_rand($animals)];```
	
The array_rand() function takes an array as input, and returns a random valid key, which we feed back into the array to get a random username.

Do the same thing for colors, with the random choices being proper HTML named colors. A complete list of named web colors can be found <a href="https://www.w3schools.com/colors/colors_names.asp">here</a>. Pick a small assortment you find pleasing.

Now, we should have the room name, the room identifier (hash), a randomly picked username, and a randomly picked color as variables within the function. Now, return the 'chat' view with this data passed along. The 'chat' view will be looking for data under these specific names:

	```name``` = The name of the chatroom, from the DB<br\>
	```hash``` = The room identifier, from the POST request<br\>
	```user``` = The randomly picked username<br\>
	```color``` = Randomly picked, named web color<br\>

# The ```send``` Function

The 'send' function is another function that uses a POST request; follow the same steps you did as above for setting it up. This function handles what happens when someone submits a new chat message. The POST request will contain data with the following field names:

	```user``` = The name of the user who submitted the message<br\>
	```color``` = The named color of the user who submitted the message<br\>
	```message``` = The message itself<br\>
	```hash``` = The identifier of the room that the message was submitted in
	
Recall that in our last lecture, I mentioned that dealing with user-generated content is potentially dangerous. In this case, we need to assume that all inputs are 'user-generated', as someone could edit the JavaScript code client-side to change the values of user, color, and hash, as well as the actual message. In order to protect ourselves, the first thing we should do is use the ```htmlspecialchars(str)``` function to escape any entities that might be used for an XSS attack. An example:

	```$user = htmlspecialchars($request->input('user'));```
	
After you've sanitized your inputs, we want to combine some of this data into a single HTML message string. Create a new string variable that contains a paragraph (```<p>```) block. Within this block, include a ```<span>``` element for the username, with an in-line style that changes the color of the username based on the color variable. Finally, include the message itself in the paragraph.

Your code should generate an HTML string that looks similar to the following:

	```<p><span style="color: orchid;">Ant</span>: It is I, ANTMAN!</p>```
	
Now, we've done most of the formatting required for the message. The last thing to do is now save our new HTML string in our database. The new DB also includes another table called 'chat', which has 'id', 'room', 'message', and 'timestamp' fields. The id field is autogenerated, so we only need to insert three pieces of data:

	```room``` = The unique identifier for the room, from the POST request<br\>
	```message``` = The HTML that we generated from the combination of the username, color, and message<br\>
	```timestamp``` = a UNIX timestamp, can be generated within the DB insert using the ```time()``` function

Here's Query Builder code to insert the data as a new row:

	```DB::table('chat')->insert(['room' => $hash, 'message' => $html, 'timestamp' => time()]);```
	
Note that this function doesn't need to return anything, as it is called via AJAX POST requests, and doesn't need or expect a response.
	
# The ```getMessages``` Function

The final function we'll define is the getMessages function, which fetches the most recent messages for a given chatroom. It takes as parameters ```$room``` and ```$id```, which are the unique identifier for the chatroom, and an integer that corresponds to an automatically generated ID from the 'chat' table.

First, we need to query the 'chat' database. We'll want to only get results that match the ```$room``` identifier, and have an ID larger than the given ```$id```, meaning they are messages newer than the given ID. We'll also want to perform a sort based on ID, to make sure we return the new messages in the right order:

	```$msgs = DB::table('chat')->where('room', '=', $room)->where('id', '>', $id)->orderBy('id', 'asc')->get();```
	
This code will return a variable ```$msgs```, which is an array that contains our data. However, we'll want to convert it into an associative array of keys (ID) and values (message).

To do this, you'll want to create an empty array, then use a PHP ```foreach()``` loop to loop over the array a row at a time. For each row in the array, we'll push the key-value pair to the array:

	```$array[$msg->id] = $msg->message;```
	
Once we have an associative array, we want to convert it to a JSON array, as we'll be passing this to the JavaScript that is running client-side. We can use the PHP ```json_encode``` function, with the special ```JSON_UNESCAPED_SLASHES``` directive to ensure that the JSON encoding doesn't mess up our message HTML by escaping slashes:

	$jsonArray = json_encode($array, JSON_UNESCAPED_SLASHES);
	
We can then return the JSON-encoded array. On the client side, JavaScript will parse this array, and append the messages in order to the chat message box.

<a name="creating-routes"></a>
## Creating Routes (4 marks, 3 for Web, 1 for API routes)

We'll need to create some routes for our web application. Go to your routes folder at ```/var/www/cosc419/routes```, and take a look at the web.php and api.php route files. They contain a TODO comment that explains which routes need to be added to which file, and what Controller functions these routes will need to call.

In total, you'll need to add two GET/VIEW routes, a GET route with a parameter, and a POST route to your Web routes file. In addition to that, you'll need to add a POST route and a GET route with two parameters to your API route file.

Once your routes and controller are properly set up, the chat functionality of your web application should function properly, although your About and Contact pages will throw an error.

<a name="creating-views"></a>
## Creating Views (4 marks)

Lastly, we'll create a couple of views for the About and Contact pages. This portion of the lab is fairly free-form; you may include whatever information you think is relevant for each page, using additional styling if you prefer.

The only requirement is that both blade templates should extend the master blade template, ```template.blade.php```. You might want to look at the ```main.blade.php``` and ```chat.blade.php``` template files to get an idea of how they integrate into the master template.

Three marks will be given for extending the master template and having no obvious flaws in presentation (```<div>``` objects overlapping each other or breaking out of containers, broken links, general ability to do front-end HTML/CSS). The last mark will be given based on content of the page: a standard about page that follows the same styling as the other pages will be given full marks, but feel free to have fun with what you do.

<a name="bonus-marks"></a>
## Bonus Mark

For a bonus mark, come up with a means for the user to enter their own username and color within the chatroom, and make sure that this data is passed along with subsequent messages.





