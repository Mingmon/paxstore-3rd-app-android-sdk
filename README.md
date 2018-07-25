# PAXSTORE 3rd App Android SDK [ ![Download](https://api.bintray.com/packages/paxstore-support/paxstore/paxstore-3rd-app-android-sdk/images/download.svg?version=6.00.00) ](https://bintray.com/paxstore-support/paxstore/paxstore-3rd-app-android-sdk/6.00.00/link)
PAXSTORE 3rd App Android SDK provides simple and easy-to-use service interfaces for third party developers to develop android apps on PAXSTORE. The services currently include the following points:

1. Download parameter
2. Inquire update for 3rd party app

By using this SDK, developers can easily integrate with PAXSTORE. Please take care of your AppKey and AppSecret that generated by PAXSTORE system when you create an app.
<br>Refer to the following steps for integration.

## Requirements
**Android SDK version**
>SDK 19 or higher, depending on the terminal's paydroid version.

**Gradle's and Gradle plugin's version**
>Gradle version 4.1 or higher
>Gradle plugin version 3.0.0+ or higher

## Download
Gradle:

    implementation 'com.pax.market:paxstore-3rd-app-android-sdk:6.00.00'

## Permissions
PAXSTORE Android SDK need the following permissions, please add them in AndroidManifest.xml.

`<uses-permission android:name="android.permission.INTERNET" />`<br>
`<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />`<br>
`<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />`<br>

## ProGuard
If you are using [ProGuard](https://www.guardsquare.com/en/products/proguard/manual) in your project add the following lines to your configuration:

    #Gson
    -dontwarn com.google.gson.**
    -keep class sun.misc.Unsafe { *; }
    -keep class com.google.gson.** { *; }
    -keep class com.google.gson.examples.android.model.** { *; }

    #JJWT
    -keepnames class com.fasterxml.jackson.databind.** { *; }
    -dontwarn com.fasterxml.jackson.databind.*
    -keepattributes InnerClasses
    -keep class org.bouncycastle.** { *; }
    -keepnames class org.bouncycastle.** { *; }
    -dontwarn org.bouncycastle.**
    -keep class io.jsonwebtoken.** { *; }
    -keepnames class io.jsonwebtoken.* { *; }
    -keepnames interface io.jsonwebtoken.* { *; }
    -dontwarn javax.xml.bind.DatatypeConverter
    -dontwarn io.jsonwebtoken.impl.Base64Codec
    -keepnames class com.fasterxml.jackson.** { *; }
    -keepnames interface com.fasterxml.jackson.** { *; }

## API Usage

### Step 1: Get Application Key and Secret
Create a new app in PAXSTORE, and get **AppKey** and **AppSecret** from app detail page in developer center.

### Step 2: Initialization
Configuring the application element, edit AndroidManifest.xml, it will have an application element. You need to configure the android:name attribute to point to your Application class (put the full name with package if the application class package is not the same as manifest root element declared package)

    <application
        android:name=".BaseApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">
        
Initializing AppKey,AppSecret and SN
>Please note, make sure you have put your own app's AppKey and AppSecret correctly

public class BaseApplication extends Application {

    private static final String TAG = BaseApplication.class.getSimpleName();
    
    //todo make sure to replace with your own app's appKey and appSecret
    private String APP_KEY = "Your APPKEY";
    private String APP_SECRET = "Your APPSECRET";
    private String SN = Build.SERIAL;

    @Override
    public void onCreate() {
        super.onCreate();
        initPaxStoreSdk();
    }

    private void initPaxStoreSdk() {
       //todo Init AppKey，AppSecret and SN, make sure the appKey and appSecret is corret.
       StoreSdk.getInstance().init(getApplicationContext(), appkey, appSecret, SN, new BaseApiService.Callback() {
                  @Override
                  public void initSuccess() {
                      //TODO Do your business here
                  }

                  @Override
                  public void initFailed(RemoteException e) {
                    //TODO Do failed logic here
                      Toast.makeText(getApplicationContext(), "Cannot get API URL from PAXSTORE, Please install PAXSTORE first.", Toast.LENGTH_LONG).show();
                  }
              });
    }
}
### Step 3：Download Parameters API
Download parameter (Optional, ignore this part if you don't have download parameter requirement)

Register your receiver (**Replace the category name with your own app package name**)
       
     <receiver android:name=".YourReceiver">
              <intent-filter>
                  <action android:name="com.paxmarket.ACTION_TO_DOWNLOAD_PARAMS" />
                  <category android:name="Your PackageName" />
              </intent-filter>
     </receiver>
     
Create your receiver. Since download will cost some time, we recommend you do it in your own service

      public class YourReceiver extends BroadcastReceiver {
          @Override
          public void onReceive(Context context, Intent intent) {
              //todo add log to see if the broadcast is received, if not, please check whether the bradcast config is correct
              Log.i("DownloadParamReceiver", "broadcast received");
              //since download may cost a long time, we recommend you do it in your own service
              context.startService(new Intent(context, YourService.class));
          }
      }
After you get broadcast, download params in your service

    public int onStartCommand(Intent intent, int flags, int startId) {
            //Specifies the download path for the parameter file, you can replace the path to your app's internal storage for security.
            final String saveFilePath = getFilesDir() + "YourPath";
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    //todo Call this method to download into your specific directory, you can add some log here to monitor
                    DownloadResultObject downloadResult = null;
                    try {
                        downloadResult = StoreSdk.getInstance().paramApi().downloadParamToPath(getApplication().getPackageName(), BuildConfig.VERSION_CODE, saveFilePath);
                    } catch (NotInitException e) {
                        Log.e(TAG, "e:" + e);
                    }

                    //businesscode==0, means download successful, if not equal to 0, please check the return message when need.
                    if(downloadResult != null && downloadResult.getBusinessCode()==0){
                        Log.i("downloadParamToPath: ", "download successful");
                        //file download to saveFilePath above.
                        //todo can start to add your logic.
                    }else{
                        //todo check the Error Code and Error Message for fail reason
                        Log.e("downloadParamToPath: ", "ErrorCode: "+downloadResult.getBusinessCode()+"ErrorMessage: "+downloadResult.getMessage());
                    }
                }
            });
            thread.start();
            return super.onStartCommand(intent, flags, startId);
        }
