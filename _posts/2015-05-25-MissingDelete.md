---
layout: post
title:  "The missing delete"
date:   2015-05-25 23:25:23
tags: c++ qt memory
---


{% highlight C++ %}
void ClipShareRunner::readFromSocket(QString str) {
    updatingClipboard = true;

    QMimeData * mimeData = new QMimeData();
    /*
      Process QString str and add to the information to the QMimeData object.
     */
    QApplication::clipboard()->setMimeData(mimeData);

    updatingClipboard = false;
}
{% endhighlight %}

Any C/C++ programmer who glances over this piece of code will figure out quite quickly that it has some unintended consequences: a memory leak caused by `mimeData = new QMimeData()` without ever calling a corresponding `delete mimeData;`. That's the general pattern for allocating persistent memory on the heap: `new` and `delete`. Modern C++ is very much fond of its RAII principle (Resource Acquisition Is Initialization), an obscure term for a simple principle. Put your `new`s in the constructor, put your `delete`s in your destructor and you are almost guaranteed to never ever have a resource leak.

However, in this case you are not just passing the pointer for the new content of the clipboard, you are also (implicitly) passing on the ownership of the data the pointer points to. Therefore, it is not our responsibility any more to clean up after ourselves. If you attempt to and the object has already been moved, you are greeted by a familiar opponent: the double free error.

We're only left with `new`. The corresponding `delete` is nowhere to be found in our code. Its instruction will be buried somewhere deep inside some compiled library binary, never to be seen again by people: just a few bytes in a sea of apparent pseudo-randomness, making sure that your computer does not crash if you have placed things on the clipboard too many times. Thank you bytes.

In retrospect, the reasoning behind also implicitly passing the ownership of the data and thereby the responsibility to delete it is completely sound: Only the operating system is able to keep track of whenever the clipboard and thereby its mimeData object have been reassigned. In current machines with huge surplusses of memory reallocating is not that efficient anymore: deleting the old object and just allocating a new one. Therefore there is no way to know for the application where the clipboard object has gone and as such it will not be able to clean up after itself.

My few hours spent on debugging this problem have had some positive though: a set of rules I apply for writing my own programs. For instance, any function taking a raw pointer is allowed to use and modify the object, but itself is responsible for releasing the memory. If this is not possible for some reason, I would use a smart pointer or, even better, a reference.


<style>
table{
  border-collapse: collapse;
  border-spacing: 0;
  border:2px solid black;
}
th{
  border:2px solid black;
}
td{
  border:1px solid black;
}
</style>

| Type                | Modify object   | Responsible for cleanup   |
|:------------------- |:---------------:|:-------------------------:|
| Raw pointer         |       x         |            x
| Shared pointer      |       x*        |
| Reference           |       x         |
| Unique pointer      |       x         |
| Constant reference  |                 |

\* only with proper synchronization if necessary
