# phantom-proxy
* Allows you to drive [phantomjs](www.phantomjs.org) from [node](www.nodejs.org)
* Does not rely on alert side effects or communicating with phantomjs
* Uses Phantom's embedded web server for communication
* Provides a full api - including a parametrized evaluate function - no more hard coded strings
* Provides additional useful methods such as waitForSelector
* Can easily integrate with feature testing frameworks such as [cucumber](https://github.com/cucumber/cucumber-js), jasmine, mocha

## Overview
PhantomJs is an incredibly useful tool for functional and unit testing.  PhantomJs runs in its own process, making it difficult to drive from node.  Phantom-proxy solves this problem by spawning a phantomjs process, then using httprequests to communicate with phantom's embedded mongoose webserver.  The result is a 

## Installation
`npm install phantom-proxy` 

## Usage
### API
#### phantomProxy object
##### create(options, callbackFn)
use this method to create an instance of the phantom proxy objects.  The return value will be an object with a page proxy and a phantom proxy.  These properties correspond to the phantom and webpage objects on the native phantom API.  

When this method is called, a new phantomjs process is spawned.  The new phantomjs process creates a mongoose webserver on localhost:1061.  All subsequent communication with phantom occurs via http requests. 
```javascript
var phantomProxy = require('phantom-proxy').create({}, function(proxy){
  var page = proxy.page,
  phantom = proxy.phantom;
  
});
```

#### phantom Object
The phantom object corresponds to the phantom object in the native phantomJs API.
#### exit(callbackFn)
Terminates phantom webserver and destroys proxy.
```javascript
require('phantom-proxy').createProxy({}, function(proxy){
  var page = proxy.page,
  phantom = proxy.phantom;
  
  page.open('http://www.w3.org', function(){
    console.log('page now open');
    phantom.exit(function(){
      console.log('phantom proxy is now offline');
    });
  }); 
});
```

#### Page Object
The page object corresponds to the webpage object in the native phantomJs API.

##### set(propertyName, propertyValue, callbackFn)
sets property on page object
```javascript
                //set viewport size for browser window
                proxy.page.set('viewportSize', 
                { width:320, height:480 }, function (result) {
                    console.log(result.toString().cyan);
                    worldCallback.call(self);
                })
```
##### open(url, callbackFn)
Opens a webpage with url and callback function arguments.
```javascript
require('phantom-proxy').createProxy({}, function(proxy){
  var page = proxy.page,
  phantom = proxy.phantom;
  
  page.open('http://www.w3.org', function(){
    console.log('page now open');
  });  
  
});
```
##### waitForSelector(selector, callbackFn)
Polls page for presence of selector, executes callback when selector is present.
```javascript
require('phantom-proxy').createProxy({}, function(proxy){
  var page = proxy.page,
  phantom = proxy.phantom;
  
  page.open('http://www.w3.org', function(){
    page.waitForSelector('body', function(){
      console.log('body tag present');
    });
    console.log('page now open');
  });  
  
});
```

##### render(fileName, callbackFn)
Renders a image of browser.
```javascript
require('phantom-proxy').createProxy({}, function(proxy){
  var page = proxy.page,
  phantom = proxy.phantom;
  
  page.open('http://www.w3.org', function(){
    page.waitForSelector('body', function(){
      console.log('body tag present');
      page.render('myimage.png', function(){
        console.log('saved my picture!');
      });
    });
    console.log('page now open');
  });  
  
});
```
##### renderBase64(type, callbackFn)
Returns a base64 representation of image. 

##### evaluate(functionToEvaluate, callbackFn, [arg1, arg2,... argN]
Executes functionToEvaluate in phantomJS browser.  Once function executes, callbackFn will be invoked with a result parameter. The Third and sebsequent arguments represent optional parameters which will be passed to the functionToEvaluate function when it is invoked in the browser.

#### Events
#### Subscribing to events
```javascript
phantomProxy = require('phantom-proxy');
phantomProxy.createProxy({}, function (proxy) {
    proxy.page.onConsoleMessage = function (event) {
        console.log(JSON.stringify(event));
    };
});
```

## FAQ
### Why do we need another nodejs runtime for phantom?
The short answer is that phantom-proxy has a better implementation.  Other drivers seem to use a side effect of alerts to communicate with phantomjs, which unfortunately is not a good long term solution. Also all the other libraries I have seen have not fully implemented the evaluate function.  For whatever reason, the other nodejs phantom driver evaluate implementations do have a parametrized client side evaluation function - this is a big problem and was the main motivation for creating this package.

