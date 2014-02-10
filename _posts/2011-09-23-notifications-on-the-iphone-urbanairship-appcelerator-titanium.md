---
layout: post
title: Notifications on the iPhone, or the lack there-of, and what I did about it...
date: 2011-09-23 14:36
disqus_identifier: 10562895846
---

Dear Apple: Are you serious? After 4 major versions of your mobile operating system I **STILL** can’t set different alert profiles for different email accounts?

In my case I want certain email accounts I have setup to provide different feedback. See, I get a lot of email, most of it is just informative messages coming from automated systems that don’t necessarily require a timely response while others are alerts that I want to be informed of immediately. This seems like such a trivial thing, yet has a huge impact on my workflow productivity.

When Apple first released the 3G I left my Blackberry Curve behind and joined the masses in coming to iPhone. After 8 months I asserted that the iPhone was not a productivity device and returned to Blackberry and the comfort of a physical keyboard and a multitude of configuration options where I happily remained until this week when my BES crashed and I had enough of fighting with it.

I considered going to an Android device, but recently FatBox has started building iPhone apps and while my iPod Touch and the simulator have done me well during this first development cycle it really behoves me to have an iPhone now that we’re getting REALLY close to releasing our first app into the App store (stay tuned for a release announcement that should happen in the next couple weeks!).

I digress. The point is here I am with my shiny new iPhone 4 facing the same problems I was facing back with my 3G. However, this time it’s a little different. This time I can do something about it, so I did…

First, a quick summary of the situation: I have a system that sends out automated emails and I want to be notified whenever I get an email from this system in a way that is different from all my other email notifications.

Since iOS doesn’t allow me to set it up the way I want using the Mail app I decided that I would write my own app and change my system to not only send the email, but also send me a push notification.

