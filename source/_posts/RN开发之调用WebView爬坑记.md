---
title: RN开发之调用WebView爬坑记
date: 2018-06-24 18:56:28
categories: React Native
tags: 爬坑
---
## RN开发之调用WebView爬坑记

#### 解决React Native之Android手机中WebView不能调用相机相册的问题

记录最近在RN开发中踩的一个坑以及分享我的爬坑记：Android手机不能在WebView中调用相机相册；我刚开始很困惑，大家都一致认为这是链接的第三方写的H5中调用相机相册的方法有问题所导致的，后来我通过查找原因，发现这是Android原生WebView中没有实现调用相机相册的功能；soga，知道原因后开始在网上查找解决方法，大概找到两三个类似的依赖包，于是开始尝试集成到现在的项目中去，结果全部jj：不是和其他依赖包冲突就是gradle build失败；唉，各种心塞心累，于是决定自己尝试桥接原生WebView，桥接的过程也是各种蜿蜒曲折啊；下面开始分享我桥接的过程，希望可以帮助有需要的人少踩点坑。

<!--more-->

##### 1. 为什么RN的android端webview不支持上传图片和调用相机？
android原生的webview，本身就需要配置一个方法来配合上传图片，所以RN封装的webView没有配置这个方法

```
// For Android 4.1
        public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
            if (mUploadMessage != null) {
                mUploadMessage.onReceiveValue(null);
            }
            mUploadMessage = uploadMsg;
            showPopSelectPic();
        }

        public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
            if (mUploadMessage != null) {
                mUploadMessage.onReceiveValue(null);
            }
            mUploadMessage = filePathCallback;
            showPopSelectPic();
            return true;
        }
```


##### 2. 如何给RN的webview配置上这个方法？
在js代码中暂时没办法处理，那只好改原生的方法，原生的webview封装在react native包里，没办法改，只好重新封装一个webview.


