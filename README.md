# Web Hacking II - Cookies, SOP, XSS and CSRF
February 24, 2017

### Tools
- [EditThisCookie](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=en)

### What's the point of using cookies?
- To track users and to keep them logged in
- To remember what's in your shopping cart
- To make websites stateful
  
### Setting Cookies
- Key=value pair: aka cookie crumb
- Expiration date: if not set, cookie is Session cookie
- Domain: allowed to access cookie
  - who can access?
    - bar.foo.com - Yes
    - *.com - No
    - foo.com/bar - Yes
  - is .foo.com different from foo.com? NO!!
- What pages should the cookie set with? path
- Flags
  - HostOnly: Can only be read by same domain that set it, foo.example.com sets it so bar.example.com cannot read it if HostOnly=1
  - Session: A session cookie expires when the user navigates away from the website
  - Secure: Cookie can only be sent over https
  - HTTPOnly: Cannot be accessed with JavaScript (prevents most XSS)
  
  
  ```
  +-----------+                                                       +-----------+
  |           |                                                       |           |
  |           |     POST \login                                       |           |
  |           |     Host: cs558web.bu.edu                             |           |
  |           |     username=attacker&password=l33th4x                |           |
  |           |  ------------------------------------------------->>  |           |
  |           |                                                       |           |
  |           |     HTTP/1.1 303 See Other                            |           |
  |           |     Location: http://cs558web.bu.edu/project2/        |           |
  |           |     Set-Cookie: authuser="!28734y8273"                |           |
  |   USER    |  <<-------------------------------------------------  | WEBSERVER |
  |           |                                                       |           |
  |           |     GET /project2/ HTTP/1.1                           |           |
  |           |     Host: cs558web.bu.edu                             |           |
  |           |     Cookie: authuser="!28734y8273";                   |           |
  |           |  ------------------------------------------------->>  |           |
  |           |                                                       |           |
  |           |    HTTP/1.1 200 OK                                    |           |
  |           |                                                       |           |
  |           |    <html>...Protected HTML...</html>                  |           |
  |           |  <<-------------------------------------------------  |           |
  +-----------+                                                       +-----------+
  ```
  
 ### zombie cookies
 - cookei that's automatically recreated after being deleted. This is accomplished by storing the cookie's content in mult locations, such as Flash local shared object, html5 web storage
 
### 3rd party cookies
- used for ad tracking
- firefox, ie, opera, chrome do allow third-party cookies by default, as long as the tird party website has Compact Privacy Policy published. Newer versions of Safari block third-party cookies.
- to check for them, inspect the page on chrome (seen under developer tools for sarafi)
- how do these cookies get included?
    - have iframe, and look what's included in that page. see that it has some javascript, which 
    - take look at local storage, all of these tracking sites have some local storage files stored. 
    - cookies can track you across different websites
    - 'put script in this page, based on number of views, we will give you $$$'
    
### In the news
- EU requires users agree to have cookies tored on their page
- let's try it
  - go to google.com
  - change location to france (i used pia
  - see change at http://www.ipfingerprints.com/
  - refresh google.com
  
### Dynamic web pages
- rather than static html, web pages can be expressed as a program, say written in javascript:

### javascript
- powerful web page programming language
scripts are embedded in web pages retured by web server
scripts are executed by brower

confining the pwower of javascript scripts
given all that power, browsers need to make sure js scripts don't abuse it
for example, donw' want sc

### same origin policy
broswers provide isolations for js scripts via the same origin policy (SOP)
simple version:
broswer associates web page elements (layout, cookies, events) with a given origin 
sop = only scripts re

### origin definitions
- port 80
- port 443

### same origin policy
- should it be able to access cookies? green for yes, red for no, yellow - differs by browser
  - examples go here
- are subdomains included in sop policy? yes
- what about local storage? should example.com be able to set local storage on a.example.com?
- implementation differs in the wild: http://www.filldisk.com/

### XSS - subverting the same origin policy
- have some attack vector where you can insert a script, and it gets run in the context of that page

### two types of XSS (Cross-Site Scripting)
- there are two main types of xss attacks
- in a stored (or "persistant") XSS attack, 
### stored xss
### stored xss: summary
Notes (2) requires
saw demo in lab

### protexting servers against XSS (OWASP)
how protext against cross site scripting?
- 

### web accessed with side effects
when the broswer goes to render the content, makes get request to get any resources it doesn't already have included in the page. make get request to url -- doesn't matter what's returned. on 

-- application -- boomerang. injects javascript in gmail client. email gets tracked. showed when other person opens it 
-- has image display none -- broswer makes get request to that image url, which will run something on the server that does something. but user can't see it so doesn't know what's going on
-- 
