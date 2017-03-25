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
  - **HostOnly:** Can only be read by same domain that set it, foo.example.com sets it so bar.example.com cannot read it if HostOnly=1
  - **Session:** A session cookie expires when the user navigates away from the website
  - **Secure:** Cookie can only be sent over https
  - **HTTPOnly:** Cannot be accessed with JavaScript (prevents most XSS)
  
  
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
  
### Zombie Cookies
 - cookie that's automatically recreated after being deleted.
 - How is this done?
    - by storing the cookie's content in mult locations, such as Flash local shared object, HTML5 Web storage, and other client-side and even server-side locations. 
    - When the cookie's absence is detected, the cookie is recreated using the data stored in these locations.           
 
### 3rd Party Cookies (3P)
- Used for ad tracking
- Firefox, Internet Explorer, Opera, and Google Chrome do **allow third-party cookies by default**, as long as the tird party website has Compact Privacy Policy published.
    - Newer versions of Safari block third-party cookies.
- To check for them...
    - Right click, then click 'Inspect' in Google Chrome
    - For Safari, [do this](https://support.apple.com/kb/PH21414?viewlocale=en_LA&locale=en_LA)
- How do these cookies get included?
    - Have iframe, and look what's included in that page. See that it has some javascript.
    - Take a look at local storage; all of these tracking sites have some local storage files stored. 
    - cookies can track you across different websites
    - Companies say: 'Hey, put this script in the page. Based on the number of views, we will give you $$$'
  
```
<iframe name="fb_xdm_frame_http" frameborder="0" allowtransparency="true" allowfullscreen="true" scrolling="no" id="fb_xdm_frame_http" aria-hidden="true" title="Facebook Cross Domain Communication Frame" tabindex="-1" src="http://staticxx.facebook.com/connect/xd_arbiter/r/1FegrZjPbq3.js?version=42#channel=fa383a9b0dbf38&amp;origin=http%3A%2F%2Fwww.huffingtonpost.com" style="border: none;"></iframe>
```
```
HTTP/1.0 200 OK
Set-Cookie: JEB2=58D...;expires=Sat, 23 Mar 2019 16:11:20 GMT; 
domain=atwola.com;path=/
```

### In the news
- EU requires users agree to have cookies tored on their page
- Let's try it!
  - Go to google.com
  - Change location to France (Sean used PIA)
  - See change at http://www.ipfingerprints.com/
  - Refresh google.com
  
### Dynamic web pages
- Rather than Static HTML, web pages can be expressed as a **program**, say written in *javascript*:

```
<title>Javascript demo page</title>

<font size=30>
Hello, <b>
<script>
var a = 1;
var b = 2;
document.write("world: ", a+b, "</b>");
</script>
```

### Javascript
- Web page programming language
- Scripts are embedded in web pages returned by web server
- Scripts are executed by browser

### Confining the Power of Javascript Scripts
- Browsers need to ensure that JS scripts don't abuse their power
    - don't want a script sent from **hackerz.com** web server to read cookies belonging to **bank.com**
    - ... or read keystrokes typed by user at **bank.com**

### Same Origin Policy (SOP)
- broswers provide isolations for JS scripts via the **Same Origin Policy** 
- Simple version:
    - broswer associates web page elements (layout, cookies, events) with a given *origin*
      - origin: web server that provided the page/cookies in the first place
      - Identity of web server is in terms of its hostname, e.g., **bank.com**
- SOP = **only scripts received from a web page’s origin have access to page’s elements**

#### Origin Definition
- http://www.example.com:80/dir/page/htm
  - **Protocol:** http://
  - **Host:** www.example.com
  - **Port:** :80
- **http** operates on port *80* so http://example.com/ = http://example.com:80/
- **https** operates on port *443* so https://example.com/ = https://example.com:443/	

- Should the following be able to access cookies from http://www.example.com? 

|                           Site                         | Yes/No/Maybe |
| ------------------------------------------------------ | ------------:|
http://www.example.com/dir/page2.html                    | Yes          |       
http://v2.www.example.com/dir/other.html                 | No           |
http://username:password@www.example.com/dir2/other.html | Yes          |
http://www.example.com:81/dir/other.html                 | No           |
https://www.example.com/dir/other.html                   | No           |
http://en.example.com/dir/other.html                     | No           |
http://example.com/dir/other.html                        | No           |
http://www.example.com:80/dir/other.html                 | Maybe        |

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
