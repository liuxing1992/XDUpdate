# XDUpdate
### Android 自动更新/在线参数

- JSON、APK、Map文件的URL需要支持外链，即可以被直接访问，可考虑放在Git仓库、OSS或自己的服务器上

![Alt text](https://raw.githubusercontent.com/xingda920813/XDUpdate/master/Screenshot_notification.png)

![Alt text](https://raw.githubusercontent.com/xingda920813/XDUpdate/master/Screenshot_dialog.png)

![Alt text](https://raw.githubusercontent.com/xingda920813/XDUpdate/master/Screenshot_downloading.png)

## 引入

build.gradle中添加

	compile 'com.xdandroid:xdupdate:+'
    compile 'com.squareup.okhttp3:okhttp:+'
    compile 'com.squareup.okio:okio:+'
    compile 'io.reactivex:rxandroid:+'
    compile 'io.reactivex:rxjava:+'
    
# 自动更新
## 1.准备描述更新信息的JSON文件
    {
    "versionCode":4,                          //新版本的versionCode,int型
    "versionName":"1.12",                     //新版本的versionName,String型
    "url":"http://contoso.com/app.apk",       //APK下载地址,String型
    "note":"Bug修复",                         //更新内容,String型
    "md5":"D23788B6A1F95C8B6F7E442D6CA7536C", //32位MD5值,String型
    "size":17962350                           //大小(字节),int型
    }

## 2.构建XdUpdateAgent对象
    XdUpdateAgent updateAgent = new XdUpdateAgent.Builder()
                .setDebugMode(false)                          //是否显示调试信息(默认:false)
                .setJsonUrl("http://contoso.com/update.json") //JSON文件的URL
                .setAllow4G(true)                             //是否允许使用运营商网络检查更新(默认:false)
                .setShowNotification(true)                    
                //使用通知提示用户有更新，用户点击通知后弹出提示框，而不是检测到更新直接弹框(默认:true，仅对非强制检查更新有效)
                .setIconResId(R.mipmap.ic_launcher)           //设置在通知栏显示的通知图标资源ID(必须指定，一般为应用图标)
                .setOnUpdateListener(new XdUpdateAgent.OnUpdateListener() {
						//取得更新信息JSON后的回调(可选)，回调在主线程，可执行UI操作，updateBean为JSON对应的数据结构  
                        public void onUpdate(boolean needUpdate, XdUpdateBean updateBean) {
                            if (!needUpdate) Toast.makeText(context,"您的应用为最新版本",Toast.LENGTH_SHORT).show();
                        }
                    })
                .setDownloadText("立即下载")                   //可选，默认为左侧所示的文本
                .setInstallText("立即安装(已下载)")
                .setLaterText("稍后再说")
                .setHintText("版本更新")
                .setDownloadingText("正在下载")
                .build();

## 3.检查更新
适用于App入口的自动检查更新。默认策略下，若用户选择“以后再说”或者划掉了通知栏的更新提示，则**当天**对**该版本**不再提示更新，防止用户当天每次打开应用时都提示，不胜其烦。  

    updateAgent.update(getActivity()); 
    
适用于应用“设置”页面的手动检查更新。此方法无视是否允许使用运营商网络和上面的默认策略，强制检查更新，有更新时直接弹出提示框。     

    updateAgent.forceUpdate(getActivity());   

弹出的更新对话框中只有“立即更新”按钮，没有“以后再说”，且不能取消对话框。用户体验不好，不推荐使用。     

    updateAgent.forceUpdateUncancelable(getActivity());   

#### 为防止内存泄漏，需调用updateAgent.onDestroy().

#### 可通过updateAgent.getDialog()得到更新提示框的AlertDialog.

# 在线参数
## 1.准备参数文件
建立JavaSE项目，先将键值对存放在Map中，然后将Map传入下面的writeObject方法，得到参数文件。

    public static void writeObject(Map<Serializable,Serializable> map) throws IOException {
        File file = new File("C:\\Users\\${user-account-name}\\Desktop\\map.obj");     //指定文件生成路径
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
        oos.writeObject(map);
        oos.close();
    }

## 2.得到在线参数
    XdOnlineConfig onlineConfig = new XdOnlineConfig.Builder()
                    .setDebugMode(false)                         //是否显示调试信息(默认:false)
                    .setMapUrl("http://contoso.com/map.obj")     //参数文件的URL
                    .setOnConfigAcquiredListener(new XdOnlineConfig.OnConfigAcquiredListener() {
						//主线程回调，可执行UI操作
                        public void onConfigAcquired(Map<Serializable, Serializable> map) {     
                            System.out.println(map);             //成功，传入Map
                        }

                        public void onFailure(Throwable e) {
                            e.printStackTrace();                 //失败，传入Throwable
                        }                           
                    }).build();
    onlineConfig.getOnlineConfig();

#### 为防止内存泄漏，需调用onlineConfig.onDestroy().
