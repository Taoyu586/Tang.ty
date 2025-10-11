---
title: Janus
category: web
event: K17 
date: 2025-09-19
---

## Challenge Details

Someone hacked my space image viewer, but it's 100% secure now! Note: Attacking nasa's API is out of scope for this challenge and you may brute force at most 100 requests at one request per second.

[https://janus.secso.cc](https://janus.secso.cc)

## Attachments

```py
from flask import Flask, request, Response, abort, render_templateimport timeimport socketimport requestsfrom urllib.parse import urlparseapp = Flask(__name__)# Resolve NASA's IP at startupNASA_HOST = "images-api.nasa.gov"@app.route("/")def index():

return render_template("index.html")@app.route("/api")def api():

NASA_IPS = set(socket.gethostbyname_ex(NASA_HOST)[2]).union({'3.175.115.68', '3.175.115.60', '3.175.115.113', '3.175.115.52'})



target_url = request.args.get("url")

if not target_url:

abort(400, "Missing url parameter")



# Parse the target URL parsed = urlparse(target_url)

if not parsed.scheme:

target_url = "https://" + target_url # assume https if missing parsed = urlparse(target_url)



hostname = parsed.hostname

if not hostname:

abort(400, "Invalid URL")



# Prevent users brute forcing our api time.sleep(1)

try:

resolved_ip = socket.gethostbyname(hostname)

except socket.gaierror as e:

abort(400, "Unable to resolve hostname")



# Verify that the url provided resolve's to NASA's IP address if resolved_ip not in NASA_IPS:

abort(403, "URL does not resolve to NASA")



# Fetch and stream the content try:

r = requests.get(target_url, stream=True, timeout=5)

if r.status_code == 429:

abort(429, f"Rate limited by NASA. (This challenge is still solvable)")



r.raise_for_status()

except requests.RequestException as e:

print("failed to fetch", target_url, e)

abort(502, "Failed to fetch data")



return Response(

r.iter_content(chunk_size=8192),

content_type=r.headers.get("Content-Type", "application/octet-stream"),

)
```

```py
from flask import Flask, request, Response, abortapp = Flask(__name__)@app.route("/")def root():

return "the flag will go here"
```

## Docker File

```
# Note that this is equivalent to:
# docker run -p 1337:1337 --dns 1.1.1.1 imagenameservices:

unnamed-dns-rebinding:

build: .

ports:

- "1337:1337"

dns:

- 1.1.1.1
```

## Steps I went through

* First thing I did was looked at the source code.
* I saw the checking code and realised that it only has an initial DNS check and validates everything else after it.
* I then proceeded to look at docker file and saw dns rebinding which is a hint for dns rebinding attack
* Solution was to initially use a nasa IP address which was given then after passing the ip validation change to local 127.0.0.1 for the flag to be outputted then curl it
* I didn't have a dns server so I use a website [https://lock.cmpxchg8b.com/rebinder.html](https://lock.cmpxchg8b.com/rebinder.html)
* Required brute force because of round robin behavior of dns leading to frequent bad gateway and needing the stars to align to get the right combination

## Solution

* Initially use a NASA IP address (one of the given IPs) to pass IP validation.
* After passing validation, change the hostname to resolve to `127.0.0.1` so the server fetches the local endpoint that outputs the flag.
* Use a rebinder (e.g. [https://lock.cmpxchg8b.com/rebinder.html](https://lock.cmpxchg8b.com/rebinder.html)) to achieve the DNS switch.
* Retry (brute force up to the allowed limit) until the request succeeds due to DNS round-robin timing.

## Flag

```
K17{DNS___more_l1ke_d0main_name_shuffl3}
```