### Step 4: Update Inquirer (Optional)
Update inquirer: Your app will be asked whether it can be updated when there is a new version afther you
integrated this function.

If you do not have this requirement, just skip this step.

Integrate with this function only need to call initInquirer() after you init StoreSdk success.

    public class BaseApplication extends Application {

        private static final String TAG = BaseApplication.class.getSimpleName();
        
        //todo make sure to replace with your own app's appKey and appSecret
        private String APP_KEY = "Your APPKEY";
        private String APP_SECRET = "Your APPSECRET";
        private String SN = Build.SERIAL;
        @Override
        public void onCreate() {
            super.onCreate();
            initPaxStoreSdk();  //Initializing AppKey，AppSecret and SN
        }

         private void initPaxStoreSdk() {
                //todo Init AppKey，AppSecret and SN, make sure the appKey and appSecret is corret.
                StoreSdk.getInstance().init(getApplicationContext(), appkey, appSecret, SN, new BaseApiService.Callback() {
                           @Override
                           public void initSuccess() {
                               initInquirer();
                           }

                           @Override
                           public void initFailed(RemoteException e) {
                               Toast.makeText(getApplicationContext(), "Cannot get API URL from PAXSTORE, Please install PAXSTORE first.", Toast.LENGTH_LONG).show();
                           }
                       });
            }


         private void initInquirer() {
                //todo Init checking of whether app can be updated
                PaxStoreSdk.initInquirer(new StoreSdk.Inquirer() {
                    @Override
                    public boolean isReadyUpdate() {
                        Log.i(TAG, "call business function....isReadyUpdate = " + !isTrading());
                        //todo call your business function here while is ready to update or not
                        return !isTrading();
                    }
                });
         }

        //This is a sample of your business logic method
        public boolean isTrading(){
            return true;
        }
    }

## Template
You can get the **parameter template file** for **demo project** from assets/param_template.xml.

## FAQ

#### 1. How to resolve dependencies conflict?

When dependencies conflict occur, the error message may like below:

    Program type already present: xxx.xxx.xxx

**Solution:**

You can use **exclude()** method to exclude the conflict dependencies by **group** or **module** or **both**.

e.g. To exclude 'com.google.code.gson:gson:2.8.5' in SDK, you can use below:

    implementation ('com.pax.market:paxstoresdk:x.xx.xx'){
        exclude group: 'com.google.code.gson', module: 'gson'
    }

#### 2. How to resolve attribute conflict?

When attribute conflict occur, the error message may like below:

    Manifest merger failed : Attribute application@allowBackup value=(false) from 
    AndroidManifest.xml...
    is also present at [com.pax.market:paxstore-3rd-app-android-sdk:x.xx.xx] 
    AndroidManifest.xml...
    Suggestion: add 'tools:replace="android:allowBackup"' to <application> element
    at AndroidManifest.xml:..

**Solution:**

Add **xmlns:tools="http\://<span></span>schemas.android.com/tools"** in your manifest header

       <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.yourpackage"
            xmlns:tools="http://schemas.android.com/tools">

Add **tools:replace = "the confilct attribute"** to your application tag:

        <application
            ...
            tools:replace="allowBackup"/>


More questions, please refer to [FAQ](https://github.com/PAXSTORE/paxstore-3rd-app-android-sdk/wiki/FAQ)

## License

See the [Apache 2.0 license](https://github.com/PAXSTORE/paxstore-3rd-app-android-sdk/blob/master/LICENSE) file for details.

    Copyright 2018 PAX Computer Technology(Shenzhen) CO., LTD ("PAX")

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at following link.

         http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.