First up is the app that will run on the iPhone. Now, I will admit that if you’ve never built an app for the iPhone it’s not creating the app itself that’s going to be your biggest hurdle, it’s all the requisite knowledge of how Apple’s PKI ([Public Key Infrastructure](http://en.wikipedia.org/wiki/Public_key_infrastructure)) works in regards to provisioning profiles, push notification certificates, and all the likes. But, if you are versed in the requisite knowledge then this exercise should only take a couple hours — It took me 4 in total but I also went a little above and beyond in the device side of things, just because that’s how I roll…

For the app I used the [Titanium framework from Appcelerator](http://appcelerator.com). If you’ve never heard of this before it allows you to create apps via ECMAScript (aka Javascript) that are compiled into native applications. Now, I’ll admit that it’s a double edged sword as this opens mobile development up to a much wider audience who for the most part probably shouldn’t be developing mobile apps in the first place because while the barrier to entry is much lower now Titanium doesn’t remove you from having to understand how to build applications properly in the first place and I’ve read one too many posts blaming Titanium when it’s very clearly the authors fault for not understanding memory management, scope, or a myriad of other fundamentals (sorry, I try not to be mean, but if you’ve looked through all the crap in the app store you should at least agree with me is the last thing the app store needs is another piece of crap $0.99 app that crashes, is buggy, drains your battery, etc.).

Alongside Titanium we’re also going to use [Urban Airship](http://urbanairship.com/) to send the actual push notifications as they have already been integrated into Titanium so it’s a breeze to send them (again, if you understand how [APNS](http://developer.apple.com/library/ios/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/ApplePushService/ApplePushService.html) works it should only take about 10 minutes to get your account setup and ready to test).

In the end 110 lines of code are all it took to get the app finished. This includes a SQLite database to hold a local copy of the messages received and a table view to display them.

{% highlight javascript %}
Ti.API.debug("FatBox Ops - Starting up...");

var APP_KEY = 'Your UA key';
var APP_SECRET = 'Your UA master secret';

function mon_db(callback) {
    var db = Ti.Database.open("monitoring");
    ret = callback(db);
    db.close();
    return ret;
}

Ti.API.debug("Preparing database");
mon_db(function(db) {
    db.execute("CREATE TABLE IF NOT EXISTS monitoring (id INTEGER PRIMARY KEY, msg TEXT)");
});

Ti.API.debug("Registering for push notifications");
Titanium.Network.registerForPushNotifications({
    types:[
        Titanium.Network.NOTIFICATION_TYPE_BADGE,
        Titanium.Network.NOTIFICATION_TYPE_ALERT,
        Titanium.Network.NOTIFICATION_TYPE_SOUND
    ],
    success: successCallback,
    error: errorCallback,
    callback: messageCallback
});

function successCallback(e) {
    Ti.API.debug("Success in registerForPushNotifications!");

    var request = Titanium.Network.createHTTPClient({
        onload:function(e) {
            if (request.status != 200 && request.status != 201) {
                request.onerror(e);
                return;
            }
        },
        onerror:function(e) {
            Ti.API.info("Register with Urban Airship Push Service failed. Error: "
            + e.error);
        }
    });

    // Register device token with UA
    request.open('PUT', 'https://go.urbanairship.com/api/device_tokens/' + e.deviceToken, true);
    request.setRequestHeader('Authorization', 'Basic ' + Titanium.Utils.base64encode(APP_KEY + ':' + APP_SECRET));

    request.send();
}

function errorCallback(e) {
    Ti.API.info("Error during registration: " + e.error);
}

function messageCallback(e) {
    var msg = e.data['aps']['alert'];
    Ti.API.debug(["Handling new message", msg].join(': '));
    mon_db(function(db) {
        db.execute("INSERT INTO monitoring (msg) VALUES (?)", msg);
    });
    addRow(msg);
}

var rowAdded = false;
function addRow(msg) {
    var newRow = Ti.UI.createTableViewRow({
        height: 'auto',
        });
    var rowView = Ti.UI.createView({
        height: 'auto',
        });
    var msgLabel = Ti.UI.createLabel({
        left: 5,
        top: 5,
        right: 5,
        bottom: 5,
        height: 'auto',
        text: msg,
    });
    rowView.add(msgLabel);
    newRow.className = 'msg';
    newRow.add(rowView);
    if (rowAdded === false) {
        msgTable.appendRow(newRow);
        rowAdded = true;
    } else {
        msgTable.insertRowBefore(0, newRow);
    }
}

// create our base window
Titanium.UI.setBackgroundColor('#FFFFFF');
mainWindow = Ti.UI.createWindow();

msgTable = Ti.UI.createTableView();
mainWindow.add(msgTable);

Ti.API.debug("Loading existing rows");
mon_db(function(db) {
    rs = db.execute("SELECT * FROM monitoring ORDER BY id ASC LIMIT 50");
    while (rs.isValidRow()) {
        addRow(rs.fieldByName('msg'));
        rs.next();
    }
});

mainWindow.open();
{% endhighlight %}

Next comes the actual sending of the push notifications, which thanks to Urban Airship means we just need to send a POST request to a web service. NOTE: I have simplified this a bit to remove my specific option parsing.

{% highlight python %}
#!/usr/bin/env python
#
# this is a simple little script to send push notifications via an urban airship broadcast message
#

from optparse import OptionParser

import simplejson as json
import urllib2
import base64
import sys

APP_ID = 'Your UA app id'
APP_SECRET = 'Your UA app secret'

parser = OptionParser()
parser.add_option("--message", action="store")
parser.add_option("--sound", action="store")

(options, args) = parser.parse_args()

data = {
        "aps": {
            "alert": options.message,
            "sound": options.sound,
            }
        }

auth_str = base64.encodestring('%s:%s' % (APP_ID, APP_SECRET)).replace('\n', '')
request = urllib2.Request("https://go.urbanairship.com/api/push/broadcast/")
request.add_header("Authorization", "Basic %s" % auth_str)
request.add_header("Content-Type", "application/json")
result = urllib2.urlopen(request, json.dumps(data))
{% endhighlight %}

And that’s it!

4 hours. Less than 150 lines of code and now I get my notifications how I want them.