##### 3. 怎么重新封装一个RN组件？
[可参考封装RN组件的教程](https://reactnative.cn/docs/0.45/native-component-android.html#content)

1. 首先把`/node_modules/react-native/ReactAndroid/src/main/java/com/facebook/react/views/webview`中的ReactWebViewManager的复制到自己的src的java包目录里
2. 然后根据自己的需求进行更改，比如现在的需求事添加选择图片的配置

(1). 新建一个ActivityResultInterface 接口回调

```
interface ActivityResultInterface {
    void callback(int requestCode, int resultCode, Intent data);
}
```

(2). PickerActivityEventListener onActivityResult回调和自定义回调链接

```
public class PickerActivityEventListener extends BaseActivityEventListener {

    private ActivityResultInterface mCallback;

    public PickerActivityEventListener(ReactApplicationContext reactContext, ActivityResultInterface callback) {
        reactContext.addActivityEventListener(this);
        mCallback = callback;
    }

    // < RN 0.33.0
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        mCallback.callback(requestCode, resultCode, data);
    }

    // >= RN 0.33.0
    public void onActivityResult(Activity activity, int requestCode, int resultCode, Intent data) {
        mCallback.callback(requestCode, resultCode, data);
    }
}
```

(3). ReactWebViewManager 自定义添加配置

```
public class ReactWebViewManager extends SimpleViewManager<WebView> implements ActivityResultInterface {
    //一些初始化变量
    private ReactApplicationContext reactApplicationContext;
    private ValueCallback mUploadMessage;
    private Uri imageUri;
    private static final int TAKE_PHOTO = 10001;
    private static final int CHOOSE_PHOTO = 10002;
    protected static final String REACT_CLASS = "RCTWebView2";
    ......
    //修改构造方法
    public ReactWebViewManager(ReactApplicationContext reactApplicationContext) {
        this.reactApplicationContext = reactApplicationContext;
        new PickerActivityEventListener(reactApplicationContext, this);
        mWebViewConfig = new WebViewConfig() {
            public void configWebView(WebView webView) {
            }
        };
    }
    ......
     protected WebView createViewInstance(final ThemedReactContext reactContext) {
        ReactWebView webView = createReactWebViewInstance(reactContext);
        webView.setWebChromeClient(new WebChromeClient() {
          ......
            // For Android 4.1
            public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
                if (mUploadMessage != null) {
                    mUploadMessage.onReceiveValue(null);
                }
                mUploadMessage = uploadMsg;
                showPopSelectPic(reactContext);
            }

            public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
                if (mUploadMessage != null) {
                    mUploadMessage.onReceiveValue(null);
                }
                mUploadMessage = filePathCallback;
                showPopSelectPic(reactContext);
                return true;
            }

        });
        ......
    }

    private void showPopSelectPic(final ThemedReactContext Context) {
        String[] items = new String[]{"相机", "相册"};
        AlertDialog.Builder builder = new AlertDialog.Builder(Context);
        builder.setTitle("提示")
                .setSingleChoiceItems(items, 0, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                        if (which == 0) {
                            //openCamera
                            File outputImage = new File(Context.getExternalCacheDir(), "output_image.jpg");
                            try {
                                if (outputImage.exists()) {
                                    outputImage.delete();
                                }
                                outputImage.createNewFile();
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                            if (Build.VERSION.SDK_INT < 24) {
                                imageUri = Uri.fromFile(outputImage);
                            } else {
                                imageUri = FileProvider.getUriForFile(Context, Context.getPackageName() + ".provider", outputImage);
                            }
                            // 启动相机程序
                            if (ContextCompat.checkSelfPermission(Context, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
                                //ActivityCompat.requestPermissions(Context, new String[]{Manifest.permission.CAMERA}, 2);
                            } else {
                                openCamera();
                            }
                        } else if (which == 1) {
                            //openAlbum
                            if (ContextCompat.checkSelfPermission(Context, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                                //ActivityCompat.requestPermissions(Context, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
                            } else {
                                openAlbum();
                            }
                        }
                    }
                })
                .setOnCancelListener(new DialogInterface.OnCancelListener() {
                    @Override
                    public void onCancel(DialogInterface dialog) {
                        if (mUploadMessage != null) {
                            mUploadMessage.onReceiveValue(null);
                            mUploadMessage = null;
                        }
                    }
                });
        builder.create().show();
    }

    void openCamera() {
        Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
        intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
        Activity currentActivity = reactApplicationContext.getCurrentActivity();
        currentActivity.startActivityForResult(intent, TAKE_PHOTO);
    }

    void openAlbum() {
        Intent intent = new Intent("android.intent.action.GET_CONTENT");
        intent.setType("image/*");
        Activity currentActivity = reactApplicationContext.getCurrentActivity();
        currentActivity.startActivityForResult(intent, CHOOSE_PHOTO); // 打开相册
    }

    @Override
    public void callback(int requestCode, int resultCode, Intent data) {
        switch (requestCode) {
            case TAKE_PHOTO:
                if (resultCode == RESULT_OK) {
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                        mUploadMessage.onReceiveValue(new Uri[]{imageUri});
                    } else {
                        mUploadMessage.onReceiveValue(imageUri);
                    }
                    mUploadMessage = null;
                } else {
                    mUploadMessage.onReceiveValue(null);
                    mUploadMessage = null;
                    return;
                }
                break;
            case CHOOSE_PHOTO:
                if (resultCode == RESULT_OK) {
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                        mUploadMessage.onReceiveValue(new Uri[]{data.getData()});
                    } else {
                        mUploadMessage.onReceiveValue(data.getData());
                    }
                    mUploadMessage = null;
                } else {
                    mUploadMessage.onReceiveValue(null);
                    mUploadMessage = null;
                    return;
                }
                break;
            default:
                break;
        }
    }
}
```

(4). 注意 需要注册FileProvider和设置xml path
(注意此处可能会和其他上传图片的依赖包中的清单文件冲突，原因是配置的authorities冲突，只要修改一致即可)

```
//清单文件中注册
<application
    ...>

   <provider
        android:name="android.support.v4.content.FileProvider"
        android:authorities="com.company.app.provider" //最好是包名+'.provider', 如果你的工程里集成有图片上传的依赖包，那么编译可能会有冲突，解决：你将此处修改成与冲突的依赖包一致即可，注意桥接方法里的图片路径(FileProvider)也要同步修改
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/provider_paths" //注意和冲突的依赖包进行比对修改
        />
    </provider>
</application
```

(5). res资源文件中新建xml文件夹新建文件provider_paths.xml

```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
       <external-path name="app_images" path="." />
</paths>
```


##### 4. 新建WebViewReactPackage.java文件，将写好的ReactWebViewManager写入到WebViewReactPackage中

```
public class WebViewReactPackage implements ReactPackage {

    public List<Class<? extends JavaScriptModule>> createJSModules() {
        return Collections.emptyList();
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactApplicationContext) {
        return Arrays.<ViewManager>asList(
                new ReactWebViewManager(reactApplicationContext)
        );
    }

    @Override
    public List<NativeModule> createNativeModules(
            ReactApplicationContext reactApplicationContext) {
        return Collections.emptyList();
    }
}
```


##### 5. 将WebViewReactPackage写入MainApplication中

```
public class MainApplication extends Application implements ReactApplication {
......
  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
  ......
    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
            new WebViewReactPackage()
      );
    }
 ......
  };
......
}
```


##### 6. 在项目新建BridgeWebView.js文件，把`/node_modules/react-native/Libraries/Components/WebView`中的webview.android.js,复制到自己的js文件夹中，做一定的修改

```
// 修改部分
var RCT_WEBVIEW_REF = 'webview';
......
class WebViewBridge extends React.Component {
    ......
    var RCTWebView = requireNativeComponent('RCTWebViewOpenImg', WebViewBridge, WebViewBridge.extraNativeComponentConfig); //RCTWebViewOpenImg与ReactWebViewManager的REACT_CLASS对应
module.exports = WebViewBridge;
```


##### 7. 最后你在需要用的js文件，引入BridgeWebView.js文件，通过Platform判断Android平台调用桥接的WebViewBridge，iOS平台调用RN封装的WebView.