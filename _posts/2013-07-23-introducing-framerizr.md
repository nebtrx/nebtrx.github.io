---
title: Introducing Framerizr
categories:
- hacks
tags:
- ASP.NET
- Cross Domain
- easyXDM
- iframes
- Realtime web
---

About a month ago I was trying to embed an ASP.NET MVC 3 App inside an external web using iframes, that sometimes useful nature’s aborted baby. 

It was all happy until I found me wishing Darth Vader kill me using telekinesis because the iframe was showing up .. guess what? .. a vertical scroll. That fact unleashed a sequence of events which I will summarize as short as possible (because I almost hate writing codeless text): 

_I had to find a solution which allow me to dynamically adjust the height of the host iframe, in a way I can iframe(or how I like to call it: framerize) any web of my site without worrying about html body content height (especially when it changes during AJAX Requests )._

So, the solution had to satisfy the following requisites:
1.  It had to be in Javascript(reasonable).
1.  It had to works in realtime.
1.  It had to be X Domain.
1.  It had to detect DOM chages which affect HTML Body Height.
1.  It had to require minimal changes to existent views.

After a little of research I found the best way to achieve that was monitoring the size of the enclosed html content using [DOM Mutation Observers](https://dvcs.w3.org/hg/domcore/raw-file/tip/Overview.html#mutationobserver) for detecting height changes in the html body and [easyXDM Sockets](http://easyxdm.net/wp/2010/03/17/setting-up-your-first-socket/) which allow me to perform X Domain Javascript calls for pushing this info to the iframe in the external web.

Server side script fragment:

```javascript
pushData: function () {
  framerizr.server.socket.postMessage($("iframe").contents().find('body').height() + 20);
},

...

MutationObserver = window.MutationObserver || window.WebKitMutationObserver;

var observer = new MutationObserver(function (mutations, observer) {
// fired when a mutation occurs

// there is a time between detecting the change and change render process difficult to catch,
// so wait until DOM renders up!.
setTimeout(function () { framerizr.server.pushData(); }, 500);

// define what element should be observed by the observer
// and what types of mutations trigger the callback
observer.observe(
  document.getElementsByTagName("iframe")[0].contentDocument.getElementsByTagName("body")[0], {
    subtree: true,
    attributes: true
  }
);
```

Client side script fragment:

``` javascript
new easyXDM.Socket({
  // remote uri for html content served
  remote: remote,
  // iframe container
  container: $container[0],
  onMessage: function(message, origin) {
    // resizing
    $container.find('iframe').height(message + "px");
    $container.height(message + "px");
  }
});
```

With this, requisites 1 to 4 were checked.

But there was a problem: _easyXDM Sockets_ get closed when you browse away from the iframe src. It was a low punch, but after a neurons ball dance I decided to ad an additional controller `FramerizrController` in my ASP.NET MVC Web Application to serves any requested URI of my own application inside a 100% dimension wide iframe which I called _transitional iframe_.This way the transitional iframe is used to serve the html content while keeping opened the easyXDM socket, otherwise once you browse away the connection will be closed and height monitoring stops working. An advantage of this approach was requisite 5 was achieved as a collateral bonus.

For better understanding and downloading the full sources refer to [Github project](https://github.com/nebtrx/framerizr). Or if you feel too lazy today and you just want a solution ready to use, you may install the Nuget Package `Framerizr.MVC`(authored by me), which does all that for you, just typing the following in your Package Manager Console:

```
PM> Install-Package Framerizr.MVC
```

That's all for today folks.  
See ya in the next one.
