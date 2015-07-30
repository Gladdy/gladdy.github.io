---
layout: post
title:  "Demystifying Node.js - part 1"
date:   2015-07-26 21:10:32
category: C++
---

Node.js: it's quick to write, easy to understand and performs very well in applications which are I/O bound for performance. The high level of abstraction offered by Node.js also has a disadvantage. Without looking at the C source code, which is something I cannot imagine many Node.js developers will do, the inner workings can appear to be almost magic. Node.js handles all the nitty-gritty low-level stuff for you: ensuring that the proper callbacks are executed at the proper time with proper values for the variables. This is great and all, but if the server application has to perform actual computation the performance suffers due to the interpreted and dynamic nature of JavaScript. The usual choice is to write the high-performance part a different language which can be compiled to an object file (C/C++) and then add the compiled program as an addon, a process which is documented [here](https://nodejs.org/api/addons.html).


### Why?
As a little side weekend-long side project, I decided to rewrite the underlying code of a small subset of Node.js in C++ without the use of any external libraries. One might wonder: why? Node.js already works perfectly well, but as previously mentioned the high abstraction level distracts from the underlying and supporting mechanisms and one of the goals is to display a simplified version of the inner workings of Node.js. Furthermore, by writing the top-level code in C++ over JavaScript it becomes trivial to perform high-speed computation directly in the server application. Lastly, it's similar to the early days of Facebooks' HipHop PHP compiler, which took their entire PHP codebase and generated a single binary server application. This, optimized, single-binary approach obviously has advantages in performance, ease of dependency management and deployment. Lastly, C++ is known for being very complex which comes with the huge amount of control you have and it includes rules which might be hard to grasp. JavaScript is quite the opposite. However, C++ does not have to be hugely complex which is why I'm trying to mimic the easy-to-understand nature of JavaScript in vanilla C++ (that is, without any `#define`s to hide some syntax).


### Restrictions
My own restriction to avoid external libraries and keep the codebase simple has some consequences. It would have been easy to use Boost.ASIO or libuv (which is used internally by Node.js). However, not relying on libraries entails writing a custom asynchronous engine which can be very complicated and hard to understand. Therefore, I decided to use a very much simplified engine, which does not support arbitrary nesting of callback functions. This has some consequences for the designs, but the arising problem: keeping track of state can be solved by using an external structure containing some additional data for each socket connection. Furthermore, low-level concepts such as UNIX socket handling and threading in C++ will also appear in this design.

### JavaScript or C++?
<style type="text/css">
    pre {
        font-size: 12px;
    }
    </style>
<div class="row">
<div class="col-md-6">
{% highlight C++ %}
TCPServer server ([&](Socket &socket) {

  console.log("Client connected");
  console.log("address: " + socket.remoteAddress());
  console.log("port:    " + socket.remotePort());

  socket.on("data", [&, socket](Data data) mutable {
    //Handle the received data
  });
  socket.on("end", [&, socket](){
    //Handle the client disconnect
  });
});
server.listen(1338);
{% endhighlight %}
</div>
<div class="col-md-6">
{% highlight JavaScript %}
var server = net.createServer(function(socket){

  console.log("Client connected");
  console.log("address: " + socket.remoteAddress);
  console.log("port:    " + socket.remotePort);

  socket.on("data", function(data){
    //Handle client message
  });
  socket.on("end", function(){
    //Handle client abort
  });
});
server.listen(1338)
{% endhighlight %}
</div>
</div>
<div class="row text-center">
Spot the differences!
</div>

<br>
At first glance the snippet above appears to be JavaScript. It's easy to write and easy to understand. Almost everything is specified in terms of callback functions. It features the familiar Node.js pattern of `socket.on("event", callback). However, it is not JavaScript. It's C++. The most notable difference between JavaScript and C++ in this snippet is the syntax for functions, which are first-order citizens in JavaScript whereas C++ has to rely on its implementation of lambda functions. The syntax is quite a lot more verbose in C++, but in turn it offers a lot more control!




