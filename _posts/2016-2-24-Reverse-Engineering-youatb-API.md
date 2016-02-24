---
layout: post
title: Reverse Engineering the API of the "you@b" Android client
published: true
---


## The problem
The official Bocconi Android app feels very unprofessional, constantly restarts, the timetable is never cached, has no design at all, etc. To write a replacement-enchacement there are three ways:
- Reverse Engineering the API
- Write a client emulating desktop browser behaviour around the youatb website
- Get the information in a different way, for example, the timetables are available on the main website by code
I'll try the API way. 
I have decompiled the application with apktool, but nothing is reusable there anyway. Luckily, the communication channel between the app and API server seems to be unencrypted at all. Let's just sniff the traffic!

## Instruments
We need:
- Wireshark
- Android in a virtual machine, quickest way to setup seems to be Genymotion
- A package of Google Apps (I use [OpenGApps](http://opengapps.org/)), the app fails to work on AOSP without the proprietary libraries
- apk file = the app. One can download from an actual device or just login to Google. There are GApps and Google Play in this virtual machine, after all
- ADB, to install the apk

##Workflow in Pictures
![Screen1-2016-2-24.png]({{site.baseurl}}/images/Screen1-2016-2-24.png)
![Screen2-2016-2-24.png]({{site.baseurl}}/images/Screen2-2016-2-24.png)
![Screen3-2016-2-24.png]({{site.baseurl}}/images/Screen3-2016-2-24.png)
![Screen4-2016-2-24.png]({{site.baseurl}}/images/Screen4-2016-2-24.png)


## Collected data
#### App initialization, step 1
 Oh, look at the funny "b0cc0n1s3cr3t"!
###### Request
```http
GET /api/v3/urls?app=UNIBOCCONI_ANDROID&v=4.0 HTTP/1.1
auth_secret: b0cc0n1s3cr3t
User-Agent: Dalvik/2.1.0 (Linux; U; Android 6.0; Custom Phone - 6.0.0 - API 23 - 768x1280 Build/MRA58K)
Host: apilocate01.cineca.it
Connection: Keep-Alive
Accept-Encoding: gzip
```
###### Response
```http
HTTP/1.1 200 OK
Cache-Control: private
Content-Length: 174
Content-Type: application/json; charset=utf-8
Server: Microsoft-IIS/7.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Wed, 24 Feb 2016 08:48:02 GMT

{"ForceAppUpdate":false,"IsInMaintenance":false,"MaintenanceNotice":null,"ShowUpdateNotice":false,"URLs":["http:\/\/ks3-mobile.unibocconi.it\/universityapp_prod\/api\/v6\/"]}
```
#### Some kind of notification
Probably will never need it, some statistics for app developer
###### Request
```http
GET /Notificatore.aspx?CMD=initapp&os=2&devtoken=86453864de648284&regid=APA91bHGSyWpY_UMmo9Y3APx9_K_L-ufcu1s-2tVp0v4r3BT2hQBONBvhkzYyKhm5Y5LuAImHx2tsuFDb95RLbYAX2zVFHTx_ZeQ_7FhagGTeKW5YWizE80K3zqU_9Gj1WcKQJIyl-gx&appkey=UNIBOCCONI&lang=en HTTP/1.1
Host: notificatore01.cineca.it
Connection: Keep-Alive
User-Agent: Apache-HttpClient/UNAVAILABLE (java 1.4)
```
###### Response
```http
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 17
Content-Type: text/html
Expires: Wed, 24 Feb 2016 09:47:03 GMT
Vary: Accept-Encoding
Server: Microsoft-IIS/7.5
X-AspNet-Version: 2.0.50727
Set-Cookie: ASP.NET_SessionId=sysas4j3vezsvc45wtidsd45; path=/; HttpOnly
X-Powered-By: ASP.NET
Date: Wed, 24 Feb 2016 08:48:02 GMT

[{"result":"OK"}]
```
#### Second step of initialization
Calling the API address one got on the first step.
###### Request
```http
GET /universityapp_prod/api/v6/app/init?os=android HTTP/1.1
auth_secret: b0cc0n1s3cr3t
User-Agent: Dalvik/2.1.0 (Linux; U; Android 6.0; Custom Phone - 6.0.0 - API 23 - 768x1280 Build/MRA58K)
Host: ks3-mobile.unibocconi.it
Connection: Keep-Alive
Accept-Encoding: gzip
```
###### Response
```javascript
{  
   "allowed_sections":[  

   ],
   "auth_session":null,
   "auth_state":0,
   "configs":[  
      {  
         "id":2,
         "key":"WebmailAuthScript",
         "value":"javascript:(function main()  \u000a{\u000a      var check_dest = false;\u000a        var values = new Array(\u000a        'https:\/\/zimbra.unibocconi.it\/zimbra\/m\/zmain'\u000a      );\u000a\u000a    for (i in values) {\u000a        if (location.href.toLowerCase().indexOf(values[i].toLowerCase()) != -1) {\u000a            check_dest = true;\u000a        }\u000a    }\u000a    if (check_dest) {\u000a        Android.currentState('OK');\u000a    }\u000a    else if (location.href.toLowerCase().indexOf('loginfailed=true') != -1) {\u000a        Android.currentState('KO');\u000a    }\u000a    else { \u000a        var check_login = false;\u000a        var values = new Array(\u000a        \u000a        'https:\/\/idp.unibocconi.it:443\/idp\/Authn\/UserPassword',\u000a        'http:\/\/idp.unibocconi.it:443\/idp\/Authn\/UserPassword',\u000a        'https:\/\/idp.unibocconi.it\/idp\/Authn\/UserPassword',\u000a        'http:\/\/idp.unibocconi.it\/idp\/Authn\/UserPassword'\u000a            );\u000a    \u000a            for (i in values) {\u000a            if (location.href.toLowerCase().indexOf(values[i].toLowerCase()) != -1) {\u000a                check_login = true;\u000a            }\u000a            }\u000a            if (check_login) {\u000a            Android.currentState('PR');\u000a        var usernameFieldName = 'j_username';\u000a        var passwordFieldName = 'j_password';\u000a                \u000a        var usernameField   = null;\u000a        var passwordField   = null;\u000a        var submitButton    = null;\u000a \u000a        var inputElements = document.getElementsByTagName('input');\u000a        for (i in inputElements) {\u000a            if(inputElements[i].type == 'text' && inputElements[i].name == usernameFieldName) {\u000a                usernameField = inputElements[i];\u000a            }\u000a            else if(inputElements[i].type == 'password' && inputElements[i].name == passwordFieldName) {\u000a                passwordField = inputElements[i];\u000a            }\u000a\u000a            if (usernameField && passwordField) {\u000a                break;\u000a            }\u000a        }\u000a        \u000a        var buttonElements = document.getElementsByTagName(\"button\");\u000a        for (b in buttonElements) {\u000a            if(buttonElements[b].type == \"submit\") {\u000a                submitButton = buttonElements[b];\u000a            }\u000a\u000a            \/\/ detected submit button, stop searching...\u000a            if (submitButton) {\u000a                break;\u000a            }\u000a        }\u000a\u000a        if (!usernameField || !passwordField || !submitButton) {\u000a            Android.currentState('KO');\u000a        }\u000a \u000a        usernameField.value = '{username}';\u000a        passwordField.value = '{password}';\u000a        submitButton.click();\u000a        }\u000a        }\u000a    }) ()\u000a"
      },
      {  
         "id":3,
         "key":"FeedEntryTemplate",
         "value":"<!DOCTYPE HTML>\u000a<html>\u000a  <head>\u000a  <title>{title}<\/title>\u000a  <meta charset=\"UTF-8\">\u000a  <meta name=\"viewport\" content=\"initial-scale=1.0; maximum-scale=1.0; user-scalable=no\" \/>\u000a  <link href=\"http:\/\/fonts.googleapis.com\/css?family=Cabin:400,400italic,500,500italic,600,600italic,bold,bolditalic\" rel=\"stylesheet\" type=\"text\/css\" \/>\u000a  <link href=\"http:\/\/fonts.googleapis.com\/css?family=PT+Sans:regular,italic,bold,bolditalic\" rel=\"stylesheet\" type=\"text\/css\" \/>\u000a  <style>\u000a    h1.feed-header\u000a    {\u000a      font-family: 'Cabin', serif;\u000a      font-size: 18px;\u000a      font-style: normal;\u000a      font-weight: 700;\u000a      text-shadow: none;\u000a      text-decoration: none;\u000a      text-transform: none;\u000a      letter-spacing: 0em;\u000a      word-spacing: 0em;\u000a      line-height: 1.2;\u000a    }\u000a    .feed-content\u000a    {\u000a      font-family: 'PT Sans', serif;\u000a      font-size: 16px;\u000a      font-style: normal;\u000a      font-weight: 400;\u000a      text-shadow: none;\u000a      text-decoration: none;\u000a      text-transform: none;\u000a      letter-spacing: 0em;\u000a      word-spacing: 0em;\u000a      line-height: 1.2;\u000a    }\u000a  <\/style>\u000a  <script type=\"text\/javascript\" src=\"http:\/\/code.jquery.com\/jquery-latest.min.js\"><\/script>\u000a  <script type=\"text\/javascript\">\u000a    $(document).ready(function(){\u000a      $(\"img\").load(function(){\u000a        var maxWidth = 200;     \/\/ Max width for the image\u000a        var maxHeight = 350;    \/\/ Max height for the image\u000a        var ratio = 0;          \/\/ Used for aspect ratio\u000a        var width = $(this).width();    \/\/ Current image width\u000a        var height = $(this).height();  \/\/ Current image height\u000a        \u000a        \/\/ Check if the current width is larger than the max\u000a        if(width > maxWidth){\u000a          ratio = maxWidth \/ width;                 \/\/ get ratio for scaling image\u000a          $(this).css(\"width\", maxWidth);           \/\/ Set new width\u000a          $(this).css(\"height\", height * ratio);    \/\/ Scale height based on ratio\u000a          height = height * ratio;                  \/\/ Reset height to match scaled image\u000a          width = width * ratio;                    \/\/ Reset width to match scaled image\u000a        }\u000a        \u000a        \/\/ Check if current height is larger than max\u000a        if(height > maxHeight){\u000a          ratio = maxHeight \/ height;               \/\/ get ratio for scaling image\u000a          $(this).css(\"height\", maxHeight);         \/\/ Set new height\u000a          $(this).css(\"width\", width * ratio);      \/\/ Scale width based on ratio\u000a          width = width * ratio;                    \/\/ Reset width to match scaled image\u000a        }\u000a        $(this).css(\"float\", \"left\")\u000a        $(this).css(\"border-style\", \"none\")\u000a      }) <!-- Closes 'each' -->\u000a  }) <!-- Closes 'ready' -->\u000a  <\/script>\u000a  <\/head>\u000a  <body>\u000a    <h1 class=\"feed-header\">{title}<\/h1>\u000a    <!-- Content begins -->\u000a    <div class=\"feed-content\">\u000a    {content}\u000a    <!-- Content ends -->\u000a    <\/div>\u000a  <\/body>\u000a<\/html>\u000a"
      },
      {  
         "id":5,
         "key":"DefaultLanguage",
         "value":"it"
      },
      {  
         "id":7,
         "key":"WebmailRequestURL",
         "value":"https:\/\/webmail.studbocconi.it\/"
      },
      {  
         "id":9,
         "key":"WebAuthScript",
         "value":"NOT_USED"
      },
      {  
         "id":14,
         "key":"VideoDetailTemplate",
         "value":"<!DOCTYPE HTML>\u000a<html>\u000a  <head>\u000a    <title>{title}<\/title>\u000a    <style type=\"text\/css\">\u000a    body {\u000a      background-color: transparent;\u000a      color: white;\u000a    }\u000a    <\/style>\u000a  <\/head>\u000a  <body style=\"margin:0\">\u000a    <object type=\"application\/x-shockwave-flash\" style=\"height: {height}px; width: {width}px\">\u000a      <param name=\"movie\" value=\"{sourceURI}\">\u000a      <param name=\"allowFullScreen\" value=\"true\">\u000a      <param name=\"allowScriptAccess\" value=\"always\">\u000a      <embed src=\"{sourceURI}\"\u000a           type=\"application\/x-shockwave-flash\"\u000a           allowfullscreen=\"true\"\u000a           allowScriptAccess=\"always\"\u000a           width=\"{width}\"\u000a           height=\"{height}\">\u000a    <\/object>\u000a  <\/body>\u000a<\/html>\u000a"
      },
      {  
         "id":15,
         "key":"MobilizerURL",
         "value":"http:\/\/www.instapaper.com\/m?u={url}"
      },
      {  
         "id":17,
         "key":"WebAuthTimeout",
         "value":"10"
      },
      {  
         "id":18,
         "key":"LauncherPhotoGalleryURI_Android",
         "value":"http:\/\/www.unibocconi.it\/wtm\/banner_f_eng.xml"
      },
      {  
         "id":20,
         "key":"LatestAppRelease",
         "value":"3"
      },
      {  
         "id":21,
         "key":"LinkAuthScript",
         "value":"javascript:(function main()  \u000d\u000a{\u000d\u000a    var check_login = false;\u000d\u000a    var values = new Array(        \u000d\u000a        'https:\/\/idp.unibocconi.it:443\/idp\/Authn\/UserPassword',\u000d\u000a        'http:\/\/idp.unibocconi.it:443\/idp\/Authn\/UserPassword',\u000d\u000a        'https:\/\/idp.unibocconi.it\/idp\/Authn\/UserPassword',\u000d\u000a        'http:\/\/idp.unibocconi.it\/idp\/Authn\/UserPassword'\u000d\u000a    );\u000d\u000a        \u000d\u000a    for (i in values) {\u000d\u000a        if (location.href.toLowerCase().indexOf(values[i].toLowerCase()) != -1) {\u000d\u000a            check_login = true;    \u000d\u000a        }\u000d\u000a    }\u000d\u000a    \u000d\u000a    var values_fail = new Array(        \u000d\u000a        'loginfailed',\u000d\u000a        'credentials not recognized'\u000d\u000a    );\u000d\u000a    \u000d\u000a    var loginFailed = false;\u000d\u000a    for (i in values_fail) {\u000d\u000a        if (location.href.toLowerCase().indexOf(values_fail[i].toLowerCase()) != -1 \u000d\u000a            || document.getElementsByTagName('html')[0].innerHTML.toLowerCase().indexOf(values_fail[i].toLowerCase()) > -1) {\u000d\u000a            loginFailed = true;    \u000d\u000a        }\u000d\u000a    }\u000d\u000a    \u000d\u000a    Android.addLog('checklogin ' + check_login + ' --- ' + location.href);\u000d\u000a    if (loginFailed) {\u000d\u000a        Android.currentState('KO');\u000d\u000a    }\u000d\u000a    else if (check_login)\u000d\u000a    {\u000d\u000a        Android.currentState('PR');\u000d\u000a        var usernameFieldName = 'j_username';\u000d\u000a        var passwordFieldName = 'j_password';\u000d\u000a                \u000d\u000a        var usernameField   = null;\u000d\u000a        var passwordField   = null;\u000d\u000a        var submitButton    = null;\u000d\u000a \u000d\u000a        var inputElements = document.getElementsByTagName('input');\u000d\u000a        for (i in inputElements) {\u000d\u000a            if(inputElements[i].type == 'text' && inputElements[i].name == usernameFieldName) {\u000d\u000a                usernameField = inputElements[i];\u000d\u000a            }\u000d\u000a            else if(inputElements[i].type == 'password' && inputElements[i].name == passwordFieldName) {\u000d\u000a                passwordField = inputElements[i];\u000d\u000a            }\u000d\u000a\u000d\u000a            if (usernameField && passwordField) {\u000d\u000a                break;\u000d\u000a            }\u000d\u000a        }\u000d\u000a        \u000d\u000a        var buttonElements = document.getElementsByTagName(\"button\");\u000d\u000a        for (b in buttonElements) {\u000d\u000a            if(buttonElements[b].type == \"submit\") {\u000d\u000a                submitButton = buttonElements[b];\u000d\u000a            }\u000d\u000a\u000d\u000a            \/\/ detected submit button, stop searching...\u000d\u000a            if (submitButton) {\u000d\u000a                break;\u000d\u000a            }\u000d\u000a        }\u000d\u000a                \u000d\u000a        \u000d\u000a        if (!usernameField || !passwordField || !submitButton) {\u000d\u000a            Android.currentState('KO');\u000d\u000a        }\u000d\u000a \u000d\u000a        usernameField.value = '{username}';\u000d\u000a        passwordField.value = '{password}';\u000d\u000a        submitButton.click();\u000d\u000a    }\u000d\u000a    else\u000d\u000a    {\u000d\u000a        Android.currentState('OK');\u000d\u000a    }\u000d\u000a}) ()\u000d\u000a"
      },
      {  
         "id":23,
         "key":"RandomURL_0",
         "value":"{auth}https:\/\/www.pb.unibocconi.it\/auth\/studente\/Appelli\/AppelliF.do?stu_id={id_student}&cod_lingua={rl}&EnableLayout=0"
      },
      {  
         "id":25,
         "key":"CourseDetailTemplate",
         "value":"<!DOCTYPE HTML>\u000a<html>\u000a  <head>\u000a  <title>{title}<\/title>\u000a  <meta charset=\"UTF-8\">\u000a  <meta name=\"viewport\" content=\"initial-scale=1.0; maximum-scale=1.0; user-scalable=no\" \/>\u000a  <link href=\"http:\/\/fonts.googleapis.com\/css?family=Cabin:400,400italic,500,500italic,600,600italic,bold,bolditalic\" rel=\"stylesheet\" type=\"text\/css\" \/>\u000a  <link href=\"http:\/\/fonts.googleapis.com\/css?family=PT+Sans:regular,italic,bold,bolditalic\" rel=\"stylesheet\" type=\"text\/css\" \/>\u000a  <style>\u000a    strong\u000a    {\u000a      font-family: 'Cabin', serif;\u000a      font-size: 18px;\u000a      font-style: normal;\u000a      font-weight: 700;\u000a      text-shadow: none;\u000a      text-decoration: none;\u000a      text-transform: none;\u000a      letter-spacing: 0em;\u000a      word-spacing: 0em;\u000a      line-height: 1.2;\u000a    }\u000a    body\u000a    {\u000a      font-family: 'PT Sans', serif;\u000a      font-size: 16px;\u000a      font-style: normal;\u000a      font-weight: 400;\u000a      text-shadow: none;\u000a      text-decoration: none;\u000a      text-transform: none;\u000a      letter-spacing: 0em;\u000a      word-spacing: 0em;\u000a      line-height: 1.2;\u000a    }\u000a  <\/style>\u000a  <\/head>\u000a  <body>\u000a    {description}\u000a    {educational_objectives}\u000a    {content}\u000a    {learning_modes}\u000a    {methods_of_teaching}\u000a    {other_info}\u000a    {prerequisites}\u000a    {references}\u000a  <\/body>\u000a<\/html>\u000a"
      },
      {  
         "id":27,
         "key":"WebAuthRequestURL",
         "value":"https:\/\/www.pb.unibocconi.it\/auth\/studente\/Piani\/PianiHome.do"
      },
      {  
         "id":28,
         "key":"RandomURLAuthScript",
         "value":"javascript:(function main()\u000a{ Android.currentState('PR');\u000a      var check_dest = false;      \u000a      var values = new Array(\u000a        \"Appelli\"\u000a    );\u000a    \u000a    for (i in values) {\u000a        if (location.href.toLowerCase().indexOf(values[i].toLowerCase()) != -1) {\u000a            check_dest = true;\u000a        }\u000a    }\u000a          \u000a    if (check_dest) {\u000a        Android.currentState('OK'); \/\/Android.showToast('Dest raggiunta');\u000a    }\u000a    else if (location.href.toLowerCase().indexOf('loginfailed=true') != -1) {\u000a        Android.currentState('KO');\u000a    }\u000a    else {\u000a          var check_login = false;\u000a            var loginURLs = new Array();\u000a\u000a            loginURLs[0] = 'https:\/\/idp.unibocconi.it:443\/idp\/Authn\/UserPassword';\u000a            loginURLs[1] = 'http:\/\/idp.unibocconi.it:443\/idp\/Authn\/UserPassword';\u000a            loginURLs[2] = 'https:\/\/idp.unibocconi.it\/idp\/Authn\/UserPassword';\u000a            loginURLs[3] = 'http:\/\/idp.unibocconi.it\/idp\/Authn\/UserPassword';\u000a                        \u000a            for (i in loginURLs) {\u000a                if (location.href.indexOf(loginURLs[i]) != -1) {\u000a                    check_login = true;\u000a                }\u000a            }\u000a            if (check_login) { \u000a            var usernameFieldName = 'j_username';\u000a            var passwordFieldName = 'j_password';\u000a            \u000a            var usernameField   = null;\u000a            var passwordField   = null;\u000a            var submitButton    = null;\u000a    \u000a            var inputElements = document.getElementsByTagName('input');\u000a            for (i in inputElements) {\u000a                if(inputElements[i].type == 'text' && inputElements[i].name == usernameFieldName) {\u000a                    usernameField = inputElements[i];\u000a                }\u000a                else if(inputElements[i].type == 'password' && inputElements[i].name == passwordFieldName) {\u000a                    passwordField = inputElements[i];\u000a                }\u000a    \u000a                if (usernameField && passwordField) {\u000a                    break;\u000a                }\u000a            }\u000a            \u000a            var buttonElements = document.getElementsByTagName(\"button\");\u000a        \u0009\u0009for (b in buttonElements) {\u000a\u0009            if(buttonElements[b].type == \"submit\") {\u000a\u0009                submitButton = buttonElements[b];\u000a\u0009            }\u000a\u0009\u000a\u0009            \/\/ detected submit button, stop searching...\u000a\u0009            if (submitButton) {\u000a\u0009                break;\u000a\u0009            }\u000a        \u0009\u0009}\u000a        \u000a            if (usernameField && passwordField && submitButton) { \/\/Android.showToast('Valorizzo dati');\u000a            usernameField.value = '{username}';\u000a            passwordField.value = '{password}';\u000a            submitButton.click();\u000a            }    \u000a          }\u000a    }\u000a    }) ()\u000a"
      },
      {  
         "id":32,
         "key":"LauncherPhotoGalleryAlternativeURI_Android",
         "value":"http:\/\/www.unibocconi.it\/wtm\/banner_f_ita.xml"
      },
      {  
         "id":34,
         "key":"SupportToRecipients",
         "value":"mobile@unibocconi.it"
      },
      {  
         "id":38,
         "key":"AboutCreditsURL",
         "value":"http:\/\/www.cineca.it\/en"
      },
      {  
         "id":39,
         "key":"AboutBannerURL",
         "value":"http:\/\/www.unibocconi.it\/"
      },
      {  
         "id":41,
         "key":"EventTemplate",
         "value":"<!DOCTYPE HTML>\u000a<html>\u000a  <head>\u000a  <title>{title}<\/title>\u000a  <meta charset=\"UTF-8\">\u000a  <meta name=\"viewport\" content=\"initial-scale=1.0; maximum-scale=1.0; user-scalable=no\" \/>\u000a  <link href=\"http:\/\/fonts.googleapis.com\/css?family=Cabin:400,400italic,500,500italic,600,600italic,bold,bolditalic\" rel=\"stylesheet\" type=\"text\/css\" \/>\u000a  <link href=\"http:\/\/fonts.googleapis.com\/css?family=PT+Sans:regular,italic,bold,bolditalic\" rel=\"stylesheet\" type=\"text\/css\" \/>\u000a  <style>\u000a    h1.event-header\u000a    {\u000a      font-family: 'Cabin', serif;\u000a      font-size: 18px;\u000a      font-style: normal;\u000a      font-weight: 700;\u000a      text-shadow: none;\u000a      text-decoration: none;\u000a      text-transform: none;\u000a      letter-spacing: 0em;\u000a      word-spacing: 0em;\u000a      line-height: 1.2;\u000a    }\u000a    .event-content\u000a    {\u000a      font-family: 'PT Sans', serif;\u000a      font-size: 16px;\u000a      font-style: normal;\u000a      font-weight: 400;\u000a      text-shadow: none;\u000a      text-decoration: none;\u000a      text-transform: none;\u000a      letter-spacing: 0em;\u000a      word-spacing: 0em;\u000a      line-height: 1.2;\u000a\u000a    }\u000a\u000a  <\/style>\u000a  <script type=\"text\/javascript\" src=\"http:\/\/code.jquery.com\/jquery-latest.min.js\"><\/script>\u000a  <script type=\"text\/javascript\">\u000a    $(document).ready(function(){\u000a      $(\"img\").load(function(){\u000a        var maxWidth = 200;     \/\/ Max width for the image\u000a        var maxHeight = 350;    \/\/ Max height for the image\u000a        var ratio = 0;          \/\/ Used for aspect ratio\u000a        var width = $(this).width();    \/\/ Current image width\u000a        var height = $(this).height();  \/\/ Current image height\u000a        \u000a        \/\/ Check if the current width is larger than the max\u000a        if(width > maxWidth){\u000a          ratio = maxWidth \/ width;                 \/\/ get ratio for scaling image\u000a          $(this).css(\"width\", maxWidth);           \/\/ Set new width\u000a          $(this).css(\"height\", height * ratio);    \/\/ Scale height based on ratio\u000a          height = height * ratio;                  \/\/ Reset height to match scaled image\u000a          width = width * ratio;                    \/\/ Reset width to match scaled image\u000a        }\u000a        \u000a        \/\/ Check if current height is larger than max\u000a        if(height > maxHeight){\u000a          ratio = maxHeight \/ height;               \/\/ get ratio for scaling image\u000a          $(this).css(\"height\", maxHeight);         \/\/ Set new height\u000a          $(this).css(\"width\", width * ratio);      \/\/ Scale width based on ratio\u000a          width = width * ratio;                    \/\/ Reset width to match scaled image\u000a        }\u000a        $(this).css(\"float\", \"left\");\u000a        $(this).css(\"border-style\", \"none\");\u000a\u000a        $(this).css(\"margin-right\", \"10px\");\u000a\u000a      }) <!-- Closes 'each' -->\u000a  }) <!-- Closes 'ready' -->\u000a  <\/script>\u000a  <\/head>\u000a  <body>\u000a    <img src=\"{thumbnail}\" \/>\u000a\u000a    <h1 class=\"event-header\">{title}<\/h1>\u000a\u000a    <!-- Content begins -->\u000a\u000a    <div class=\"event-content\" style=\"display: block;\">\u000a\u000a      <span style=\"color: gray;\">{date}<\/span><br \/>\u000a\u000a      <strong>{location}<\/strong>\u000a\u000a      {speaker}<br \/>\u000a\u000a      {payment}<br \/>\u000a\u000a      {content}<br \/>\u000a    <\/div>\u000a    <!-- Content ends -->\u000a  <\/body>\u000a<\/html>\u000a"
      },
      {  
         "id":42,
         "key":"AboutInfo",
         "value":"<!DOCTYPE HTML>\u000a<html>\u000a  <head>\u000a  <title>{title}<\/title>\u000a  <meta charset=\"UTF-8\">\u000a  <meta name=\"viewport\" content=\"initial-scale=1.0; maximum-scale=1.0; user-scalable=no\" \/>\u000a  <link href=\"http:\/\/fonts.googleapis.com\/css?family=Cabin:400,400italic,500,500italic,600,600italic,bold,bolditalic\" rel=\"stylesheet\" type=\"text\/css\" \/>\u000a  <link href=\"http:\/\/fonts.googleapis.com\/css?family=PT+Sans:regular,italic,bold,bolditalic\" rel=\"stylesheet\" type=\"text\/css\" \/>\u000a  <style>\u000a    strong\u000a    {\u000a      font-family: 'Cabin', serif;\u000a      font-size: 18px;\u000a      font-style: normal;\u000a      font-weight: 700;\u000a      text-shadow: none;\u000a      text-decoration: none;\u000a      text-transform: none;\u000a      letter-spacing: 0em;\u000a      word-spacing: 0em;\u000a      line-height: 1.2;\u000a    }\u000a    body\u000a    {\u000a      font-family: 'PT Sans', serif;\u000a      font-size: 16px;\u000a      font-style: normal;\u000a      font-weight: 400;\u000a      text-shadow: none;\u000a      text-decoration: none;\u000a      text-transform: none;\u000a      letter-spacing: 0em;\u000a      word-spacing: 0em;\u000a      line-height: 1.2;\u000a    }\u000a  <\/style>\u000a  <\/head>\u000a  <body>\u000a    <p><strong>Universit Commerciale Luigi Bocconi<\/strong><br \/>Via Sarfatti, 25 - Milan<\/p>\u000a  <\/body>\u000a<\/html>\u000a"
      }
   ],
   "featured_messages":[  

   ],
   "randomurl_configs":[  
      {  
         "auth":true,
         "module_id":208,
         "script":null,
         "type":0,
         "url":"https:\/\/webmail.studbocconi.it\/"
      },
      {  
         "auth":true,
         "module_id":301,
         "script":null,
         "type":0,
         "url":"https:\/\/lib.unibocconi.it\/iii\/cas\/login?service=https%3A%2F%2Flib.unibocconi.it%3A443%2Fpatroninfo~S8*eng%2FIIITICKET&lang=eng&scope=8"
      }
   ],
   "sections":[  
      100,
      131,
      121,
      132,
      133,
      200,
      201,
      202,
      205,
      208,
      0,
      301,
      9
   ]
}
```
#### Slides downloading
There are some inspirational images of students and buildings downloaded, don't need that.
#### Authorization
Seems to be the same as initialization, but now a parameter "Authorization" is added.
###### Request
```http
GET /api/v3/urls?app=UNIBOCCONI_ANDROID&v=4.0 HTTP/1.1
If-Modified-Since: Wed, 24 Feb 2016 08:48:02 GMT+00:00
auth_secret: b0cc0n1s3cr3t
Authorization: BASIC [REDACTED]
User-Agent: Dalvik/2.1.0 (Linux; U; Android 6.0; Custom Phone - 6.0.0 - API 23 - 768x1280 Build/MRA58K)
Host: apilocate01.cineca.it
Connection: Keep-Alive
Accept-Encoding: gzip
```

###### Response
```http
HTTP/1.1 200 OK
Cache-Control: private
Content-Length: 174
Content-Type: application/json; charset=utf-8
Server: Microsoft-IIS/7.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Wed, 24 Feb 2016 08:48:39 GMT

{"ForceAppUpdate":false,"IsInMaintenance":false,"MaintenanceNotice":null,"ShowUpdateNotice":false,"URLs":["http:\/\/ks3-mobile.unibocconi.it\/universityapp_prod\/api\/v6\/"]}
```
###### Comment
Nice, they are using Basic access authentication. So, to login one needs to pass `username:password` encoded using the RFC2045-MIME variant of Base64. No need to be scared of `MTExMjIyOnN0cm9uZ3Bhc3M=`, it's just `111222:strongpass`.

#### Authorization, step 2
The second step of authorized initialization is very important, if you are authorized. It lets you get the `id` which will be needed for timetable downloading.
###### Request
```http
GET /universityapp_prod/api/v6/app/init?os=android HTTP/1.1
If-Modified-Since: Wed, 24 Feb 2016 08:47:55 GMT+00:00
auth_secret: b0cc0n1s3cr3t
Authorization: BASIC [REDACTED]
User-Agent: Dalvik/2.1.0 (Linux; U; Android 6.0; Custom Phone - 6.0.0 - API 23 - 768x1280 Build/MRA58K)
Host: ks3-mobile.unibocconi.it
Connection: Keep-Alive
Accept-Encoding: gzip
```

###### Response
Gives the same massive amount of JSON as the unauthorized version, what's important is the difference between the responses. The diff is small.

```
@@ -1,9 +1,29 @@
+
 	{  
 	   "allowed_sections":[  
 
 	   ],
-	   "auth_session":null,
-	   "auth_state":0,
+	   "auth_session":{  
+	      "careers":[  
+	         {  
+	            "date_end":null,
+	            "date_start":"\/Date(1423868400000+0100)\/",
+	            "description":"Attivo per Immatricolazione",
+	            "id":[THE VERY IMPORTANT ID IS HERE],
+	            "is_blocked":false,
+	            "notes":"[PROGRAM DESCRIPTION]",
+	            "registration_number":"111222",
+	            "title":"[PROGRAM NAME]"
+	         }
+	      ],
+	      "firstname":"John",
+	      "id":null,
+	      "lastname":"Doe",
+	      "photo_url":"http:\/\/ks3-mobile.unibocconi.it\/Esse3Photo\/userphoto?userId=[USERID]",
+	      "type":1,
+	      "user_id":111222
+	   },
+	   "auth_state":1,
 	   "configs":[  
 	      {  
 	         "id":2,
@@ -118,14 +138,14 @@
 	      {  
 	         "auth":true,
 	         "module_id":208,
-	         "script":null,
+	         "script":"[SOME SCRIPT]",
 	         "type":0,
 	         "url":"https:\/\/webmail.studbocconi.it\/"
 	      },
 	      {  
 	         "auth":true,
 	         "module_id":301,
-	         "script":null,
+	         "script":"[SOME SCRIPT]",
 	         "type":0,
 	         "url":"https:\/\/lib.unibocconi.it\/iii\/cas\/login?service=https%3A%2F%2Flib.unibocconi.it%3A443%2Fpatroninfo~S8*eng%2FIIITICKET&lang=eng&scope=8"
 	      }
```

I have deleted personal information and replaced it with placeholders for John Doe with username 111222. That's the place where one can get the `id` that will be needed to access the timetable.

#### Agenda for today
###### Request
```http
GET /universityapp_prod/api/v6/students/[THE ID FROM INITIALIZATION]/agenda?rl=en&start=20160224 HTTP/1.1
auth_secret: b0cc0n1s3cr3t
Authorization: BASIC [REDACTED]
User-Agent: Dalvik/2.1.0 (Linux; U; Android 6.0; Custom Phone - 6.0.0 - API 23 - 768x1280 Build/MRA58K)
Host: ks3-mobile.unibocconi.it
Connection: Keep-Alive
Accept-Encoding: gzip
```
###### Response
```http
HTTP/1.1 200 OK
Cache-Control: private
Content-Length: 1027
Content-Type: application/json; charset=utf-8
Server: Microsoft-IIS/7.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Wed, 24 Feb 2016 08:48:41 GMT

[{"date":"\/Date(1456268400000+0100)\/","events":[{"date_end":"\/Date(1456305300000+0100)\/","date_start":"\/Date(1456299900000+0100)\/","description":"[COURSE CODE] [PROGRAMS TAUGHT] - [PROFESSOR]","id":[SOME ID],"supertitle":"[ROOM]","title":"[COURSE NAME]","type":1},{"date_end":"\/Date(1456311600000+0100)\/","date_start":"\/Date(1456306200000+0100)\/","description":"[COURSE CODE] [PROGRAMS TAUGHT] - [PROFESSOR]","id":[SOME ID],"supertitle":"[ROOM]","title":"[COURSE NAME]","type":1},{"date_end":"\/Date(1456318800000+0100)\/","date_start":"\/Date(1456313400000+0100)\/","description":"[COURSE CODE] [PROGRAMS TAUGHT] - [PROFESSOR]","id":[SOME ID],"supertitle":"[ROOM]","title":"[COURSE NAME]","type":1},{"date_end":"\/Date(1456335900000+0100)\/","date_start":"\/Date(1456326900000+0100)\/","description":"[COURSE CODE] [PROGRAMS TAUGHT] - [PROFESSOR]","id":[SOME ID],"supertitle":"[ROOM]","title":"[COURSE NAME]","type":1}]}]
```
#### Agenda for a period of time
###### Request
```http
GET /universityapp_prod/api/v6/students/[THE ID FROM INITIALIZATION]/agenda?rl=en&start=20160131&end=20160206 HTTP/1.1
auth_secret: b0cc0n1s3cr3t
Authorization: BASIC [REDACTED]
User-Agent: Dalvik/2.1.0 (Linux; U; Android 6.0; Custom Phone - 6.0.0 - API 23 - 768x1280 Build/MRA58K)
Host: ks3-mobile.unibocconi.it
Connection: Keep-Alive
Accept-Encoding: gzip
```
###### Response
Will be in the same form as for one day.


## Looking forward
Let's write an app!
