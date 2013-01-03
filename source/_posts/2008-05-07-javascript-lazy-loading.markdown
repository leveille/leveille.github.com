---
layout: post
title: "JavaScript Lazy Loading"
date: 2008-05-07 20:40
comments: false
categories: JavaScript
---

I just finished a great [Digital Web Magazine](http://www.digital-web.com/) article about improving page load times with [lazy loading techniques](http://www.digital-web.com/articles/improve_page_performance_with_lazy_loading/). The article (by [Jakob Heuser](http://www.digital-web.com/about/contributors/jakob_heuser)) was very well written and described [proximity, timeout, and event](http://www.digital-web.com/extras/lazy_loading/yui_3types.html) based lazy loading. If you continue to read you will see how I was able to reduce page load by 300+kb by implementing lazy loading.

<!--more-->

##What is Lazy Loading

Lazy loading, [described by Martin Fowler](http://www.martinfowler.com/eaaCatalog/lazyLoad.html) in “[Patterns of Enterprise Application Architecture](http://www.martinfowler.com/books.html#eaa)”, is the process by which an object isn’t loaded with data until it is needed. Of course “needed” is subjective to the needs of your application (for example needed in proximity or based on an event). In the context of this article, Jakob Heuser was really talking about lazy loading JavaScript files when they are needed, as opposed to loading them on page load. The loading of the files happens via [XHR](http://en.wikipedia.org/wiki/XMLHttpRequest) (therefore the request is subject to the browser [same origin policy](http://en.wikipedia.org/wiki/Same_origin_policy)). He provides examples of [Gaia Online](http://www.gaiaonline.com/) and [Zimbra](http://www.zimbra.com/) who both reduced their page load by 200k+ after implementing lazy loading techniques.

##Lazy Loading and Me

I have had a few instances over the last month or so where projects I have worked on required some form of lazy loading ([Fowler describes 4 flavors](http://www.martinfowler.com/eaaCatalog/lazyLoad.html)).

###Lazy Initialization

{% blockquote Martin Fowler http://www.martinfowler.com/eaaCatalog/lazyLoad.html Lazy Load %}
uses a special marker value (usually null) to indicate a field isn’t loaded. Every access to the field checks the field for the marker value and if unloaded, loads it.
{% endblockquote %}

In my first example (original blog post - no longer accessible), I had the need to display [EST](http://www.timeanddate.com/library/abbreviations/timezones/na/est.html)/[EDT](http://www.timeanddate.com/library/abbreviations/timezones/na/edt.html) time via JavaScript, regardless of the location of the client. The solution of course is to pull in a timestamp from the server (whose time is in EST/EDT), and feed that to the JavaScript Date constructor. From there, all that is needed is a [setInterval](http://developer.mozilla.org/en/docs/DOM:window.setInterval), called every 1 second, and an increment of the timestamp variable by 1 second.

``` javascript
var TIME = function(){

    var milliseconds = null;

    return {

        construct: function(){
            //lazy load milliseconds
            if (!milliseconds) {

                $.getJSON("time.php", function(r){

                    //ensure that we are getting the proper data
                    if(r[0].zone && r[0].timestamp) {
                       $("#time .zone").html(r[0].zone);
                       milliseconds = r[0].timestamp;
                       TIME.update(new Date(milliseconds));
                       window.setInterval("TIME.construct()", 1000);
                    }
                    else {
                       return;
                    }

                });
            }
            else {
                TIME.update(new Date(milliseconds));
            }
        },

        update: function(time){
            $("#time .time").html(time.toLocaleTimeString());
            milliseconds += 1000;
        }
    };
}
();
```

In the case of the code above, milliseconds starts at null. In the construct method a check is performed to see if milliseconds is null or not. If it is, an XHR request is made to time.php to pull in a timestamp (measured in milliseconds). The first call to construct updates the time display, assigns a value to milliseconds, and sets up the setInterval for TIME.construct(), at a frequency of 1000 milliseconds (1 second). With the second call to TIME.construct(), because the milliseconds variable is already loaded, another XHR request is not necessary. Instead, all that is needed is a call to TIME.update(), and the construction of a new Date object. Piece of cake. So, in this instance, lazy initialization enhanced the application because it provided a means for checking whether or not an XHR request should be made to obtain a timestamp, or if a timestamp already existed.

###ghost

{% blockquote Martin Fowler http://www.martinfowler.com/eaaCatalog/lazyLoad.html Lazy Load %}
A ghost is the real object without any data. The first time you call a method the ghost loads the full data into its fields.
{% endblockquote %}

This second example is more along the lines of those provided in the Digital Web article, however it is even more simplified. In this case, what I actually have is a combination of ghost and lazy initialization (leaning more in the direction of lazy initialization). The application in which I implemented this bit of code required most of the UI to be constructed through JavaScript. Part of the application requires the implementation of [Windows Live Messanger](http://get.live.com/messenger/overview) for tech support purposes. This portion of the application is not visible on page load, and is only available on request. Initially I was loading the JavaScript/CSS/Image files on page load, and the application was suffering a significant performance hit as a result. I don’t have exact numbers, but Windows Live Messanger has in excess of 300+k of JavaScript/CSS/Image files, and I was loading them (via requests for the resources through an iframe) on page load. As you can imagine, the rest of the application was suffering. It was sluggish and unresponsive for seconds at a time.

{% blockquote %}
Through the implementation of lazy loading (tied to a click event), I was able to do away with this initial 300+k load, and garner significant page responsiveness.
{% endblockquote %}

The goodies:

``` javascript
var msnPayLoad = null;

...

function setupLiveMessangerChat() {

   if(config.playerMode.isLive)
   {
      $('a#activate-chat').toggle(
         function () {

            //lazy load msn javascript
            //this payload is only needed if the user actually clicks on the live support link
            if(!msnPayLoad)
            {
               msnPayLoad = activateMsnJavascript;
               msnPayLoad();
            }

            $('#support').show();
            $('#activate-chat').html('Close Live Support');
         },
         function () {
            $('#support').hide();
            $('#activate-chat').html('Open Live Support');
         }
      );
   }
   else
   {
      $('#support').hide();
   }
}

...

function activateMsnJavascript() {
   //Grab the necessary scripts for live chat
   $.getScript("http://settings.messenger.live.com/controls/1.0/PresenceButton.js", function(){
      $.getScript("http://messenger.services.live.com/users/etc", function(){
         $("#support div.support").load("core/section-includes/live-messanger-chat.inc.htm?stamp=" + getStamp());
      })
   });
}
```

In the case of the above example, I have a null payload variable, which is loaded when the activate chat link is clicked the first time. During this time a call is also made to activateMsnJavascript(), and 300kb+ of files join the party.

##I’m so loaded man

So, I hope if you’ve gotten anything from this article it is that employing lazy loading techniques doesn’t have to be hard. What it might bring to your application table is well worth the time it will take to investigate this very interesting topic.
