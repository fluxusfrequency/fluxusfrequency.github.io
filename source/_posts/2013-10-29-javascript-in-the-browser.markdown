---
layout: post
title: "Javascript in the Browser"
date: 2013-10-29 19:33
---

Having done many Javascript tutorials and read many Javascript books, I was hoping that I would be better at using the language by now. Since I still had very little clue of how to incorporate JS into developing websites, I decided to reread Chapters 9-11 in Marijn Haverbeke's [Eloquent Javascript](http://www.amazon.com/Eloquent-JavaScript-Modern-Introduction-Programming/dp/1593272820/ref=sr_1_1?ie=UTF8&qid=1383098181&sr=8-1&keywords=eloquent+javascript) (which I highly recommend). These chapters, entitled Web Programming: A Crash Course, The Document Object Model, and Browser Events, respectively, cover the use of Javascript in the browser.

I took notes as I read, and decided to share them here for reference.

<br />

##The Window Object
This is the global object in the context a browser.

###Window#open("url")
Opens a popup.
e.g.: var popup = window.open(‘www.google.com’);

###Window#close
Closes a window.
e.g.: popup.close

<br />

##The Document Object
This property of the window represents the document within the window.

###Document#location#href
Displays information about the current URL. Use this by having a script call it while document is being loaded, causing the written HTML to be inserted into the page.

###Document#write()
Write something (perhaps a variable) to the place in the page where this script is run.

###Document#getElementById(‘id’)
Accesses the node with the given id, which can then be edited.

```javascript example
var picture = document.getElementById(“picture”);
picture.src;
=> “img/bird.png”
picture.src = “img/ostrich.png”;
```

###Document#createElement(“div”)
Creates an element of the given type.
e.g.: header = document.createElement(“H1”);
title = document.createTextNode(“My Title”);

###Document#createTextNode(“text”)
Creates some text. It should then be appended with:
header.appendChild(title);
document.body.appendChild(header);


<br />

##Elements


###Element#onclick
Accesses the element when it is clicked.

###Element#focus
Tells the window to focus on the given element when the code is executed.


<br />

##Nodes


###Element#nodeType
Returns a number: 1 for regular nodes or 3 for text nodes.

###Document#body#firstChild#nodeName;
e.g.: “H1”

###Document#body#firstChild#nodeValue;
e.g.: “Chapter 1: Equipment”

###Document#node#innerHTML = “<p>text</p>”;
Allows you to replace the text in the specified node.

###Node#appendChild(child)
Puts the specified child inside the node.

###Node#parentNode#removeChild(node)
Removes the node from the document.

###Node#setAttribute(“tag”, “attr”)
Sets attributes on a node.

```javascript example
var newImage = document.createElement(“IMG”);
newImage.setAttribute(“src”, “img/yinyang.png”);
document.body.appendChild(newImage);
```

###Node#getAttribute(“attr”)
Gets the specified attribute from the node.
e.g.:newImage.getAttribute(“src”);
=>”img/yinyang.png”;

###Node#style
Accesses the CSS styling for the given node, which can then be set.

```javascript example
picture.style.borderwidth = “4px”;
picture.style.display = “none”;
picture.style.position = “absolute”;
picture.style.width = “400px”;
```

<br />

##Event Handlers

###Node#attachEvent(event, function)
Attaches a function to an event, such “on click”, or “charCode” || “keyCode” (the || is used to cover different browsers).

###Node#addEventListener(event, function, boolean)
Same thing as Node#attachEvent, but for older browsers. The boolean indicates it will bubble up through DOM normally rather than taking priority.

###Node#detachEvent
Opposite of #attachEvent.


###Node#removeEventListener
Opposite of #addEventListener.


###Event Handling For All Browsers

```javascript example
function registerEventHandler(node, event, handler) {
     if (typeof node.addEventListener === “function”)
     node.addEventListener(event, handler, false);
else
     node.attachEvent(“on” + event, handler);
};
```

e.g.: registerEventHandler(button, “click”, function(){print(“click2”);});

###function unregisterEventHandler
Opposite of registerEventHandler.

###Internet Explorer
This pattern is sometimes used for Internet Explorer:
event = event || window.event;




<br />

##Forms

###encodeURIComponent(string)
Applies escapes to the string to make it a legal URI.

###decodeURIComponent(encodedURI)
Transforms an encoded URI back into a string.

###Document#forms
Links to all forms on the page.
e.g.: document.forms.userinfo will access the user info form.

###Form#action
e.g.: “url”

###Form#method
e.g.: “GET”

###Form#elements
Accesses one of the elements in the form.
e.g.: formName.elements.name.value = “Eugene”; would set the name field to that name.

###Form#submit
Manually submits the form to the server.

###Checking form validity:
  1. Change input “submit” into “button”.
  2. Access the form with formName = document.forms.htmlformname.
  3. Check the form method and action by calling formName.method and formName.action.
  4. Check the elements using formName.elements.name, etc.
  5. Write a function to check inputs like so:

```javascript example
function validInfo(form) {
return form.elements.name.value != “” &&
/email_verification_regex/.test(form.elements.email.value);
};
```

  Then set the button’s onClick event like so:

```javascript example
formName.elements.send.onclick = function () {
     if (validInfo(formName)) {
          formName.submit();
     } else {
          alert(“Invalid email”);
     }
};
```

<br />

##Time


###new Date()
Creates a new date object

###Date#Gethours()
<br />

###Date#getMinutes()





<br />

##Timers

###Window\#setTimeOut\(function\(\), ms\)
Calls the function in the first argument after the milliseconds have elapsed.

```javascript example
window.setTimeout(function() {
document.loacation.href = “/spoiler.html”;
}, 5000);
```
This would redirect to spoiler if the user did nothing for 5 seconds.

###Window#clearTimeOut
Cancels a timeout.

###Window#setInterval(function, ms)
Runs the function every time n milliseconds elapse.


<br />

##Navigator

###Navigator#userAgent
Shows the name of the browser being used.
###Navigator#vendor
Shows the vendor of the browser.
###Navigator#platform
Shows info about the computer being used. This can be useful to find out what the browser is and run code specific to that browser (IE6, Chrome, etc).

