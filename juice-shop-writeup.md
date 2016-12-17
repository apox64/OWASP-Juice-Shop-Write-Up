# Write-up: OWASP Juice Shop Challenges (v2.19.1)

* [juice-shop](https://github.com/bkimminich/juice-shop) by [@bkimminich](https://github.com/bkimminich)
* write-up by [@apox64](https://github.com/apox64)
* influenced by [7 Minute Security (Episode #230)](https://7ms.us/7ms-230-pentesting-owasp-juice-shop-part-1/)
* current status:
  * v2.19.1
  * 36/37 challenges solved (97%)
  * Continue Code: laHquqhrtJcOInTvC6sjFEi8fESZUnHVuLh2tDcNIETMCvslFxiBfbSJUJH3uJhYtKcgIQTz

## Tools
* Kali Linux (2016.2) tools:
  * Burp Suite
  * dirb
  * sqlmap
* Google Account for challenges 28 & 34 (OAuth2.0)
* _search engine (Internet)_

## Notes
* Start Burp and set a proxy to 127.0.0.1, port 8080 (this is the Burp proxy). You can use the FireFox Plug-In 'FoxyProxy Basic' to quickly switch on/off using a proxy. Have Burp ready in the background, since many challenges can be solved with this tool.
* You might need to add the Burp CA certificate to the known certificates of your browser.

## Challenges
* [1 Star Challenges](#1-star-challenges)
* [2 Star Challenges](#2-star-challenges)
* [3 Star Challenges](#3-star-challenges)
* [4 Star Challenges](#4-star-challenges)
* [5 Star Challenges](#5-star-challenges)


<!------------------------------- 1 Star Challenges ------------------------------->

## 1 Star Challenges

#### 1: "Find the carefully hidden 'Score Board' page."
* no pre-solved challenge necessary

Go to `http://192.168.99.101:3000/#/score-board` to show the scoreboard. This was solved by looking at the HTML source code of the website.

#### 2: "Provoke an error that is not very gracefully handled."
* no pre-solved challenge necessary

Many ways to solve this. Just search for `' something`.

> This should be properly error-handled showing something like "No results found.", but it doesn't.

#### 3: "XSS Tier 1: Perform a reflected XSS attack."
* no pre-solved challenge necessary

Just put the given string in the search field to reflect the alert `<script>alert("XSS1")</script>` to solve the challenge.

#### 4: "Get rid of all 5-star customer feedback."
* solve challenge 8 first

Manually delete all 5 star customer-feedback from the `/administration` page (1 total).

#### 5: "Access a confidential document."
* no pre-solved challenge necessary

Use directory buster (dirb) to list folders on the server.
`dirb "http://192.168.99.101:3000/"`
Look at the ones with status "code 200". /ftp contains a few files. Download the file "acquisitions.md" from there.

#### 6: "Access the administration section of the store."
* prequisites: log in as any user

Just browse to `/#/administration`. This was solved by just guessing the name from the challenge.

#### 7: "Give a devastating zero-star feedback to the store."
* no pre-solved challenge necessary

Go to "Contact Us", turn on Burp Intercept, then post a feedback giving any rating and change the value in the body of the intercepted request to `"rating":0`.


<!------------------------------- 2 Star Challenges ------------------------------->

## 2 Star Challenges

#### 8: "Log in with the administrator's user account."
* no pre-solved challenge necessary

You get the administrator's email from challenge 6.
Bypassing the authentication works with a simple manual SQL injection like `something' OR 1=1 --` in the email field. You will be logged in as the administrator.


#### 9: "Log in with the administrator's user credentials without previously changing them or applying SQL Injection."
* solve challenge 18 first

Crack the hash for admin@juice-sh.op retrieved from challenge 18: "Retrieve a list of all user credentials via SQL Injection".
You can use an online md5 cracker like [hashkiller](https://hashkiller.co.uk/md5-decrypter.aspx) to get the clear text password.

#### 10: "Access someone else's basket."
* prequisites: log in as any user

Use Burp Intercept to change the value in the Header `GET /rest/basket/4` to any other number.

#### 11: "Access a salesman's forgotten backup file."
* solve challenge 5 first

To bypass the error when trying to download a .bak file from the ftp, we can use a nullbyte (%00) to pretend downloading something that's allowed. We need to URL encode the `%` as `%25` followed by the null byte `00` and an allowed ending.
`/ftp/coupons_2013.md.bak%2500.pdf` let's us download the .md.bak file. Remove the .bak ending and open the file with a text editor.

#### 12: "Inform the shop about an algorithm or library it should definitely not use the way it does."
* solve challenge 32 first

The `rot13` algorithm is definitely *not* secure to encode data.
Also, the `z85` algorithm is not secure.

#### 13: "Order the Christmas special offer of 2014."
* solve challenge 18 first

* prequisites: log in as any user

When playing around with the succeeding payload from **challenge 18**'s SQL injection, one will find that the search for `q=something')) UNION ALL SELECT NULL,id,description,price,NULL,NULL,NULL,NULL from products--` displays all products. You can see that our desired product has the id = 9.

Now intercept the request with Burp when you place any other item in your basket and change the value of `"ProductID:"` to `9` to place the Christmas Special Offer 2014 into your basket.

Check out and the challenge is solved.

<!------------------------------- 3 Star Challenges ------------------------------->

## 3 Star Challenges

#### 14: "Log in with Jim's user account."
* solve challenge 18 first

Crack the hash for jim@juice-sh.op retrieved from challenge 18: "Retrieve a list of all user credentials via SQL Injection".
You can use an online md5 cracker like [hashkiller](https://hashkiller.co.uk/md5-decrypter.aspx) to get the clear text password.

#### 15: "Log in with Bender's user account."
* solve challenge 34 first

Works exactly like challenge 34 (Option: `X-User-Email`)

#### 16: "XSS Tier 2: Perform a persisted XSS attack bypassing a client-side security mechanism."
* no pre-solved challenge necessary

We need to put `<script>alert("XSS2")</script>` somewhere and get it executed, so that it doesn't get filtered out by some JavaScript security mechanism on the client side. Simply pasting it in "Contact Us" doesn't work.
Another try could be to register a user who has this script in his credentials somewhere and thus _storing_ it on the server.
Putting it directly into the "Email" field does not work (Error: "Email address is not valid.").
But what if we intercept the packet (again with Burp) _after_ it was checked by the client-side JavaScript and inject the script there?
`{"email":"user@domain.com<script>alert("XSS2")</script>","password": ...`
This will return an error, since the first `"` before the `XSS2` ends the string of the email value.
We have to escape the quotes in the script like this to solve the challenge:
`{"email":"user@domain.com<script>alert(\"XSS2\")</script>","password": ...`

#### 17: "XSS Tier 3: Perform a persisted XSS attack without using the frontend application at all."
* no pre-solved challenge necessary

Okay, so if we want to change something somewhere, we can't use HTTP GET, but we need to PUT something somewhere. We will forge our own packet that will update the description of a product containing the script.

When we search for a product in the shop we can see that the request looks like this: `GET /rest/product/search?q=O-Saft`.

If then open the product and request more information (Eye-Symbol), we can see a request `GET /api/Products/8?d=Thu%20Nov%2010%202016 HTTP/1.1`

If we open `http://192.168.99.100:3000/api/Products/8` directly, we get a JSON file displayed. As you can see, this is the same data, but it has no graphical frontend. We will use the API to send a PUT request to that updates the description of a product with our script.

Let's forge a packet ... (You can use Burp Repeater to make re-sending it easier.)

```
PUT /api/Products/8 HTTP/1.1
Host: 192.168.99.100:3000
Content-Type: application/json;charset=utf-8
Content-Length: 276
Connection: close

{"description":"<script>alert(\"XSS3\")</script>O-Saft is an easy to use tool ..."}
```

> _You can "turn off" the alert again by just sending the same PUT request, but removing the script, so it doesn't get executed anymore._

#### 18: "Retrieve a list of all user credentials via SQL Injection"
* no pre-solved challenge necessary

We will use the following parameters:
* URL : http://192.168.99.100:3000
* Path : /rest/product/search/
* Parameter : q
* Database : sqlite (open information from the website, but sqlmap can find this out too)

So the command look like this:

`sqlmap -u 'http://192.168.99.100:3000/rest/product/search?q=something' -p 'q' --dbms='sqlite'`

I had to increase the risk level to 2 to get results:

`--level=2`

My succeeding payload is: `q=something')) UNION ALL SELECT NULL,'qzbxq'||'mZqQCoizQFtquCOMVIMxLCpyaAjHXowBYRJPxgIJ'||'qzkvq',NULL,NULL,NULL,NULL,NULL,NULL-- FlvR` (this might look different for you)

We can now extract the general schema behind this query:
* we don't need anything that comes after the comment `--`
* we will add `from users` before the comment to retrieve the data from the users table
* we can substitute `'qzbxq'||'mZqQCoizQFtquCOMVIMxLCpyaAjHXowBYRJPxgIJ'||'qzkvq'` for `email` or `password` (this is just guessing of the column name) to see the results printed on the screen
* Simply by trying and replacing `NULL` values with guessed column names, we can forge a payload that returns the emails and their hashed passwords:

This payload solves the challenge:

`q=something')) UNION ALL SELECT NULL,email,password,NULL,NULL,NULL,NULL,NULL from users--`

You can now put the hashes into an online md5 cracker like [hashkiller](https://hashkiller.co.uk/md5-decrypter.aspx) and get the clear text passwords.

#### 19: "Post some feedback in another users name."
* log in as any user

Go to "Contact Us" as any user logged in.
Again, turn on Burp's Proxy Intercept to be able to edit the packet before actually sending it.
Enter a different value for `"UserId":` to solve the challenge.

#### 20: "Place an order that makes you rich."
* log in as any user

Log in as any user (a self-registered user also works). Turn on Burp Intercept. Then put some stuff in your basket. Take a look into the intercepted request and change the value `"quantity":` to something negative and forward the request.

#### 21: "Access a developer's forgotten backup file."
* solve challenge 11 first

Download "package.json.bak" with the same technique as described in challenge 11: "Access a salesman's forgotten backup file." This package.json file contains the used libraries and their versions.

> This is a gold nugget for us ...

#### 22: "Change the href of the link within the O-Saft product description into `http://kimminich.de`."
* solve challenge 17 first

```
PUT /api/Products/8 HTTP/1.1
Host: 192.168.99.100:3000
Content-Type: application/json;charset=utf-8
Content-Length: 244
Connection: close

{"description":"O-Saft is an easy to use tool
to show information about SSL certificate and
tests the SSL connection according given list
of ciphers and various SSL configurations.
<a href=\"http://kimminich.de\" target=\"_blank\">More...</a>"}
```
> _NOTE: You might need to remove some `\r` (line breaks) that were added for formatting._

#### 23: "Inform the shop about a vulnerable library it is using. (Mention the exact library name and version in your comment.)"
* solve challenge 22 first

package.json.bak file. Just remove the .bak at the end and open the file to see the packages the developer of the shop used. Under "dependencies" you can find all the used libraries.

For example [snyk.io](https://snyk.io) offers a web search for known vulnerabilities:

| Name | Version | Vulnerable?
|:---|:---|:---:
|body-parser|1.15|[![Known Vulnerabilities](https://snyk.io/test/npm/body-parser/1.15.2/badge.svg)](https://snyk.io/test/npm/body-parser/1.15.2)
|colors|1.1|[![Known Vulnerabilities](https://snyk.io/test/npm/colors/1.1.2/badge.svg)](https://snyk.io/test/npm/colors/1.1.2)
|cookie-parser|1.4|[![Known Vulnerabilities](https://snyk.io/test/npm/cookie-parser/1.4.3/badge.svg)](https://snyk.io/test/npm/cookie-parser/1.4.3)
|cors|2.8|[![Known Vulnerabilities](https://snyk.io/test/npm/cors/2.8.1/badge.svg)](https://snyk.io/test/npm/cors/2.8.1)
|errorhandler|1.4|[![Known Vulnerabilities](https://snyk.io/test/npm/errorhandler/1.4.3/badge.svg)](https://snyk.io/test/npm/errorhandler/1.4.3)
|express|4.14|[![Known Vulnerabilities](https://snyk.io/test/npm/express/4.14.0/badge.svg)](https://snyk.io/test/npm/express/4.14.0)
|express-jwt|5.1|[![Known Vulnerabilities](https://snyk.io/test/npm/express-jwt/5.1.0/badge.svg)](https://snyk.io/test/npm/express-jwt/5.1.0)
|glob|5.0|[![Known Vulnerabilities](https://snyk.io/test/npm/glob/5.0.15/badge.svg)](https://snyk.io/test/npm/glob/5.0.15)
|hashids|1.1|[![Known Vulnerabilities](https://snyk.io/test/npm/hashids/1.1.1/badge.svg)](https://snyk.io/test/npm/hashids/1.1.1)
|helmet|2.3|[![Known Vulnerabilities](https://snyk.io/test/npm/helmet/2.3.0/badge.svg)](https://snyk.io/test/npm/helmet/2.3.0)
|jsonwebtoken|7.1|[![Known Vulnerabilities](https://snyk.io/test/npm/jsonwebtoken/7.1.9/badge.svg)](https://snyk.io/test/npm/jsonwebtoken/7.1.9)
|morgan|1.7|[![Known Vulnerabilities](https://snyk.io/test/npm/morgan/1.7.0/badge.svg)](https://snyk.io/test/npm/morgan/1.7.0)
|multer|1.2|[![Known Vulnerabilities](https://snyk.io/test/npm/multer/1.2/badge.svg)](https://snyk.io/test/npm/multer/1.2)
|pdfkit|0.8|[![Known Vulnerabilities](https://snyk.io/test/npm/pdfkit/0.8.0/badge.svg)](https://snyk.io/test/npm/pdfkit/0.8.0)
|sanitize-html|1.4.2|[![Known Vulnerabilities](https://snyk.io/test/npm/sanitize-html/1.4.2/badge.svg)](https://snyk.io/test/npm/sanitize-html/1.4.2)
|saucelabs|1.3|[![Known Vulnerabilities](https://snyk.io/test/npm/saucelabs/1.3/badge.svg)](https://snyk.io/test/npm/saucelabs/1.3)
|sequelize|1.7|[![Known Vulnerabilities](https://snyk.io/test/npm/sequelize/1.7/badge.svg)](https://snyk.io/test/npm/sequelize/1.7)
|sequelize-restful|0.4|[![Known Vulnerabilities](https://snyk.io/test/npm/sequelize-restful/0.4/badge.svg)](https://snyk.io/test/npm/sequelize-restful/0.4)
|serve-favicon|2.3|[![Known Vulnerabilities](https://snyk.io/test/npm/serve-favicon/2.3/badge.svg)](https://snyk.io/test/npm/serve-favicon/2.3)
|serve-index|1.8|[![Known Vulnerabilities](https://snyk.io/test/npm/serve-index/1.8/badge.svg)](https://snyk.io/test/npm/serve-index/1.8)
|socket.io|1.4|[![Known Vulnerabilities](https://snyk.io/test/npm/socket.io/1.4/badge.svg)](https://snyk.io/test/npm/socket.io/1.4)
|sqlite3|3.1|[![Known Vulnerabilities](https://snyk.io/test/npm/sqlite3/3.1/badge.svg)](https://snyk.io/test/npm/sqlite3/3.1)
|z85|0.0|[![Known Vulnerabilities](https://snyk.io/test/npm/z85/0.0/badge.svg)](https://snyk.io/test/npm/z85/0.0)

Send one of the vulnerable libraries and the version number in the comment box in the shop to solve the challenge.

> Although socket.io and sequelize-restful also have known Vulnerabilities, currently only reporting `sanitize-html 1.4.2` or `sequelize 1.7` solve the challenge.

#### 24: "Find the hidden easter egg."
* solve challenge 11 first

Download "eastere.gg" with the same technique as described in challenge 11: "Access a salesman's forgotten backup file."

#### 25: "Travel back in time to the golden era of web design."
* no pre-solved challenge necessary

When you look at the source of the **HOT**-button you can see that the button is in: `css/geo-bootstrap/img/hot.gif`

Open the Console of your browser and enter the following command: `document.getElementById("theme")`
This will return the theme that's currently in use: `href="bower_components/bootswatch/slate/bootstrap.min.css"`

You need to set a new href to the geo-bootstrap to activate the css. When you search for "geo-bootstrap" on the internet, you can find a github Project and find a path on where the .css file is located : `geo-bootstrap/swatch/bootstrap.css`

Now let's combine everything together ... We need to **set** the **theme** to `css/geo-bootstrap/swatch/bootstrap.css`.

In the console the command `document.getElementById("theme").setAttribute("href", "css/geo-bootstrap/swatch/bootstrap.css")` solves the challenge.

> NOTE: Changing it back to normal with href: `document.getElementById("theme").setAttribute("href", "bower_components/bootswatch/slate/bootstrap.min.css")` or simply reloading the page.

#### 26: "Upload a file larger than 100 kB."
* logged in as any user

Let's create file with random content that has a .pdf ending and which is smaller than 100 kb.
``` bash
base64 /dev/urandom | head -c 99900 > random.pdf
```
The resulting file will have a size be 99,9 kb.

Intercept and forward until you find the right packet, scroll down where all the text is and add at least another 101 characters (_or just mash on the keyboard for a while_).

#### 27: "Upload a file that has no .pdf extension."
* solve challenge 27 first

Repeat the first steps from challenge 27 and intercept the packet again. Change `filename="random.pdf"` to something else. Solved.

#### 28: "Log in with Bjoern's user account without previously changing his password, applying SQL Injection, or hacking his Google account."
* solve challenge 6 first

Log in with some valid Google Account account `Log in with Google`. Intercept the packets with Burp. Look at the packet that contains a `POST` to `/api/Users/`. In this packet, apparently the new user get's put into the Juice Shop user database. You can see that the password here is not the one you used for the Google login (the Juice Shop never sees this password). So it had to be created somehow ... After playing around a little with different encodings, you can find out that this password is the mail address, encoded with base64.
Let's try to apply this to bjoern's mail address we can get from the `/adminstration` section. Encode with base64 and login.

<!------------------------------- 4 Star Challenges ------------------------------->

## 4 Star Challenges

#### 29: "XSS Tier 4: Perform a persisted XSS attack bypassing a server-side security mechanism."
* solve challenge 24 first

Let's see what the developer implemented on the server side: "sanitize-html": "1.4.2"
There is an issue that sanitizing is not used recursively, so if we put a script into a script, only the outer script will be santized and the inner script will still run.

`<script>alert("XSS4")</script>`

We will put the same string after the first `<` and the `s` to make it look like this:

`<<script>alert("XSS4")</script>script>alert("XSS4")</script>`

Plus, we need another end tag `</script>` before the last closing script tag to make it execute:

`<<script>alert("XSS4")</script>script>alert("XSS4")<</script>/script>`

You can post this under "Contact Us" to solve the challenge.

#### 30: "Wherever you go, there you are."
* no pre-solved challenge necessary

The ribbon in the top right corner "Fork me on Github" contains the link `/redirect?to=https://github.com/bkimminich/juice-shop`. As always, when tampering with URLs, we should try null-bytes `%00`. When going to `/redirect?to=/%00https://github.com/bkimminich/juice-shop`, we give the redirect a valid target `https://github.com/bkimminich/juice-shop`, but keep it from actually going there.

#### 31: "Change Bender's password into slurmCl4ssic."
* solve challenge 15 first
* be logged in as Bender

When changing the password, we can intercept a packet with a GET request like this:
`GET /rest/user/change-password?current=idontknow&new=slurmCl4ssic&repeat=slurmCl4ssic`

Let's remove the current parameter and see if it still gets accepted ...
`GET /rest/user/change-password?new=slurmCl4ssic&repeat=slurmCl4ssic`

Yes, the password still changes. So now we changed the password without knowing the old one.

> NOTE: The md5 hash of "slurmCl4ssic" is 06b0c5c1922ed4ed62a5449dd209c96d. Although Hashkiller currently has no entry for the reversal of this hash, the unsalted "slurmCl4ssic" password is not a safe password as it might be added to the database one day.

#### 32: "Apply some advanced cryptanalysis to find the real easter egg."
* solve challenge 25 first

In the eastere.gg file from /ftp there is a string `L2d1ci9xcmlmL25lci9mYi9zaGFhbC9ndXJsL3V2cS9uYS9ybmZncmUvcnR0L2p2Z3V2YS9ndXIvcm5mZ3JlL3J0dA==`.
Decoding this with base64 (Burp Decoder can do this), we get `/gur/qrif/ner/fb/shaal/gurl/uvq/na/rnfgre/rtt/jvguva/gur/rnfgre/rtt`. That looks like a path. It doesn't lead nowhere, but we have to be close (because we can see the slashes). This could be a simple shift cypher. After trying a little bit, we can find out that this is a rot-13 shift.
Decoding it, we get `the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg`. Following this link, the challenge is solved. And we get a huge orange in JavaScript as a reward.

#### 33: "Retrieve the language file that never made it into production."
* solve challenge 5 first

Change the language a few times and look at the HTTP request history.
You can see that the files are located in the /i18n/ folder and that they have a .json ending.
`GET /i18n/sv.json HTTP/1.1`

We can use dirb to run through all the language files, but we need a full word list.
Let's generate our own:
``` python
#!/usr/bin/python

from itertools import product
from string import ascii_lowercase

length = input("Enter length of words to be generated: ")
print "Generating wordlist with length", length

f = open('wordlist.txt', 'w')

for count, i in enumerate(product(ascii_lowercase, repeat=length)):
    f.write(''.join(i) + '\n')
print "words generated: %d" % count

f.close()

```

Let's generate all words with length **3** and then run `dirb` with the following parameters:

``` bash
dirb "http://192.168.99.100:3000/i18n/" wordlist.txt -X .json
```
We can find `tlh.json` with this method and solve the challenge (tlh is Klingon).

#### 34: "Exploit OAuth 2.0 to log in with the Chief Information Security Officer's user account."
* solve challenge 6 first

Login with a valid Google Account and intercept the communication with Burp.
Look at the packet containing the `POST` to `/rest/user/login` and manually add the Header: `X-User-Email: ciso@juice-sh.op`. You can leave the Body like it is `({"email":"something@gmail.com","password":"c29tZXRoaW5nQGdtYWlsLmNvbQ==","oauth":true})`.

```
POST /rest/user/login HTTP/1.1
Host: 192.168.99.100:3000
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:45.0) Gecko/20100101 Firefox/45.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json;charset=utf-8
Referer: http://192.168.99.100:3000/
X-User-Email: ciso@juice-sh.op
Content-Length: 88
Connection: close

{"email":"something@gmail.com","password":"c29tZXRoaW5nQGdtYWlsLmNvbQ==","oauth":true}
```

<!------------------------------- 5 Star Challenges ------------------------------->

## 5 Star Challenges

#### 35: "Forge a coupon code that gives you a discount of at least 80%."
* solve challenge 11 first

The file "coupons_2013.md" in the /ftp Folder contains some codes.
These are encoded with "z85" [bkimminich/z85-cli](https://github.com/bkimminich/z85-cli), also in "package.json.bak" under "Dependencies".
I wrote this little script that runs through the "coupons_2013.md" file and outputs the decoded strings.

``` bash
#!/bin/bash
file="/root/Downloads/coupons_2013.md"
while IFS= read -r line
do
	eval "z85 -d \"$line\""
done <"$file"
```

You should be able to see some sort of pattern here : "MMMYY-DD" (MonthYear-Discount)
Create your own code from this month, for example "NOV16-90".

Encode it with z85:

``` bash
z85 -e "NOV16-90"
```

This string will give you a z85 encoded value of `pes[Bhz3{y`. Enter it in the coupon option and solve the challenge.

#### 36: "Fake a continue code that solves only (the non-existent) challenge #99."
* Solve challenge 21 first.

In the package.json from the ftp Server we can find that `hashids` was used. Some searching on the web brings us to this site : `http://hashids.org/`. On the demo site, we can play around with the hashing mechanism. A number can be specified (default is `8`) that determines the length of the output hash. The continue codes are 60 chars long, so let's replace 8 with 60. Let's try decoding any currently valid continue code by changing the variable `numbers` ...
``` JavaScript
var numbers = hashids.decode("1ZQmmZnR751JxwB4KjXVoQMzD8AWqH3u4G6LYgl29OvP3aENeqWpbykroVl3");
```
This continue code gets decoded to `[1,2,34]`. These look like the numbers of currently solved challenges!

> You should note your numbers (your currently solved challenges) somewhere to be able to restore your progress later.

Let's try and encode only the number `99` with the same parameters _("this is ...", 60, "abcdef ...")_.
``` JavaScript
var id = hashids.encode(99);
```
The resulting continue code `69OxrZ8aJEgxONZyWoz1Dw4BvXmRGkKgGe9M7k2rK63YpqQLPjnlb5V5LvDj` solves the challenge.

> To restore your progress, just forge another continue code with the numbers you noted down before.

#### 37: "Log in with the support team's original user credentials without applying SQL Injection or any other bypass."
* Not solved yet.

Download the kdbx file from the FTP Server (`/ftp/incident-support.kdbx%2500.md`).
The account Email is `support@juice-sh.op` (found on `http://192.168.99.100:3000/#/administration`).
