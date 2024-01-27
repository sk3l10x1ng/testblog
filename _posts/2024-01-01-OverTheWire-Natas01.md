---

title: "OverTheWire - Natas - Level 1 -> Level 2"

categories: [Overthewire]

tags: [web,scripting]

---

### Description

Natas teaches the basics of serverside web-security.

Each level of natas consists of its own website located at http://natasX.natas.labs.overthewire.org

Intial Access Credentials
```
Username: natas0
Password: natas0
URL:      http://natas0.natas.labs.overthewire.org
```


# Natas : Level 1 -> Level 2

1. Browse to the URL natas0.natas.labs.overthewire.org

2. View page source of the HTML page to get the password to next level natas1.


Below is the python script to Automate using request library 

```python
import requests as r
import re


def auth(username,password,URL):

	with r.session() as s:
		response = s.get(URL, auth =(username,password))
		content =response.text
		print(re.findall("<!--The password for natas1 is (.*) -->", content)[0])

def main():
	username = password = 'natas0'
	URL = f"http://{username}.natas.labs.overthewire.org/"
	results = auth(username,password,URL)
	
main()
```
