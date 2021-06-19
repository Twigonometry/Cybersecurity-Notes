# Juicy Part 3 Writeup

This was a challenge I wrote that involved a deserialisation vulnerability in a PHP web app. Find the source code [here](https://github.com/ShefESH/BakeryTF/tree/main/Web/Saucy/Saucy%20Part%203)

## Solution

Visiting the page, we see it seems to be some sort of web app for viewing and creating recipes:

![[Pasted image 20210511212151.png]]

If we try searching for `horseradish-sauce.txt` as suggested, we get a recipe for horseradish sauce back:

![[Pasted image 20210511212241.png]]

We can also read recipes from the database by ID - let's try an ID of 1 by visiting `/get-recipe.php` with the `?id=1` parameter:

![[Pasted image 20210511212911.png]]

We get a 'secret recipe' back - but it seems to be encrypted!

![[Pasted image 20210511212937.png]]

We can also use the third option to 'build' a recipe using a string that's saved in the database. This is especially interesting - giving it a random string takes us to the `get-recipe.php` page also, but this time with a `string_id` parameter. Supplying `1` gives us an encrypted recipe for mayonnaise.

Let's try and find out more about how the app works. The fact we have to specify `.txt` when reading a recipe file is interesting - that suggests we may be directly accessing the filesystem. Can we get an LFI?

The source has a message for developers:

![[Pasted image 20210511212346.png]]

```html
<!-- DEVS: See code for retrieving recipes in get-recipe.php -->
<!-- DEVS: See code for submitting new recipes in create-recipe.php -->
```

We can try reading one of these files - after a bit of experimenting, supplying `../get-recipe.php` works!

![[Pasted image 20210511212441.png]]

The `..` operator lets us go up a directory, presumably from a directory storing recipe `.txt` files to the main directory where code is stored.

We can now arbitrarily read the web application code!

![[Pasted image 20210511212512.png]]

We could also use this to poke around for sensitive files - perhaps database creds, or SSH keys - but I know this challenge doesn't involve remote access to the database or webserver itself, so I won't go into doing that.

`create-recipe.php` shows we can insert data into the database:

![[Pasted image 20210511212607.png]]

Here's the key code:

```php
$recipe_string = $_GET["recipe_recipe"];

//check they're submitting a Secret Recipe class
if(strpos($recipe_string, 'O:12:"SecretRecipe"') !== false) {
    //insert into recipe_strings database
    error_log("Inserting");

    Database::initialize();

    $stmt = Database::$conn->prepare("INSERT INTO recipe_strings (recipe_recipe) VALUES (?)");

    // execute statement with submitted string

    $stmt->bind_param("s", $recipe_string);

    $stmt->execute();

    // close statement
    $stmt->close();

    // get ID of last item
    $id = Database::$conn->insert_id;

    echo("Thanks for submitting! Your ID: ". $id);
}
```

It looks like it is expecting a serialised object of the `SecretRecipe` class, as it checks our string contains `O:12:"SecretRecipe"`. This is an indicator that there may be some deserialisation down the line - perhaps in the `get-recipe.php` file?

If we give the program a `string_id`, we can pull down a serialised string from the database:

![[Pasted image 20210511212810.png]]

Crucially, this is then deserialised (by the `unserialize()` call), and the wakeup function then reads the recipe from the database based on its ID:

```php
public function __wakeup() {
    //grab from DB when deserialised
    error_log("Woken up - getting from DB");

    $this->get_from_db();
}
```

This suggests we may be able to create a serialised `SecretRecipe` object that grabs the encrypted recipe - but how can we exploit this?

Well, there is an `if` statement that checks the value of the object's `$encrypted` variable - this determines whether the recipe is hashed or not:

![[Pasted image 20210511213257.png]]

When we create a serialised PHP object, we have full control of its variables - so we can set our own object's `$encrypted` variable to `False`, like so:

```php
O:12:"SecretRecipe":4:{s:9:"encrypted";b:0;s:5:"title";s:4:"Test";s:8:"contents";s:4:"Test";s:2:"id";i:1;}
```

This is a serialised PHP object - the `O:12:"SecretRecipe"` string specifies its class, and the strings within the curly brackets specift the class variables. The important ones are  `"id";i:1`, specifying the integer `1` (the ID of the recipe we want to pull down), and `"encrypted";b:0`, specifying we want to set `$encrypted` to false.

If this doesn't make sense, you can learn more about deserialisation with a simpler example from my [[Deserialisation Demo|demo on the topic]].

Let's submit this at the `create-recipe.php` page, by visiting `http://localhost:5000/create-recipe.php?recipe_recipe=O:12:%22SecretRecipe%22:4:{s:9:%22encrypted%22;b:0;s:5:%22title%22;s:4:%22Test%22;s:8:%22contents%22;s:4:%22Test%22;s:2:%22id%22;i:1;}`:

![[Pasted image 20210511213836.png]]

This data isn't available to you when doing the challenge, but as a developer I can see this is successfully inserted into the database:

![[Pasted image 20210511214137.png]]

(*note:* you could read the database creds with the LFI, but it's not accessible remotely - I guess it would be possible to read the database file itself)

Now we just need to trigger our deserialisation, by searching for our recipe string by ID:

![[Pasted image 20210511214013.png]]

Voila - we have a flag!

![[Pasted image 20210511214340.png]]

Well, sort of - it just needs a bit of decoding... If we submit `sesh{c00k13s_sUg4R_bYT3s_bYT3s}`, we get the points!