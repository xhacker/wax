Wax is being maintained by alibaba
----------

Thank @probablycorey for creating such a greate project.
Wax is the best bridge between Lua and Objective-C, which should not be declining, so we will maintaine it. We have fixed a lot of problem such as 64 bit support, muti-thread safe. We have also add many features such as  convert Lua function to OC block , call OC block in Lua, get/set private ivar, inbuilt commonly used C function, and even Lua code debug.

Wax
---

Wax is a framework that lets you write native iPhone apps in 
[Lua](http://www.lua.org/about.html). It bridges Objective-C and Lua using the 
Objective-C runtime. With Wax, **anything** you can do in Objective-C is **automatically**
available in Lua! What are you waiting for, give it a shot!

Why write iPhone apps in Lua?
-----------------------------

I love writing iPhone apps, but would rather write them in a dynamic language than in Objective-C. Here 
are some reasons why many people prefer Lua + Wax over Objective-C...

* Automatic Garbage Collection! Gone are the days of alloc, retain, and release.

* Less Code! No more header files, no more static types, array and dictionary literals! 
  Lua enables you to get more power out of less lines of code.

* Access to every Cocoa, UITouch, Foundation, etc.. framework, if it's written in Objective-C, 
  Wax exposes it to Lua automatically. All the frameworks you love are all available to you!

* Super easy HTTP requests. Interacting with a REST webservice has never been eaiser

* Lua has closures, also known as blocks! Anyone who has used these before knows how powerful they can be.

* Lua has a build in Regex-like pattern matching library.

Examples
--------

For some simple Wax apps, check out the [examples folder](https://github.com/alibaba/wax/tree/master/examples).

How would I create a UIView and color it red?

    -- forget about using alloc! Memory is automatically managed by Wax
    view = UIView:initWithFrame(CGRect(0, 0, 320, 100))

    -- use a colon when sending a message to an Objective-C Object
    -- all methods available to a UIView object can be accessed this way
    view:setBackgroundColor(UIColor:redColor())

What about methods with multiple arguments?

    -- Just add underscores to the method name, then write the arguments like
    -- you would in a regular C function
    UIApplication:sharedApplication():setStatusBarHidden_animated(true, false)

How do I send an array/string/dictionary

    -- Wax automatically converts array/string/dictionary objects to NSArray,
    -- NSString and NSDictionary objects (and vice-versa)
    images = {"myFace.png", "yourFace.png", "theirFace.png"}
    imageView = UIImageView:initWithFrame(CGRect(0, 0, 320, 460))
    imageView:setAnimationImages(images)

What if I want to create a custom UIViewController?

    -- Created in "MyController.lua"
    --
    -- Creates an Objective-C class called MyController with UIViewController
    -- as the parent. This is a real Objective-C object, you could even
    -- reference it from Objective-C code if you wanted to.
    waxClass{"MyController", UIViewController}

    function init()
      -- to call a method on super, simply use self.super
      self.super:initWithNibName_bundle("MyControllerView.xib", nil)
      return self
    end

    function viewDidLoad()
      -- Do all your other stuff here
    end

You said HTTP calls were easy, I don't believe you...

    url = "http://search.twitter.com/trends/current.json"

    -- Makes an asyncronous call, the callback function is called when a
    -- response is received
    wax.http.request{url, callback = function(body, response)
      -- request is just a NSHTTPURLResponse
      puts(response:statusCode())

      -- Since the content-type is json, Wax automatically parses it and places
      -- it into a Lua table
      puts(body)
    end}

Since Wax converts NSString, NSArray, NSDictionary and NSNumber to native Lua values, you have to force objects back to Objective-C sometimes. Here is an example.

    local testString = "Hello lua!"
    local bigFont = UIFont:boldSystemFontOfSize(30)
    local size = toobjc(testString):sizeWithFont(bigFont)
    puts(size)
    
How do I convert Lua function to Objective-C block? 

```
UIView:animateWithDuration_animations_completion(1, 
	toblock(
		function()
			label:setCenter(CGPoint(300, 300))
		end
	),
	toblock(
		 	function(finished)
				print('lua animations completion ' .. tostring(finished))
			end
	,{"void", "BOOL"})
)

---OC method is -(void)testReturnIdWithFirstIdBlock:(id(^)(id aFirstId, BOOL aBOOL, int aInt, NSInteger aInteger, float aFloat, CGFloat aCGFloat, id aId))block
self:testReturnIdWithFirstIdBlock(
toblock(
	function(aFirstId, aBOOL, aInt, aInteger, aFloat, aCGFloat, aId)
		print("aFirstId=" .. tostring(aFirstId))
		-- assert(aFirstId == self, "aFirstInt不等")
		assertResult(self, aBOOL, aInt, aInteger, aFloat, aCGFloat, aId)
		print("LUA TEST SUCCESS: testReturnIdWithFirstIdBlock")
		return aFirstId
	end
	, {"id","id", "BOOL", "int", "NSInteger", "float", "CGFloat", "id"})
)
```
What about call the Objective-C block?

```
--OC block type is id (^)(NSInteger, id, BOOL, CGFloat)
local res = luaCallBlock(block, 123456, aObject, true, 123.456,);

--or you can call like this:
local res = luaCallBlockWithParamsTypeArray(block, {"id","NSInteger", "id", "BOOL", "CGFloat"},  123456, aObject, true, 123.456);
```

If my instance variable is private implementated?

```
print(self:view():getIvarInt("_countOfMotionEffectsInSubtree"))

self:setIvar_withObject("_title", "abcdefg")
print(self:getIvarObject("_title"))

self:setIvar_withObject("_infoDict", {k11="v11", k22="v22"})
print(toobjc(self:getIvarObject("_infoDict")))
```
You want to call some C function?

```
luaSetWaxConfig({wax_openBindOCFunction=true})--bind built-in C function

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), 
        toblock(
            function( )
                print(string.format("dispatch_async"));
            end)
    );

UIGraphicsBeginImageContext(CGSize(200,400));
UIApplication:sharedApplication():keyWindow():layer():renderInContext(UIGraphicsGetCurrentContext());
local aImage =UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();

local imageData =UIImagePNGRepresentation(aImage);
print("imageData.length=", imageData.length);
local image = aImage;
local data = UIImageJPEGRepresentation(image, 0.8);
print("data.length=", data:length());

```
Any way to debug my Lua code?   
Ofcourse, you can use the powerfull ZeroBraneStudio to debug. [see more detail](https://github.com/alibaba/wax/tree/master/examples/LuaCodeDebug).

Use with cocoapods
----------
see examples/TestWaxPod demo.  
* add `pod 'wax', :git=>'git@github.com:alibaba/wax.git', :tag=>'1.0.0'` to your podfile. (tag in your needs)  
* then you can run lua code.

```
wax_start(nil, nil);
wax_runLuaString("print('hello wax')");
```
	

Setup & Tutorials
-----------------

[Setting up Wax](https://github.com/probablycorey/wax/wiki/Installation)

[How does Wax work?](https://github.com/probablycorey/wax/wiki/Overview)

[Simple Twitter client in Wax](https://github.com/probablycorey/wax/wiki/Twitter)

Which API's are included?
-------------------------

They all are! I can't stress this enough, anything that is written in Objective-C (even external frameworks) will work automatically in Wax! UIAcceleration works like UIAcceleration, MapKit works like MapKit, GameKit works like GameKit, Snozberries taste like Snozberries!

Created By
----------
Corey Johnson (probablycorey at gmail dot com)

More
----
* [Feature Requests? Bugs?](http://github.com/probablycorey/wax/issues) - Issue tracking and release planning.
* [Mailing List](http://groups.google.com/group/iphonewax)
* Quick question or problem? Send email to [@Zhengwei Yin (Junzhan)](mailto:junzhan.yzw@taobao.com)

Contribute
----------
[@Zhengwei Yin (Junzhan)](mailto:junzhan.yzw@taobao.com)  
Fork it, change it, commit it, push it, send pull request; instant glory!


The MIT License
---------------
Wax is Copyright (C) 2009 Corey Johnson See the file LICENSE for information of licensing and distribution.
