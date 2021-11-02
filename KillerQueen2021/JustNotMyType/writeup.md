# Just Not My Type - Write up
This challenge give us a website to access and the source code as the following.

Source Code : 
```html
<h1>I just don't think we're compatible</h1>

<?php
$FLAG = "shhhh you don't get to see this locally";

if ($_SERVER['REQUEST_METHOD'] === 'POST') 
{
    $password = $_POST["password"];
    if (strcasecmp($password, $FLAG) == 0) 
    {
        echo $FLAG;
    } 
    else 
    {
        echo "That's the wrong password!";
    }
}
?>

<form method="POST">
    Password
    <input type="password" name="password">
    <input type="submit">
</form>
```

## Analysis
Looking at the code, we can obviously see that we will get the flag when ` strcasecmp($password, $FLAG) == 0` . However, we don't know the `$FLAG` so we have to exploit the `strcasecmp` function somehow. 
After doing some trial & error and some googling, I've learn that `strcasecmp(a,b)` will return `null` when either `a` or `b` is not string. In addition, `null` has the same numerical value as `0`. Since we use `==` operator instead of `===` here, then `null == 0`.

Therefore, we can change the `$password` from our input in the payload of the POST request after we submitted a value like this (In this case, I used burpsuite to intercept and edit the payload),

```$password[]=hello``` 

You can substitute `hello` to anything, the point here is that we want to change `$password` to array instead of string so that `strcasecmp` function return `null` as we want to. Then the website will return the flag.

Flag : ```{no_way!_i_took_the_flag_out_of_the_source_before_giving_it_to_you_how_is_this_possible}```