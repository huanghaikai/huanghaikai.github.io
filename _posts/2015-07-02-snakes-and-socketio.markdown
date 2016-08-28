---
layout: post
title:  "Snakes and Socket.IO"
date:   2015-07-02 11:15:00
categories: recurse-center js go
comments_enabled: true
---

Last Wednesday during my commute on the train, I coded a game just to see how long it would take. The game I coded is a web version of the classic Snake Game and can be checked out [here][1].

I showed it to [Brad][2], and he wanted to make a multiplayer version of it. The same day,  [Aishwarya][4] and [Heidi][3] gave a workshop on creating a [Web multiplayer game using SocketIO][5].

## Implementing the game ##

We began with a small design session, in which we agreed that it should be Web based (this meant we could re-use parts of my single player version) and that we should use [Go][7] for the back-end.

We tried to use TCP/IP sockets, but unfortunately they are not available in the front-end JS world.

We tried to then see an alternative, we even thought on re-making the front-end as a Java Applet in order to have TCP/IP. We ended up finding an alternative which was to use [Socket.IO][6].

Oh! and there is a SocketIO implementation for go called [go-socket-io][8].

## Socket.IO & WebSockets ##

Socket.IO is based on [WebSockets][10]. This is a technology that the Web Server must support which allows "Real Time Web Applications" by providing a connection which is full-duplex communication channel. This must be supported by the Web Server and by the Browser.

The WebSocket connection is requested by the browser by using a special header: `Connection: Upgrade`.

In the case the Web Server accepts, the HTTP connection is replaced by a Web Socket connection re-using the same port (HTTP 80 or HTTPS 443).

### The game protocol ###

For the game we went for the simplest protocol we could think of. The board after our design discussion can be seen below:

![DesignSession](/public/multiplayer-snake-design.jpg "Design of the Protocol image")

All the client code which handles the communication fits in less than 20 lines of code:

```javascript
	var socket = io();
    if (!playing) {
      socket.emit('join game','j');
      playing = true;
    }
    socket.on('init setup',function(data){
    var currentState = data;  
	if (game != undefined){ game.setStatus(currentState);}
      else {game = new Game(currentState);}
      socket.emit('ready', 'y');
	 });
    socket.on('tick', function(){
        game.tick();
    });
    socket.on('die', function(){
        alert('You just died!');
    });
    window.addEventListener('keydown', function(event) {
      switch (event.keyCode) {
        case 37:
            socket.emit('change', 'LEFT');
            break;
        case 38:
            socket.emit('change', 'UP');
            break;
        case 39:
            socket.emit('change', 'RIGHT');
            break;
        case 40:
            socket.emit('change', 'DOWN');
            break;
      }
    }, false);
```

### What we could have improved ? ###

Currently, we are passing many times the `init setup` message. We pass it at each direction change, when a snake dies, when a new player joins the game, and even if someone eats the fruit!

What really bothers me is that we are doing the same computation on the client and on the server at the same time. In some way, we have duplication of knowledge and we are not being "DRY".

This makes me wonder: **How many other client-server applications have duplicate knowledge accross boundaries?**.

**How many times we repeat code (or knowledge) just because it is the fastest solution we think at the moment?**

## Conclusions ##

It was pretty fun to work with [Brad][2] once again! Also, WebSockets seems to be performing ok (I expected it to be slower since it is a higher level abstraction than raw TCP sockets).

Unfortunately, I think is my last project at [RC][9] since it is my last day.

[1]: http://pietro.menna.net.br/web_snake
[2]: https://twitter.com/daveypocket
[3]: https://twitter.com/HeidiKasemir
[4]: https://twitter.com/aishwarya923
[5]: https://github.com/hkasemir/RCworkshopMultiPlayerGame
[6]: https://github.com/Automattic/socket.io
[7]: https://golang.org
[8]: https://github.com/googollee/go-socket.io
[9]: http://recurse.com/
[10]: https://www.websocket.org/aboutwebsocket.html
