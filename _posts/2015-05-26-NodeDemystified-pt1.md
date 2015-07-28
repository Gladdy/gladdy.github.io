---
layout: post
title:  "Demystifying Node.js - part 1"
date:   2015-05-26 21:10:32
category: C++
---

Node.JS

{% highlight JavaScript %}
var server = net.createServer(function(socket){
  socket.on("data", function(data){
    //Handle client message
  });
  socket.on("end", function(){
    //Handle client abort
  });
});
server.listen(12345)
{% endhighlight %}

{% highlight C++ %}
TCPServer server ([](Socket socket){
  socket.on("data", [](Data data){
    //Handle client message
  });
  socket.on("end", [](){
    //Handle client abort
  });
});
server.listen(12345);
{% endhighlight %}
