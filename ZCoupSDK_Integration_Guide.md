# ZCoup SDK Integration

1. [Introduction](#introduction)

2. [Integration Notes](#note)

3. [Integration ZCoup SDK](#integration)

    [3.1 Integrating the ZCoup SDK in to project](#step1)

    [3.2 Initialize the Zcoup SDK](#step2)

    [3.3 Android code obfuscation](#step3)

4. [Request Ad](#request)

    [4.1 Native](#native)
    * [Element-Native](#common)
    * [Element-Native with adCache](#cache)
    * [Element-Native for Multiple](#multi)
    
    [4.2 Banner](#banner)
    [4.3 Interstitial](#interstitial)
    [4.4 Appwall](#appwall)
    [4.5 Rewarded Video](#reward)
    [4.6 Native Video](#native_video)
5. [Error Code For SDK](#error)

## <a name="introduction">Introduction</a>

- ZCoup SDK supports Banner, Interstitial, Native, Native Video andRewarded Video.
- ZCoup Android SDK supports Android API 15+.
- Please make sure you have added an app and at least one ad slot in ZCoup Platform.
- Please download [our latest SDK](https://github.com/zero-sdk/Android_SDK/blob/master/AndroidSDK.zip) .

## <a name="integration">Integration ZCoup SDK</a>  

### <a name="step1">2.1 Integrating the ZCoup SDK in to project</a>

* [Download the latest SDK](<https://github.com/zero-sdk/Android_SDK/blob/master/ZcoupSDK.zip> )

* Detail of the different jars：

  | jar name                 | jar function                                    | require(Y/N) |
  | ------------------------ | ----------------------------------------------- | ------------ |
  | zcoup_base_xx.jar        | basic functions(banner\interstitial\native ads) | Y            |
  | zcoup_imageloader_xx.jar | imageloader functions                           | N            |
  | zcoup_appwall_xx.jar     | appwall ads functions                           | N            |
  | zcoup_video_xx.jar       | video ads functions                             | N            |

* Configure the module's build.gradle for basic functions：

``` groovy
    dependencies {
        compile files('libs/zcoup_base_xx.jar')
    }
```

* Configure AndroidManifest.xml

```xml
	<!--Necessary Permissions-->
	<uses-permission android:name="android.permission.INTERNET"/>
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

	<!-- Necessary -->
	<activity android:name="com.zcoup.base.view.InnerWebViewActivity" />

	<!--Optional-->
	<receiver android:name="com.zcoup.base.receiver.UtilityReceiver">
    	<intent-filter>
        	<action android:name="android.intent.action.PACKAGE_ADDED" />
        	<data android:scheme="package" />
    	</intent-filter>
	</receiver>

	<!--If your targetSdkVersion is 28, you need update <application> as follows:-->
  	<application
    	...  	
        android:usesCleartextTraffic="true"
        ...>
        ...
    </application>
```



## <a name="step2">2.2 Initialization</a>  

> Init the SDK in your Application as detailed below: 

**Please make sure to initialize ZCoup SDK before doing anything.**

```java
    ZcoupSDK.initialize(context, "Your slotID");
```

**Use this interface to upload consent from affected users for GDPR**

```java
    /**
     * @param context       context
     * @param consentValue  whether the user agrees
     * @param consentType   the agreement name signed with users
     * @param listener      callback
     */
     ZcoupSDK.uploadConsent(this, true, "GDPR", new HttpRequester.Listener() {
            @Override
            public void onGetDataSucceed(byte[] data) {
            }


            @Override
            public void onGetDataFailed(String error) {
            }
     });
```

## <a name="step3">2.3 Obfuscation Configuration</a> 
> If it needs to obfuscate the codes in building the project process, you should add the following codes into the proguard file:

``` java
    #for sdk
    -keep public class com.zcoup.**{*;}
    -dontwarn com.zcoup.**

    #for gaid
    -keep class **.AdvertisingIdClient$** { *; }

    #for js and webview interface
    -keepclassmembers class * {
        @android.webkit.JavascriptInterface <methods>;
    }
    
```


## <a name="note">Integration Notes</a>

​	If you live in a country, such as China, which is forbidden google play, two prerequisites to get Zcoup ads: 
> * GooglePlay has installed on your device.
> * Your device have connect to VPN.

   

​	We suggest you define a class to implement the CTAdEventListener yourself , then you can just override the methods you need when you getBanner or getNative. See the following example:

``` java
public class MyCTAdEventListener extends CTAdEventListener {
    @Override
    public void onReceiveAdSucceed(ZCNative result) {
    }

    @Override
    public void onReceiveAdVoSucceed(AdsNativeVO result) {
    }

    @Override
    public void onInterstitialLoadSucceed(ZCNative result) {
    }

    @Override
    public void onReceiveAdFailed(ZCNative result) {
        Log.i("sdksample", "==error==" + result.getErrorsMsg());
    }

    @Override
    public void onLandpageShown(ZCNative result) {
    }

    @Override
    public void onAdClicked(ZCNative result) {
    }

    @Override
    public void onAdClosed(ZCNative result) {
    }
}
```

## <a name="native">4.1 Native Ads Integration</a>

### <a name="common">Native ads interface</a>

> The container and the layout for Native ad:

```java
    ViewGroup container = (ViewGroup) view.findViewById(R.id.container);
    ViewGroup adLayout = (ViewGroup)View.inflate(context,R.layout.native_layout, null);
```

> The method to load Native Ads:

``` java
    /**
     * @param slotId     cloudmobi id
     * @param context    context
     * @param adListener callback listener 
     */
 	ZcoupSDK.getNativeAd("Your slotID", context, new MyCTAdEventListener(){
        @Override
        public void onReceiveAdSucceed(ZCNative result) {
            if (result == null) {
                return;
            }
            ZCAdvanceNative zcAdvanceNative = (ZCAdvanceNative) result;
            showAd(zcAdvanceNative);
            ZCLog.e("onReceiveAdSucceed");
            super.onReceiveAdSucceed(result);
        }
     });
```

* Show the Native ad

``` java
   private void showAd(ZCAdvanceNative zcAdvanceNative) {
        ImageView img = (ImageView) adLayout.findViewById(R.id.iv_img);
        ImageView icon = (ImageView) adLayout.findViewById(R.id.iv_icon);
        TextView title = (TextView)adLayout.findViewById(R.id.tv_title);
        TextView desc = (TextView)adLayout.findViewById(R.id.tv_desc);
        Button click = (Button)adLayout.findViewById(R.id.bt_click);
        ImageView adChoice = (ImageView)adLayout.findViewById(R.id.choice_icon);
        
        //show the image and icon yourself 
        String imageUrl = zcAdvanceNative.getImageUrl();
        String iconUrl = zcAdvanceNative.getIconUrl();          
        title.setText(zcAdvanceNative.getTitle());
        desc.setText(zcAdvanceNative.getDesc());
        click.setText(zcAdvanceNative.getButtonStr());
        adChoice.setImageURI(zcAdvanceNative.getAdChoiceIconUrl());
        //offerType（1 : download ads; 2 : content ads）
        int offerType = zcAdvanceNative.getOfferType();  
         
        zcAdvanceNative.registeADClickArea(adLayout);
        container.addView(adLayout);
   }
```



### <a name="cache">Native ads interface for AdCache</a>

> Get Ads for cache

```java
    /**
     * @param slotId     cloudmobi id
     * @param context    context
     * @param adListener callback listener 
     */
    ZcoupSDK.getNativeAdForCache("Your slotID",context,new MyCTAdEventListener() {
        @Override
        public void onReceiveAdVoSucceed(AdsNativeVO result) {
            if (result == null) {
                return;
            }
            ZCLog.e("onReceiveAdVoSucceed");
            AdHolder.adNativeVO = result;
            super.onReceiveAdVoSucceed(result);
        }
    });

```

> Show ad from cache

```java
      ZCAdvanceNative zcAdvanceNative = new ZCAdvanceNative(getContext());
      AdsNativeVO nativeVO = AdHolder.adNativeVO;

	  if (nativeVO != null) {
            zcAdvanceNative.setNativeVO(nativeVO);
            zcAdvanceNative.setSecondAdEventListener(new MyCTAdEventListener() {
                @Override
                public void onAdClicked(com.zcoup.base.core.ZCNative result) {
                    ZCLog.e("onAdClicked");
                    super.onAdClicked(result);
                }
            });
            showAd(zcAdvanceNative);
       }
```



### <a name="multi">Multi Native ad interface</a>

> The method to load multi Native ad

``` java
    /**
     * @param reqAdNumber request ads num
     * @param slotId      cloudmobi id
     * @param context     context
     * @param adListener  callback listener 
     */
	ZcoupSDK.getNativeAds(10, "Your slotID", getContext(), new MultiAdsEventListener() {
        public void onMultiNativeAdsSuccessful(List<ZCAdvanceNative> res) {
        }

        @Override
        public void onReceiveAdFailed(ZCNative result) {
        }

        @Override
        public void onAdClicked(ZCNative result) {
            super.onAdClicked(result);
        }
    });
```



## <a name="banner">4.2 Banner Ad Integration</a>

> The method to load Banner Ad:

``` java
	ViewGroup container = (ViewGroup) view.findViewById(R.id.container);

    /**
     * @param context           context
     * @param slotId            cloudmobi id
     * @param adSize			AdSize.AD_SIZE_320X50,
     							AdSize.AD_SIZE_320X100,
     							AdSize.AD_SIZE_300X250;
     * @param adListener        callback listener 
     */
	 ZcoupSDK.getBannerAd(context, "Your slotID", adSize,new MyCTAdEventListener() {
         @Override
         public void onReceiveAdSucceed(ZCNative result) {
             if (result != null) {
                 container.setVisibility(View.VISIBLE);
                 container.removeAllViews();
                 container.addView(result);   //把广告添加到容器
             }
             super.onReceiveAdSucceed(result);
         }


         @Override
         public void onReceiveAdFailed(ZCNative result) {
             super.onReceiveAdFailed(result);
         }


         @Override
         public void onAdClicked(ZCNative result) {
             super.onAdClicked(result);
         }


         @Override
         public void onAdClosed(ZCNative result) {
             container.removeAllViews();
             container.setVisibility(View.GONE);
             super.onAdClosed(result);
         }
     });
```

> When you successfully integrated the Banner Ad, you will see the ads are like this


![-1](https://user-images.githubusercontent.com/20314643/42366029-b6289f2a-8132-11e8-9c3e-86557d164d85.png)
![320x100](https://user-images.githubusercontent.com/20314643/42370991-c4188812-8140-11e8-80e9-ab6947c12e92.png)
![300x250](https://user-images.githubusercontent.com/20314643/42370999-c74139f8-8140-11e8-91ff-ba0cdb0ae08a.png)



## <a name="interstitial">4.3 Interstitial Ads Integration</a>

> Configure the AndroidManifest.xml for Interstitial

```xml
	<activity android:name="com.zcoup.base.view.InterstitialActivity" />    
```

> The method to show Interstitial Ads

``` java
    /**
     * @param context           context
     * @param slotId            cloudmobi id
     * @param adListener        callback listener 
     */
    ZcoupSDK.preloadInterstitialAd(context, "Your slotID",new MyCTAdEventListener() {
                    
        @Override
        public void onReceiveAdSucceed(ZCNative result) {              
            if (result != null && result.isLoaded()) {
                Log.w(TAG, "onReceiveAdSucceed");
                if (ZcoupSDK.isInterstitialAvailable(result)) {
            		ZcoupSDK.showInterstitialAd(result);
        		}
            }
            super.onReceiveAdSucceed(result);
        }

        @Override 
        public void onLandpageShown(ZCNative result) {
            super.onLandpageShown(result);
            Log.e(TAG, "onLandpageShown:");
        }


        @Override
        public void onReceiveAdFailed(ZCNative error) {
            Log.w(TAG, "onReceiveAdFailed: " + error.getErrorsMsg());
            super.onReceiveAdFailed(error);
        }


        @Override
        public void onAdClosed(ZCNative result) {
            super.onAdClosed(result);
            Log.e(TAG, "onAdClosed:");
        }
    });
```

> When you successfully integrated the Interstitial Ad, you will see the ads are like this

![image](https://user-images.githubusercontent.com/20314643/41895879-b4536200-7955-11e8-9847-587f175c4a54.png)
![image](https://user-images.githubusercontent.com/20314643/41895941-e0c6ad1a-7955-11e8-9393-ed91e4a4906f.png)



## <a name="appwall">4.4 Appwall integration</a>

> Configure the module's build.gradle for Appwall：

``` groovy
	dependencies {
        	compile files('libs/zcoup_base_xx.jar')
        	compile files('libs/cloudssp_appwall_xx.jar')       // for appwall        
        	compile files('libs/cloudssp_imageloader_xx.jar')   // for imageloader
	}
```

> Configure the AndroidManifest.xml for Appwall

``` xml
    <activity
            android:name="com.zcoup.appwall.AppwallActivity"
            android:screenOrientation="portrait" />  
```

> Preload appwall

> It‘s better to preload ads for Appwall, to ensure they show properly and in a timely fashion. You can have ads preload with the following line of code:

``` java
   ZcoupAppwall.preloadAppwall(context, "Your slotID");
```

> Customize the appwall color theme(optional).

``` java
    CustomizeColor custimozeColor = new CustomizeColor();
    custimozeColor.setMainThemeColor(Color.parseColor("#ff0000ff"));
    ZcoupAppwall.setThemeColor(custimozeColor);
```

> Show Appwall.

``` java
     ZcoupAppwall.showAppwall(context, "Your slotID");
```

> When you successfully integrated the APP Wall Ad, you will see the ads are like this

![image](https://user-images.githubusercontent.com/20314643/42366246-47526c9c-8133-11e8-963c-bd0eb7a3e1a6.png)



## <a name="reward">4.5 Rewarded Video Ad Integration</a>

> **Google Play Services**

1. Google Advertising ID

 The Rewarded Video function requires access to the Google Advertising ID in order to operate properly.
     See this guide on how to integrate [Google Play Services](https://developers.google.com/android/guides/setup).

2. Google Play Services in Your Android Manifest

    Add the following  inside the <application> tag in your AndroidManifest:

    ```
    <meta-data
    	android:name="com.google.android.gms.ads.AD_MANAGER_APP"
        android:value="true" />
    ```

3. If you have integrated the admob-sdk for basic ads, it's not necessary to do this.


> Configure the module's build.gradle for Rewarded Video：

``` groovy
	dependencies {
        	compile files('libs/zcoup_base_xx.jar')
        	compile files('libs/zcoup_video_xx.jar')
        	compile files('libs/zcoup_imageloader_xx.jar')
	}
```

> Configure the AndroidManifest.xml for Rewarded Video:

``` xml    
<activity
	android:name="com.zcoup.video.view.RewardedVideoActivity"           android:configChanges="keyboard|keyboardHidden|orientation|screenLayout|uiMode|screenSize|smallestScreenSize" />
```

> Setup the UserID

> You should set the userID for s2s-postback before preloading the RewardedVideo function call


```java
	ZcoupVideo.setUserId("custom_id");
```

> Preload the RewardedVideo

> It‘s best to call this interface when you want to show RewardedVideo ads to the user. This will help reduce latency and ensure effective, prompt delivery of ads


``` java
 	ZcoupVideo.preloadRewardedVideo(getContext(), Config.slotIdReward,
                new VideoAdLoadListener() {
                    @Override
                    public void onVideoAdLoadSucceed(ZCVideo videoAd) {
                        video = videoAd;
                        Log.w(TAG, "onVideoAdLoadSucceed: ");
                    }


                    @Override
                    public void onVideoAdLoadFailed(ZCError error) {
                        Log.w(TAG, "onVideoAdLoadFailed: " + error.getMsg());
                    }
                });

```

> Show Rewarded Video ads to Your Users
> Before showing the video, you can request or query the video status by calling:

```java
    boolean available = ZcoupVideo.isRewardedVideoAvailable(video);
```

> Because it takes a while to load the video creatives. You may need to wait until video creatives loads completely. Once you get an available Reward Video, you are ready to show this video ad to your users by calling the showRewardedVideo() method as in the following example:

```java
  ZcoupVideo.showRewardedVideo(video, new RewardedVideoAdListener() {
                @Override
                public void videoStart() {
                    Log.e(TAG, "videoStart: ");
                }

                @Override
                public void videoFinish() {
                    Log.e(TAG, "videoFinish: ");
                }

                @Override
                public void videoError(Exception e) {
                    Log.e(TAG, "videoError: ");
                }

                @Override
                public void videoClosed() {
                    Log.e(TAG, "videoClosed: ");
                }

                @Override
                public void videoClicked() {
                    Log.e(TAG, "videoClicked: ");
                }

                @Override
                public void videoRewarded(String rewardName, String rewardAmount) {
                    Log.e(TAG, "videoRewarded: ");
                }
            });

```

> Reward the User

The SDK will fire the videoRewarded event each time the user successfully completes a video. The RewardedVideoListener will be in place to receive this event so you can reward the user successfully.

The Reward object contains both the Reward Name & Reward Amount of the SlotId as defined in your ZCoup SSP:

```java

public void videoRewarded(String rewardName, String rewardAmount) {
      //TODO - here you can reward the user according to the given amount
       Log.e(TAG, "videoRewarded: ");
   }
    
```

> When you successfully integrated the Rewarded Video, you will see the ads are like this

![image](https://user-images.githubusercontent.com/20314643/42371626-94e8e6a2-8142-11e8-9598-eb75de753070.png)


## <a name="native_video">4.6 Native Video Ad Integration</a>

> Configure the module's build.gradle for Native Video：

```groovy
	dependencies {
        compile files('libs/zcoup_base_xx.jar')
        compile files('libs/zcoup_video_xx.jar')
	}
```

> Load the NativeVideo

```java
   ZcoupVideo.getNativeVideo(context, Config.slotIdNativeVideo, new VideoAdLoadListener(){
            @Override
            public void onVideoAdLoadSucceed(ZCVideo video) {
                showNativeVideo((ZCNativeVideo) video);
            }


            @Override
            public void onVideoAdLoadFailed(ZCError error) {
            }
        });

```

> Show the NativeVideo

```java
    private void showNativeVideo(ZCNativeVideo zcNativeVideo) {
        if (video == null) {
            return;
        }
        
        //layout for NativeVideo
        ViewGroup videoLayout = (ViewGroup) View.inflate(context, R.layout.native_video_layout, null);
      
        //whether autoplay for 4G
        zcNativeVideo.setWWANPlayEnabled(false);

        TextView video_title = videoLayout.findViewById(R.id.video_title);
        RelativeLayout video_container = videoLayout.findViewById(R.id.video_container);
        SimpleDraweeView video_choice = videoLayout.findViewById(R.id.video_choice);
        TextView video_desc = videoLayout.findViewById(R.id.video_desc);
        TextView video_button = videoLayout.findViewById(R.id.video_button);

        video_title.setText(zcNativeVideo.getTitle());
        video_container.addView(zcNativeVideo.getNativeVideoAdView());
        video_choice.setImageURI(Uri.parse(zcNativeVideo.getAdChoiceIconUrl()));
        video_desc.setText(Html.fromHtml(zcNativeVideo.getDesc()));
        video_button.setText(zcNativeVideo.getButtonStr());

        //register for tracking， videolayout is the clickarea
        zcNativeVideo.registerForTracking(videoLayout, 
            new NativeVideoAdListener() {
                @Override
                public void videoPlayBegin() {
                }

                @Override
                public void videoPlayFinished() {
                }

                @Override
                public void videoPlayClicked() {
                }

                @Override
                public void videoPlayError(Exception e) {
                }
        });

        //container in your app
        rl_container.removeAllViews();
        rl_container.addView(videoLayout);
    }
```

