---
layout: post
title: "PhantomJS Creating &amp; Connecting to Server"
date: 2014-06-15 16:37
comments: true
categories: unit testing phantomjs server static file server css js img html
---

At my work we are a Java shop. So spinning up a server is quite a process. Additionally our static front end files are spread out across the system and we use RequireJS (ugh) to wrangle everything. Then in order to test we were running a selenium test to hit the test_runner page and wait for mocha to run and the tests. This is so unbelievably slow and quite hectic when it comes to managing relative paths of our config.

So to make things simpler I decided it'd be in my best interest to use PhatonmJS. I decided to spin up a server using `var server = require('webserver').create();` and manage the requests and just route and serve up all the correct files. I thought this would work however I ran into a snag. Here is some sample code below

<!-- more -->


```
var server = require('webserver').create();
var url = 'localhost:8000'
server.listen(url, function(){ 
	console.log('someone connected');
})

page.open(url, function(status) {
	console.log(status);
});
```
Well I thought this would work based upon the docs but I was wrong. `status` would be success but the log on the server would never trigger. So I started mixing it up, here are a few things I tried.

`var url = '127.0.0.1:8000'`

```

server.listen(8000);

page.open('127.0.0.1:8000');

```

```

server.listen(8000, {keepAlive: true});

```

After spending an hour debugging and attempting to not flip a table I figured it out. Apparently the solution was to add `http`. Example:

```

server.listen(8000, {keepAlive: true}, function() {
	console.log('sucess');	
});

page.open('http://127.0.0.1:8000');

```

So I got it working, I was sending the files down but on larger files I was continually getting PhantomJS throwing `Parser errors`. I was setting proper content types and with `keep alive` connections you have to send the content lengths. The solution? Get rid of `keepAlive:true`. I was under the assumption it was necessary, it was also in my code when I actually got a successful connection so I assumed it was necessary. In the end it was very simple and probably a result of some minor idiocy on my part and slight lack of documentation.


If you ever need to have your unit test server up CSS/JS/HTML, even mock Rest API end points, server up mocked data (there are better ways to do this) then here is your PhantomJS solution.


```

var PORT = 8000,
	url = 'http://127.0.0.1:' + PORT,
	server = require('webserver').create(),
	page = require('webpage').create(),
	fs = require('fs'),
	system = require('system'); // This was used to take in args to change what PORT to connect to but not necessary for most people

var contentTypes = {
	
	'css': 'text/css',
	'html': 'text/html',
	'js': 'application/javascript',
	'png': 'image/png',
	'gif': 'image/gif',
	'jpg': 'image/jpeg',
	'jpeg': 'image/jpeg'
};

	server.listen(PORT, function(req, res) {


		var filePath = fs.workingDirectory + fs.separator + req.url.split('/').join(fs.separator), // make it OS agnostic
			fileName = req.url.split('/').shift().split('?')[0], //remove any query string
			ext = fileName.split('.').shift(),
			fileContent = '';

		res.statusCode = 200;
        res.headers = {
        	"Cache": "no-cache", 
        	"Content-Type": contentTypes[ext] || 'text/html' //no content type?
        };


		if (fs.isReadable(filePath)) {
			fileContent = fs.readFile(filePath);
		} else {
			res.statusCode = 404;
			//maybe 501? Your error codes may vary
		}

		res.write(fileContent);
		res.close();
		
	});


	page.open(url, function(status) {

		if (status !== 'success') {
			phantom.exit(1);
		}


	})



```

This is very basic and assumes your running in the root of your files you need to server. In my case I wasn't and had to do some additional parsing and mapping of the URL to get the correct filepath but it should be a decent start for you.
