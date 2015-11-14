---
layout: post
title: "What are service worker ?"
date: 2015-11-14 15:35:21 +0530
comments: true
categories:
- service worker
- html5
- offline
- cache
- flipkart
- snapdeal
---

There was buzz among my developer friends on twitter about recent announcement of [flipkart lite](https://twitter.com/flipkart_tech/status/663929833307574272) and [snap-lite](https://twitter.com/anandc/status/664833643215392769). When you have millions of people using highly unrealiable network we need this kind of initiative from big companies. Hoping others will follow it.

But as software developer I am more interested in how they built this experience. Flipkart shared their story [here](http://tech-blog.flipkart.net/2015/11/progressive-web-app/). They use lots of cool features which are implemented in chrome. One of thing which stand out is use of `service worker`. Which bring me to question `What are service worker?`

### Specification 

Service worker is standard build by W3C working group. Here is how specification explain service worker

```
The service worker is a generic entry point for event-driven background processing in the Web Platform that is extensible by other [specifications](http://www.w3.org/TR/service-workers/#extensibility).
```
you can read full specification [here](http://www.w3.org/TR/service-workers/) if you want to read 10's pages.If you want brief introduction to build your application using it then carry on reading this post.

Let's asked ourself following questions about `service worker` and get answer for each one them

1. Why should I use service worker ?
2. What are service worker ?
3. How to use service worker ?


### Why shuold I use service worker ?

Here is list of where you can use service worker.

1. You can control frontend caching using service worker. No need to use (ambiguos)[http://alistapart.com/article/application-cache-is-a-douchebag] `App Cache`. So your website will load fast.
2. You can build `offline first` app
3. You can implement native app like `background sync` feature in web app.
4. `push notification` to your website. finally :clap:

I will be writing blog post explaning every use case in near future. But first lets get our basic right.

### What are ServiceWorker ?

here are some basic features of ServiceWorker

1. ServiceWorker run in it's own thread in browser, it has it's own context
2. ServiceWorker is not tied to particular page but has it's own scope. (explain below)
3. ServiceWorker script does not run in main thread in browser. So it can not access DOM.
4. ServiceWorker can run even if webpage is not open (thats why it can do push notification and background sync)
5. ServiceWorker are `event driven` which means browser can start and terminate them depend on events
6. You can start service worker only on `https` (reason's are explain below)

### How to use ServiceWorker ?

After knowing all usecase and features of ServiceWorker you must be excited to use your awesome web app. But before that we need to know life cycle of ServiceWorker.

### Life cycle of ServiceWorker

ServiceWorker can be one of step below

1. Downloading
2. Installing
3. Error
4. Activated
5. Idle
6. Terminated

Let's figure our what are these states stand for

1. Downloading

	When we asked browser to `register` script as ServiceWorker it gets downloaded. Every refresh will check if 
	ServiceWorker script is changed or not. If changed then it will initiated ServiceWorker `Update` sequence (explained below).

2. Installing

	after script downloaded browser installs it. In which it will create thread for script to run

3. Error

	any uncaught exception in script will bring ServiceWorker in `Error` state. Do not worry next time you refresh page it will do through same step again.

4. Activated
	
	If all things go right ServiceWorker goes to activation state. After that you can get all `fetch` event from page.

5. Idle

	As name suggest when there are nothing to do in script, ServiceWorker goes to idle state.

6. Terminated

	Browser control ServiceWorkers so it can terminate service worker any time. And you won't get any notification before that. But whenever event create in your web app browser will activate service worker.		


Here is simple flow chart of ServiceWorker's lifecycle

->![life cycle of ServiceWorker](/static/img/sw-lifecycle.png)<-

### Getting started

here simple way to register your script as service worker 

```
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/my-app/sw.js', {
    scope: '/my-app/'
  }).then(function(reg) {
    console.log('Yey!', reg);
  }).catch(function(err) {
    console.log('Boo!', err);
  });
}
```

as code snippet says, We are first checking whether browser support ServiceWorker, [here](http://caniuse.com/#feat=serviceworkers) is list of browser who support ServiceWorker.

If users browser support then we ask browser to download script from `/my-app/sw.js` path and register it as service woker. Api is promise base so we have to give success and failure callback.

Another important thing is registering script as service worker with `/my-app/` scope. ServiceWorker controls page which has url `/my-app/*`. If scope is not pass while registeration default is taking from path of script.

### Events in ServiceWorker

remember I said service worker run in seperate thread than main thread of your web app. So main thread  communicate service worker using events. 

following are event you can handle in Service Worker

1. `install`

	```
	self.addEventListener('install', function (event) {
		console.log('service worker install successfully')
	})	
	```

	This event fired when first time or new version of service worker script get installs. We can do some cache/db initialization here. 

2. `activate`
	
	```

	self.addEventListener('activate', function (event) {
		console.log('service worker in activated !!!')
	})

	```
	you can event saying service worker reach to 'activate' state. And you can do good to go for your bussiness logic

3. `fetch`

	```	

	self.addEventListener('fetch', function (event) {
		console.log(event.request.url)
	})

	This one of the most powerful feature service worker has. 

	Remeber I mentioned following things about service worker 
	1. Service worker control page for given scope. 
	2. Service worker only works on `https` connection except localhost (use for testing)

	It is because this feature.

	Service worker get fetch event on every request made by your web app. except following times

	1. iframes
	2. service worker registration fetch calls
	3. any fetch request from service worker itself to avoid infinite loop

	So in other times basically service worker can work as proxy to your web app. take following example

	```
	self.addEventListener('fetch', function(event) {
	  fetch('/some-advertiser/log_url?url=' + event.request.url) //logging every outgoing request
	  return fetch(event.request.url)
	});

	```
	If some how hacker hijack your servicer worker script then it can do real damage. In order to avoid
	`man in middle` attack `https` condition is in force.


4. `message`
	
	```
	self.addEventListener('message', function (event) {
		//after getting message we send received data back to all control page
		self.clients.matchAll().then(function(clients) {
		  clients.forEach(function(client) {
		    console.log(client);
		    client.postMessage(event.data);
		  });
		});
	})

	``` 

	till now we are just know communication from page to service worker. But `postMessage` api gives bidirectional communication channel. 

	So in above example we are listening on message sent from controlled page using `postMessage` api, after getting message we just send message to all pages service worker controlled. 

	This is very useful for logging activity on page.


### am I controlling page now ?

I have mentioned lot of time in this post about 'controlling page' from service worker because of power full api we have.

But main question is when will your service worker control pages ?

As I mention in `Getting Started` that we need to register service worker. So When you register worker first time document is all ready loaded so service worker can not control page. But after you refresh page 
service worker takes control. 

Things get little interesting when you trying to update service worker scripts.

### Updating Service Worker 

Updating service worker code is tedious. 

Remember service worker control all web page for given scope in addition to that we can have multiple page opens from same origin which can be in different state this makes updating service worker tedious.

Updating service following steps followed by `chrome` update. 
So on every page refresh browser will check for service worker script update. It check bytewise for update of script. If new version of script found browser will download it in background and install it. But it wont make it active as multiple tab from same origin can use differnt service worker script which is not desirable. 

So to activate newly updated service worker we have to close all tab from browser. Another way is if old service worker got garbage collected then next time when event fire from page it will start new updated service worker.

The easiest way at the moment is to close & reopen the tab (cmd+w, then cmd+shift+t on Mac), or shift+reload then normal reload.

[here](https://www.youtube.com/watch?v=VEshtDMHYyA) is video which demostrate how updating service worker works.

### Debugging Service Worker

Service worker run in different thread in browser so debugging it is different than normal web page

in chrome you can go to `chrome://serviceworker-internals` to see all service worker running in your browser. In addition to that you can inspect service worker code. It is very powerful features. [Here](https://www.chromium.org/blink/serviceworker/service-worker-faq) is place where you can get full info debugging service worker in chrome.


firefox is still building debugging tool for service worker [here](https://bugzilla.mozilla.org/show_bug.cgi?id=1003097) is issue for that. But in firefox nightly you can do to `about:serviceworkers` to get service worker info (this is true when blog was written)

### Conclusion 

Web are always one step behind native apps on mobile. But `service worker` will really put native and web app neck to neck.

You can stay updated with `service worker` by reading [this](https://jakearchibald.github.io/isserviceworkerready/) website.

To get more info about service worker please follow following people on github and twitter

1. Jake Archibald [github](https://github.com/jakearchibald) [twitter](https://twitter.com/jaffathecake)
2. Alex Russell [github](https://github.com/slightlyoff) [twitter](https://twitter.com/slightlylate)

Thanks for reading !!! Keep hacking !!! :smiley: :smiley: :smiley:











