
Cordova AdMob plugin<br>[![License][license]][npm-url] [![NPM version][npm-version]][npm-url] [![NPM downloads][npm-downloads]][npm-url]
====================

[npm-url]: https://www.npmjs.com/package/cordova-admob
[license]: http://img.shields.io/badge/license-MIT-blue.svg?style=flat
[npm-version]: https://img.shields.io/npm/v/cordova-admob.svg?style=flat
[npm-downloads]: https://img.shields.io/npm/dm/cordova-admob.svg?style=flat

## It simply works :)
Monetize your Cordova/Phonegap/XDK HTML5 hybrid apps and games with AdMob ads, **using latest Google AdMob SDK**.
- Now available with Ionic Native too
- Supports banner, interstitials and rewarded
- Optional [Tappx](http://www.tappx.com/?h=dec334d63287772de859bdb4e977fce6) backfill

With this Cordova/Phonegap/XDK plugin you can show AdMob ads as easy as:

```js
    admob.createBannerView({publisherId: "ca-app-pub-XXXXXXXXXXXXXXXX/BBBBBBBBBB"});
```

---
## Plugin update (phonegap/cordova cli) ##

`cordova-admob~4.1.15` and later are now updated to Firebase SDK (ios 7.13.1 and later, android managed by gradle)

To update the plugin you should remove the plugin and add it again:

```
$ cordova plugin rm cordova-admob
$ npm cache clear
$ cordova plugin add cordova-admob
```

Sometimes removing the plugin causes an error (it's been reported to cordova https://issues.apache.org/jira/browse/CB-12083). If that happens, remove first `cordova-libgoogleadmobads` manually:


```
$ rm plugins/cordova-libgoogleadmobads/ -rf
$ cordova plugin rm cordova-admob
$ npm cache clear
$ cordova plugin add cordova-admob
```

---
## Platform SDK supported ##

* iOS, using AdMob SDK for iOS, v7.13.1
* Android, using latest Google Play Services for Android (managed by gradle)


---
## Quick start ##

To install this plugin, follow the [Command-line Interface Guide](http://cordova.apache.org/docs/en/edge/guide_cli_index.md.html#The%20Command-line%20Interface). You can use one of the following command lines:

* `cordova plugin add cordova-admob`
* `cordova plugin add https://github.com/appit-online/appit-cordova-admob.git`


To use in [Phonegap Build](https://build.phonegap.com), place the following tag in your `config.xml` file:

```xml
<gap:plugin name="phonegap-admob" source="npm"/>
```

To start showing ads, place the following code in your `onDeviceReady` callback. Replace corresponding id's with yours:

*Note: ensure you have a proper [AdMob](https://apps.admob.com/admob/signup) and [tappx](http://www.tappx.com/?h=dec334d63287772de859bdb4e977fce6) accounts and get your publisher id's*.

```javascript
    
    function onDeviceReady() {
      document.removeEventListener('deviceready', onDeviceReady, false);
      
      // Set AdMobAds options:
      admob.setOptions({
        publisherId:           "ca-app-pub-XXXXXXXXXXXXXXXX/BBBBBBBBBB",  // Required
        interstitialAdId:      "ca-app-pub-XXXXXXXXXXXXXXXX/IIIIIIIIII",  // Optional
        autoShowBanner:        true,                                      // Optional
        autoShowRInterstitial: false,                                     // Optional
        autoShowRewarded:      false,                                     // Optional
        tappxIdiOS:            "/XXXXXXXXX/Pub-XXXX-iOS-IIII",            // Optional
        tappxIdAndroid:        "/XXXXXXXXX/Pub-XXXX-Android-AAAA",        // Optional
        tappxShare:            0.5                                        // Optional
      });
      
      // Start showing banners (atomatic when autoShowBanner is set to true)
      admob.createBannerView();
      
      // Request interstitial ad (will present automatically when autoShowInterstitial is set to true)
      admob.requestInterstitialAd();

      // Request rewarded ad (will present automatically when autoShowRewarded is set to true)
      admob.requestRewardedAd();
    }
    
    document.addEventListener("deviceready", onDeviceReady, false);
```


:warning: Be sure to start ads on "deviceready" event otherwise, the plugin would not work.


---
## Complete example code ##
Note that the admob ads are configured inside `onDeviceReady()`. This is because only after device ready the AdMob Cordova plugin will be working.

```javascript

    var isAppForeground = true;
    
    function initAds() {
      if (admob) {
        var adPublisherIds = {
          ios : {
            banner : "ca-app-pub-XXXXXXXXXXXXXXXX/BBBBBBBBBB",
            interstitial : "ca-app-pub-XXXXXXXXXXXXXXXX/IIIIIIIIII"
          },
          android : {
            banner : "ca-app-pub-XXXXXXXXXXXXXXXX/BBBBBBBBBB",
            interstitial : "ca-app-pub-XXXXXXXXXXXXXXXX/IIIIIIIIII"
          }
        };
    	  
        var admobid = (/(android)/i.test(navigator.userAgent)) ? adPublisherIds.android : adPublisherIds.ios;
            
        admob.setOptions({
          publisherId:          admobid.banner,
          interstitialAdId:     admobid.interstitial,
          autoShowBanner:       true,
          autoShowInterstitial: false,
          autoShowRewarded:     false,
          tappxIdiOS:           "/XXXXXXXXX/Pub-XXXX-iOS-IIII",
          tappxIdAndroid:       "/XXXXXXXXX/Pub-XXXX-Android-AAAA",
          tappxShare:           0.5,
        });

        registerAdEvents();
        
      } else {
        alert('AdMobAds plugin not ready');
      }
    }
    
    function onAdLoaded(e) {
      if (isAppForeground) {
        if (e.adType === admob.AD_TYPE.AD_TYPE_BANNER) {
          console.log("New banner received");
        } else if (e.adType === admob.INTERSTITIAL) {
          console.log("An interstitial has been loaded and autoshown. If you want to automatically show the interstitial ad, set 'autoShowInterstitial: true' in admob.setOptions() or remove it");
          admob.showInterstitialAd();
        } else if (e.adType === admob.AD_TYPE_REWARDED) {
          console.log("New rewarded ad received");
          admob.showRewardedAd();
        }
      }
    }
    
    function onPause() {
      if (isAppForeground) {
        admob.destroyBannerView();
        isAppForeground = false;
      }
    }
    
    function onResume() {
      if (!isAppForeground) {
        setTimeout(admob.createBannerView, 1);
        setTimeout(admob.requestInterstitialAd, 1);
        isAppForeground = true;
      }
    }
    
    // optional, in case respond to events
    function registerAdEvents() {
      document.addEventListener(admob.events.onAdLoaded, onAdLoaded);
      document.addEventListener(admob.events.onAdFailedToLoad, function (e) {});
      document.addEventListener(admob.events.onAdOpened, function (e) {});
      document.addEventListener(admob.events.onAdClosed, function (e) {});
      document.addEventListener(admob.events.onAdLeftApplication, function (e) {});
      
      document.addEventListener("pause", onPause, false);
      document.addEventListener("resume", onResume, false);
    }
        
    function onDeviceReady() {
      document.removeEventListener('deviceready', onDeviceReady, false);
      initAds();

      // display a banner at startup
      admob.createBannerView();
        
      // request an interstitial ad
      admob.requestInterstitialAd();

      // request a rewarded ad
      admob.requestRewardedAd();
    }
    
    document.addEventListener("deviceready", onDeviceReady, false);
```

---
## Contributing ##
You can use this cordova plugin for free. You can contribute to this project in many ways:

* Testimonials of apps that are using this plugin gives your app free marketing and will be especially helpful. [Open an issue](https://github.com/appit-online/appit-cordova-admob/issues).
* [Reporting issues](https://github.com/appit-online/appit-cordova-admob/issues).
* Patching and bug fixing, especially when submitted with test code. [Open a pull request](https://github.com/appit-online/appit-cordova-admob/pulls).
* Other enhancements.

You can also support this project by sharing 2% Ad traffic (it's not mandatory: if you are unwilling to share, please fork and remove the donation code).

Love the project? Wanna buy me a coffee (or a beer :D)? [Click here](https://paypal.me/dave7117/5)

---
## License ##
```
The MIT License

Copyright (c) 2014 AppIT

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

