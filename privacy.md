Too many internet users take their online privacy for granted. Even developers, I have found, often have very little knowledge of just how much information about themselves is leaked to the websites that they visit.

In an attempt to remedy this in some small way, I've created this page to demonstrate just a few of the things that I believe *you* should be aware of.


Information You've Given Me
---------------------------
```
IP Address: {{ ip }}
Web Browser: {{ browser }}
Operating System: {{ os }}
Last Site Visited: {{ referer }}
Cookies: {{ cookies }}
Geolocation: {{ location }}
```

If these fields are non-informational, congratulations - you've limited what information your browser sends to the websites you visit. Otherise, don't panic! I don't keep any of it. You should, though, be aware of what information you're sharing each time you visit a website.


Google Tracks You
------------------
Beyond the information your browser sends to servers, advertising companies do their best to profile users by putting tracking scripts into the ads they place on a variety of sites. They do this so that they can serve content more relevant to you, increasing the likelyhood that you'll click on the ad and thereby increasing their revenue. Do you have a Google account? [Check out your profile](https://www.google.com/settings/ads/onweb/).

Here's what Google has to say about me: 
```
Age: 18-24
Gender: Male
Language: English
Interests:
    Bicycles & Accessories
    Business & Productivity Software
    C & C++
    Computer & Video Games
    Computers & Electronics
    Data Sheets & Electronics Reference
    Electronic Components
    Email & Messaging
    Linux & Unix
    Mobile & Wireless
    News
    Performing Arts
    Software
    Web Design & Development
    Writer's Resources
```

While I'm not terribly ashamed of this list, it's eye opening to realize that every single search you make is logged and analyzed. 

I don't mind search engine companies collecting user data and using it to deliver targeted ads. I may even *want* to see an ad for a new model of an oscilloscope or a promotional offer on domain names. If it stopped there everything would be okay.

The real problem with all of this tracking comes with how the data is handled afterward. Google's [privacy policy ](http://www.google.com/policies/privacy/) says nothing about how long they keep user data, and has some very interesting points about how they share it. [For one, Google regularly responds to requests from courts to share user data][1]. This is to be expected, but once again, is something that users need to be aware of. More interesting is 


HTTP Headers
------------

The HTTP/1.1 specification (which you have used to reach this page) lists variety of information that is to be sent from your browser to the server that it is connecting to.


[1]: http://www.google.com/transparencyreport/userdatarequests/
