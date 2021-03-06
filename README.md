<link rel="icon" href="docs/images/favicon.ico">

<img id="logo" src="http://www.pwmckenna.com/img/bittorrent_medium.png" />

# Backbone.Btapp.js
Backbone.Btapp.js provides access to a browser plugin version of uTorrent/BitTorrent via a tree of [Backbone Models and Collections](http://documentcloud.github.com/backbone/ "backbone"). The intent of this project is to allow access to the extensive functionality of a torrent client, from web apps that are as simple as a single Backbone View. Backbone.Btapp.js takes responsibility for getting the plugin installed as well, so you're free to assume that its available. In addition to the local torrent client, you can also easily access a torrent client anywhere else in the world (assume you either configured it originally or have access to that client's username/password).

The project is [hosted on GitHub](https://github.com/pwmckenna/btapp/ "github"), and the [annotated source code](http://pwmckenna.github.com/btapp/docs/backbone.btapp.html "source") is available. An [example application](http://pwmckenna.github.com/nud.gs/ "see it run!") with [annotated source](http://pwmckenna.github.com/nud.gs/docs/nudgs.html "annotation") is also available through [GitHub](http://github.com/pwmckenna/nud.gs/ "source").

#### Downloads & Dependencies
[Nightly Version (0.1)](https://raw.github.com/pwmckenna/btapp/master/backbone.btapp.js "backbone.btapp.js") 28kb, Full source, lots of comments

Backbone.Btapp.js's dependencies are a subset of Backbone.js, so just get that working and you'll be good to go.
[backbone.js](http://cdnjs.cloudflare.com/ajax/libs/backbone.js/0.5.3/backbone-min.js "backbone") ([documentation](http://documentcloud.github.com/backbone/ "backbone"))  

## Introduction

Backbone.Btapp.js builds off of Backbone.js to provide easy access to a torrent client, either on the local machine or a remote machine. The documentation, annotated source, and examples are designed to be as similar to the getting started experience of Backbone as possible. However, the functionality provided through these backbone models and collections is quite extensive and powerful, so its probably worth a look at the [Api Browser](http://pwmckenna.github.com/btapp_api_viewer/ "api") to get an idea of what is possible. Many of the attributes and functions that are made available through this library have examples to give you some idea of what they can be used for. 

## Getting Started

The first step is simply to create a new Btapp object. This will provide you with a javascript representation of a torrent client. You can use these to connect to either your local machine's client or a remote machine (the possibilities are less obvious with remote connections, but this is in my opinion the most powerful and truly unique part of this library)

#### Local Connection
To connect to the plugin client on your local machine...
```javascript
btapp = new Btapp;
btapp.connect();
```
*this code was executed when the page was loaded to ensure that the examples below work even if you forget about this...*

#### Remote Connection 
*(referred to occasionally as the falcon proxy)*
Lets setup the local machine with some proxy credentials and see if we can't connect to it via the falcon proxy. This proxy Btapp object will point to the same torrent client as your original Btapp object (Though you might notice the update times are much longer as it gets routed through a proxy instead of over localhost). You can have unlimited objects all representing the same torrent client, but be careful that they not step on each others toes.

<div class="run" title="Run"></div>
```javascript
username = prompt("Please enter your name","Harry Potter");
password = prompt("Please enter your password","Abracadabra");

btapp.bt.connect_remote(
    function() { }, 
	username,
	password
);
```

Now that we're connected to the falcon proxy we can connect to your current local machine by executing the following code from any computer in the world!

<div class="run" title="Run"></div>
```javascript
remote_btapp = new Btapp;
remote_btapp.connect({  
    'username':username,  
	'password':password
});
```

### Listening for state changes
I'm about to show you how to add and remove data, and here is where the dependency on Backbone makes the most sense. The data in the Btapp object is updates as the state of the torrent client changes, so to listen for objects being added to your torrent client, you can bind add listeners to the corresponding Backbone Models and Collections.  
  
For instance, to show an alert each time a torrent is added to the client, just bind to the torrent list...__Note:__ We're not guaranteed the list of torrents will be there either...so lets listen for that as well.
<div class="run" title="Run"></div>
```javascript
function torrent_list_handler(torrent_list) {
}
var torrent_list = btapp.get('torrent');
if(torrent_list) {
	torrent_list_handler(torrent_list);
} else {
	btapp.bind('add:torrent', torrent_list_handler);
}
```
If this seems a bit messy for you, there is an addition bit of javascript call the [Btapp Listener](#section-4-2 "listener") that you can include that will make these bind add chains much easier to deal with.

### Adding torrents via urls/magnet links
Easy-peasy
<div class="run" title="Run"></div>
```javascript
var url = 'http://www.clearbits.net/get/1766-the-tunnel.torrent';
btapp.get('add').bt.torrent(function() {}, url);
```
And by easy-peasy I mean, wtf?!? So, the base object has an attribute called *add*, which is an object that stores all the functionality for adding stuff to the client (torrents, rss_feeds, rss_filters, etc)...because *add* is a torrent client object, the functions are in the *bt* member. the *torrent* function of the *add* member adds a torrent to the client. Phew. 

If you find yourself down a dark alley needing rss feeds, its almost the same to add one of those.
<div class="run" title="Run"></div>
```javascript
var url = 'http://www.clearbits.net/feeds/cat/short-form-video.rss';
btapp.get('add').bt.rss_feed(function() {}, url);
```


Ok, adding existing content is pretty nice, but your users might want to share their own content...

### Creating torrents
<div class="run" title="Run"></div>
```javascript
btapp.bt.browseforfiles(function () {}, function(files) {
	_.each(files, function(value, key) {
			btapp.bt.create(
				function(ret) {
					alert('called create for ' + value);
				}, 
				'', 
				[value], 
				function(hash) {
					alert('torrent created for ' + value);
				}
			);
	});
});
```
__Warning__: this will launch a file browser on the machine that the client is running on...so if you're connected via falcon you won't be able to see the dialog pop up (but someone might get an unexpected surprise!)

### Deleting torrents
The types that bubble up from the client are either Backbone Collections or Models depending on if they are collections of other torrent client types or not...In the case of the list of torrents, its created as a Collection, which has all those convenient underscore functions available. Yippie!
<div class="run" title="Run"></div>
```javascript
	btapp.get('torrent').each(function(torrent) {
		torrent.bt.remove();
	});
```

## General Concepts

### Btapp arguments
Todo
### Custom events
Todo
### Filters
Todo
### Underlying RESTless API
Todo

## Utilities

The following utilities are designed to get you started working with the library. Part of getting started includes installing the same plugin that your users will need to install in order to use your app (Provided you didn't go through this process when playing with the demo code above). As all these utilities are themselves apps that use this library, clicking on any of these will take you through the process (You only need to install once, regardless of which browsers you use). If you're unfamiliar with all the functionality that the torrent client has to offer, the [api viewer](http://pwmckenna.github.com/btapp_api_viewer/ "api") is probably a good first stop. 

### Api Viewer

The api viewer is a one stop shop for examining the data coming from your torrent client in real time. It is itself a web app that uses backbone.btapp.js, so the [annotated source](http://pwmckenna.github.com/btapp_api_viewer/docs/index.html "annotated source") may be useful to skim through as well. It just creates a backbone view for each bit of info bubbled up from the torrent client.

<a href="http://pwmckenna.github.com/btapp_api_viewer/"><img src="http://pwmckenna.com/img/api_viewer.png"></img></a>
This snapshot was taken while also using the Nud.gs app, which uses labels to categorize its torrents...If you're curious about the dragon file, [check it out here](http://pwmckenna.com/img/dragon.jpg "dragon!")

### Btapp Listener

The Btapp Listener is designed to allow you to listen for the additions of models at any level of the data tree much the same way jQuery uses *delegate* to bind event handlers to DOM elements that don't yet exist.

For instance, to list the name of all the files in all torrents you need to write code similar to the following...

<div class="run" title="Run"></div>
```javascript
var torrents = btapp.get('torrent');
```

### Btapp Plugin
The btapp plugin in managed by plugin.btapp.js, which is hosted on [GitHub](https://github.com/pwmckenna/btapp_plugin "plugin"). It is responsible for loading the necessary plugins and running the torrent client as necessary. It is dynamically loaded by backbone.btapp.js when you create a local Btapp object (unless you specifically pass in username/password arguments your Btapp object will be pointing at your local torrent client).

## Examples
### Nud.gs
### Gibe
