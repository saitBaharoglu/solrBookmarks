# solrBookmarks
Solr is an open source enterprise search platform built on Apache Lucene. So, it can be modified to fit your needs. However as an easy solution, Google Chrome's Bookmark can be used to make the repetative work to consume less from your time. Bellow you find the steps to create a very useful bookmark to use for the Solr Query UI.  

# How Can Bookmarks help?
Bookmarks on Google Chrome can take you to a specific URL with a single click, additionally it is possible to write a JS code in the URL field, so when clicked the JS code will be executed.

The following example shows an alert with "Hello World!" message when clicked. Add it to your bookmarks and make sure it works before continuing to other parts.
```js
javascript: !function () {
  alert("Hello World!");
}();
```

# Open Solr (new tab)
This bookmark opens the Solr Query page in a new tab. This helps in two ways:
1- Most of the times you need to open Solr while you are working on another tab.
2- The opened tab will have no previous tab, so it will prevent you to press the back button of chrome, so the configirations will not be lost.

DO NOT forget to write the IP address of the Solr server in URL bellow.
```js
javascript: !function () {
  window.open("http://##.###.##.##:####/solr/#/logmng/query", "_blank");
}();
```
# Default Configirations
This bookmark setts default values to the text fields.

q: Query
fq: Filter Query
sort: Sort :P
fl: Fields to include in the result

IMPORTANT Note: read the notes at the end of this page before trying the code bellow!
```js
javascript: !function () { 
    const date = new Date(); 
 const formatedDate = date.toISOString(); 
 const fl = "controllerName,url, requestTime, responseTime, responseMessage, " + 
    "requestMessage, responseHeader, requestHeader, httpStatus, methodName, "+ 
    "modulName, userName, isFault"; 
  
    document.querySelector('#fq').value = "responseTime:{\"" + formatedDate + "\" TO *}"; 
    document.querySelector("#q").value = "userName:\"sait.baharoglu\""; 
    document.querySelector("#sort").value = "requestTimedesc"; 
    document.querySelector("#fl").value = fl; 
    document.querySelector('#fq').value = "requestTime:{\"" + formatedDate + "\" TO *}"; 
}();
```

# fq Dates
The bookmarks in this section makes it soooo easy to add an query filter date range of your current time as the "responseTime"! 

IMPORTANT Note: read the notes at the end of this page before trying any of the codes bellow!

- Filter the query for the past 30 seconds:
```js
javascript: !function () { 
    const date = new Date(); 
    date.setTime(date.getTime() - 30 * 1000); 
    const formatedDate = date.toISOString(); 
  
    document.querySelector('#fq').value = "responseTime:{\"" + formatedDate + "\" TO *}"; 
}();
```


- Filter the query for the past minute:
```js
javascript: !function () { 
    const date = new Date(); 
    date.setTime(date.getTime() - 60 * 1000); 
    const formatedDate = date.toISOString(); 
  
    document.querySelector('#fq').value = "responseTime:{\"" + formatedDate + "\" TO *}"; 
}();
```


- Filter the query for the past 10 minute:
```js
javascript: !function () { 
    const date = new Date(); 
    date.setTime(date.getTime() - 10 * 60 * 1000); 
    const formatedDate = date.toISOString(); 
  
    document.querySelector('#fq').value = "responseTime:{\"" + formatedDate + "\" TO *}"; 
}();
```


- Filter the query for the past 1 hour:
```js
javascript: !function () { 
    const date = new Date(); 
    date.setTime(date.getTime() - 60 * 60 * 1000); 
    const formatedDate = date.toISOString(); 
  
    document.querySelector('#fq').value = "responseTime:{\"" + formatedDate + "\" TO *}"; 
}();
```


- Filter the query starting from the start of your working day (9:00):
```js
javascript: !function () { 
    const date = new Date(); 
    date.setHours(6, 0, 0, 0); 
    const formatedDate = date.toISOString(); 
    document.querySelector('#fq').value = "responseTime:{\"" + formatedDate + "\" TO *}"; 
}();
```

# fq date by the first query result in the page
This bookmark will see the previous query's result, takes the 'responseTime' and adds it to the fq.

IMPORTANT Note: read the notes at the end of this page before trying the code bellow!

```js
javascript: !function () { 
    const a = $("span:contains('responseTime')")[6]; 
    const b = a.nextElementSibling; 
    const c = b.children[0]; 
    const t = c.innerText.slice(1, -5); 
    document.querySelector('#fq').value = "responseTime:{\"" + t + "999Z\" TO *}"; 
}();
```

# Export Query as a CURL raw text
The following bookmark will ask you to enter the port, and the X-Auth-Token as well, once you enter them you will not be asked again to enter them.
If you want to update the port or X-Auth-Token simply open a new Solr tab!
The exported CURL text is copied to Clipboard, alternatively you can find it attached at the end of the Solr page.

Note: the default request is assumed to be POST, you can change it to GET in the CURL text.
IMPORTANT Note: This bookmark gets the data through a new connection, this means that if a new query data is generated then the query on the page and the result of the CURL will not match, therefore you have to Excute Query again.

```js
javascript: !function () { 
    var xmlHttp = new XMLHttpRequest(); 
    var url = document.querySelector('#url').text; 
    xmlHttp.open("GET", url, false); 
    xmlHttp.send(null); 
    var res = xmlHttp.responseText; 
    var jres = JSON.parse(res); 
    let i = prompt("which element?", 0); 
    var req = jres["response"]["docs"][i]; 
    if (!window._baharoglu_port) { 
        window._baharoglu_port = prompt("What is the port?", '8902'); 
    } 
    if (!window._baharoglu_XAT) { 
        window._baharoglu_XAT = prompt("What is your X-Auth-Token?", 'XXXXXXXXXXXX'); 
    } 
    var subUrl = req["url"][0]; 
    if (subUrl.startsWith("/cus/")) { 
        subUrl = subUrl.substr(4); 
    } 
    var curl = "curl --location --request POST 'http://localhost:" + window._baharoglu_port + subUrl + "' \\\n"; 
    curl += "--header 'X-Auth-Token: " + window._baharoglu_XAT + "' \\\n"; 
    curl += "--header 'Content-Type: application/json' \\\n"; 
    curl += "--header 'operation-id: yourOperationIDGoesHere' \\\n"; 
    if (req.requestMessage) { 
        curl += "--data-raw '" + req.requestMessage[0] + "'"; 
    } 
    if (!window._baharoglu_dummy) { 
        window._baharoglu_dummy = $('<TEXTAREA>'); 
    } 
    window._baharoglu_dummy.val(curl).appendTo('body').select(); 
    document.execCommand('copy'); 
}();
```

# IMPORTANT NOTES
Whenever one of the bookmarks above make any change to the UI, the change will not take a place in when Excute Query pressed, until manually you edit each of the TextFields. therefore I removed one whitespace from each of the fields, as the following:
- q: userName:"sait.baharolgu": add a space between the key and the value.
- fq: requestTime:{"2021-06-30T07:18:56.366Z" TO \*}: add a space between the key and the value.
- sort: requestTimedesc: add a space like: requestTime desc.
- fl: add a space right after the first comma.

Whenever you do any change do not forget to add/remove anything manually from the textfield.

