## Native iOS App
**1. Download this project**</br>
**2. Get xCode project exported from Unity**</br>
Since iOS project exported from Unity contains large files it can't be uploaded to github. You can download 
xCode project exported from Unity [from here](http://3dconfiguration.com/out/iOSUnityDemo.zip), or get [Unity project from github](https://github.com/obywan/DemoUnityProject) 
and make export yourself.</br>
**3. Setup Xcode workspace**
<br>Xcode workspace allows to work on multiple projects simultaneously and combine their products
- open NativeiOSApp.xcodeproj from Xcode
- create workspace and save it at UaaLExample/both.xcworkspace. (File / New / Workspace)
- close NativeiOSApp.xcodeproj project all Next steps are done from just created Workspace project
- add NativeiOSApp.xcodeproj and generated Unity-iPhone.xcodeproj from step #2 to workspace on a same level ( File / Add Files to “both” )
  <br><img src="https://github.com/Unity-Technologies/uaal-example/blob/master/docs/images/ios/workspaceProjects.png">

**4. Add UnityFramework.framework**
<br>With this step we add Unity player in the form of a framework to NativeiOSApp, it does not change the behavior of NativeiOSApp yet
- select NativeiOSApp target from NativeiOSApp project
- in "General / Frameworks, Libraries, and Embedded  Content" press +
- select Unity-iPhone/UnityFramework.framework
  <br><img src="https://github.com/Unity-Technologies/uaal-example/blob/master/docs/images/ios/addToEmbeddedContent.png">
- in "Build Phases" tab, expand "Link Binary With Libraries"
- remove UnityFramework.framework from the list (select it and press - )
  <br><img src="https://github.com/Unity-Technologies/uaal-example/raw/master/docs/images/ios/removeLink.png">

**5. Expose NativeCallProxy.h**
<br>Native application implements NativeCallsProtocol defined in following file:
- find and select Unity-iPhone / Libraries / Plugins / iOS / NativeCallProxy.h
- enable UnityFramework in Target Membership and set Public header visibility (small dropdown on right side to UnityFramework)
  <br><img src="https://github.com/Unity-Technologies/uaal-example/raw/master/docs/images/ios/nativeCallProxyTarget.png">
  
 **6. Make Data folder to be part of the UnityFramework**
 <br>By default Data folder is part of Unity-iPhone target, we change that to make everything encapsulated in one single framework file.
 - change Target Membership for Data folder to UnityFramework
   <br><img src="https://github.com/Unity-Technologies/uaal-example/raw/master/docs/images/ios/dataTargetMembership.png" height='300px'>
 - (optional) If you want to use Unity-iPhone sheme you need to point UnityFramework to a new place where Data is located by calling from Unity-iPhone/MainApp/main.mm:
   ```
   [ufw setDataBundleId: "com.unity3d.framework"];
   // On Demand Resources are not supported in this case. To make them work instead of the calls above 
   // you need to copy Data folder to your native application (With script at Build Phases) and 
   // skip a calls above since by default Data folder expected to be in mainBundle.
   ```
   <br><img src="https://github.com/Unity-Technologies/uaal-example/raw/master/docs/images/ios/setDataBundleId.png">
  
## Workspace is ready
Everything is ready to build, run and debug for both projects: Unity-iPhone and NativeiOSApp (select NativeiOSApp scheme to run Native App with integrated Unity or Unity-iPhone to run just Unity App part)
If all went successfully at this point you should be able to run NativeiOSApp:


## Notes
**Loading**
Unity player is controlled with UnityFramework object. To get it you call UnityFrameworkLoad (it loads UnityFramework.framework if it wasn't, and returns singleton instance to UnityFramework class observe Unity-iPhone/UnityFramework/UnityFramework.h for its API ). 
Observe UnityFrameworkLoad in: NativeiOSApp/NativeiOSApp/MainViewController.mm or in Unity-iPhone/MainApp/main.mm
```
#include <UnityFramework/UnityFramework.h>

UnityFramework* UnityFrameworkLoad()
{
    NSString* bundlePath = nil;
    bundlePath = [[NSBundle mainBundle] bundlePath];
    bundlePath = [bundlePath stringByAppendingString: @"/Frameworks/UnityFramework.framework"];

    NSBundle* bundle = [NSBundle bundleWithPath: bundlePath];
    if ([bundle isLoaded] == false) [bundle load];

    UnityFramework* ufw = [bundle.principalClass getInstance];
    if (![ufw appController])
    {
        // Initialize Unity for a first time
        [ufw setExecuteHeader: &_mh_execute_header];       

        // Keep in sync with Data folder Target Membership setting
        [ufw setDataBundleId: "com.unity3d.framework"]; 
       
    }
    return ufw;
}
```
