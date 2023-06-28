---
title: "Creating an Augmented Reality viewer using Native Modules - part  2: iOS (Swift)"
seoTitle: "Augmented Reality viewer using Native Modules: iOS (swift)"
seoDescription: "Small React Native (RN) demo that has a list of AR file items. Pressed item will be displayed in AR Quicklook using native code in Swift."
datePublished: Wed Jun 28 2023 08:03:11 GMT+0000 (Coordinated Universal Time)
cuid: cljffj65e00030aladnygdo02
slug: creating-an-augmented-reality-viewer-using-native-modules-part-2-ios-swift
tags: swift, react-native, augmented-reality, nativemodules

---

In this article, We create a small React Native (RN) demo that has a list of AR file items. When an item is pressed, it will be displayed in the native platform's Augmented Reality. The main purpose of this demo is to have a better understanding of how we can create native modules and send data from Javascript to them. We will be able to communicate the Javascript side with the native platform API and we won't need any third-party library.

The entire demo is divided into two articles, [the previous one covering the Android platform](https://osiel.hashnode.dev/creating-an-augmented-reality-viewer-using-native-modules-part-1-android) and this one for iOS. The final code will be available with free access on GitHub. I recommend starting by cloning the code base already created in the previous article, you can clone it [here](https://github.com/osielmesa/ARViewerDemo) in the [post-1-Android branch](https://github.com/osielmesa/ARViewerDemo/tree/post-1-Android).

Corrections, suggestions, ideas or comments are more than welcome. You are free to express yourself ;)

What's the plan? In the existing RN project we created in [the previous post](https://osiel.hashnode.dev/creating-an-augmented-reality-viewer-using-native-modules-part-1-android), we will **communicate Javascript with the native iOS API** providing the file location of the pressed item, finally, the **native API will trigger the AR viewer (**[**AR Quick Look**](https://developer.apple.com/documentation/arkit/arkit_in_ios/previewing_a_model_with_ar_quick_look)**)** using the received file location. Let's get started.

I will start preparing all the javascript side parts since we will need different data for the iOS platform. We need different data because the file type required for iOS is different from the type required for Android and we need to use the set of data depending on the platform.

### **User interface (javascript side)**

The project structure is already explained in the previous post. Let's remember how the UI looks like in the Android demo:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687450095012/f0330a40-0d8e-4c14-9383-8b86a276d219.png?auto=compress,format&format=webp align="center")

These files can be used on Android platform but not on iOS, so the first task will be to load the files info depending on the current platform:

Let's head to the `data` folder and modify the `index.ts` file:

```javascript
import {ItemType} from '../@types';
import {Platform} from 'react-native';

const dataAndroid: ItemType[] = [
  {
    name: 'Cube',
    description: 'Animated cube with marble texture.',
    imageUrl:
      'https://github.com/KhronosGroup/glTF-Sample-Models/raw/master/2.0/AnimatedCube/screenshot/screenshot.gif',
    aRUrl:
      'https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/AnimatedCube/glTF/AnimatedCube.gltf',
  },
  {
    name: 'Avocado',
    description: 'Cool 3d representation of an avocado.',
    imageUrl:
      'https://github.com/KhronosGroup/glTF-Sample-Models/raw/master/2.0/Avocado/screenshot/screenshot.jpg',
    aRUrl:
      'https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/Avocado/glTF/Avocado.gltf',
  },
  {
    name: 'Damage helmet',
    description: 'Futurist damaged helmet 3d model.',
    imageUrl:
      'https://github.com/KhronosGroup/glTF-Sample-Models/raw/master/2.0/DamagedHelmet/screenshot/screenshot.png',
    aRUrl:
      'https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/DamagedHelmet/glTF/DamagedHelmet.gltf',
  },
  {
    name: 'Lantern',
    description: 'Old street lantern.',
    imageUrl:
      'https://github.com/KhronosGroup/glTF-Sample-Models/raw/master/2.0/Lantern/screenshot/screenshot.jpg',
    aRUrl:
      'https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/Lantern/glTF/Lantern.gltf',
  },
  {
    name: 'Water bottle',
    description: 'Water bottle 3d model.',
    imageUrl:
      'https://github.com/KhronosGroup/glTF-Sample-Models/raw/master/2.0/WaterBottle/screenshot/screenshot.jpg',
    aRUrl:
      'https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/WaterBottle/glTF/WaterBottle.gltf',
  },
];

const dataIOS: ItemType[] = [
  {
    name: 'Biplane toy',
    description: 'Amazing airplane toy that can be place on the ground.',
    imageUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/biplane/biplane.jpg',
    aRUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/biplane/toy_biplane_idle.usdz',
  },
  {
    name: 'Sneaker',
    description: 'Sneaker air force style.',
    imageUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/nike-air-force/nike-air-force.png',
    aRUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/nike-air-force/sneaker_airforce.usdz',
  },
  {
    name: 'Car toy',
    description: 'Futurist car toy that can be place on the ground.',
    imageUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/toycar/toycar.jpg',
    aRUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/toycar/toy_car.usdz',
  },
  {
    name: 'Guitar',
    description: 'Red electric guitar.',
    imageUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/stratocaster/stratocaster.jpg',
    aRUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/stratocaster/fender_stratocaster.usdz',
  },
  {
    name: 'Robot',
    description: 'Great robot cartoon representation.',
    imageUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/vintagerobot2k/vintagerobot2k.jpg',
    aRUrl:
      'https://developer.apple.com/augmented-reality/quick-look/models/vintagerobot2k/robot_walk_idle.usdz',
  },
];

export default Platform.OS === 'android' ? dataAndroid : dataIOS;
```

As you can see we modified this file to export data based on `Platform.OS`. In a real-world scenario, this data could come from different sources (e.g. API rest). In iOS we need to use `.usdz` files for [AR Quick Look](https://developer.apple.com/documentation/arkit/arkit_in_ios/previewing_a_model_with_ar_quick_look). This is how the UI looks like in iOS now:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687857302778/23afb051-77f2-48b7-962b-13dee93febc6.png align="center")

Now that we have the UI ready with the corresponding data, let's jump to [iOS Native Modules API](https://reactnative.dev/docs/native-modules-ios) to create our module that will display the pressed file in Augmented Reality.

### **Native Module (iOS side)**

Now we are going to create the module that will receive the file's URL and will open the Augmented Reality viewer. In this demo, we are going to use [**AR Quick Look**](https://developer.apple.com/documentation/arkit/arkit_in_ios/previewing_a_model_with_ar_quick_look) for this purpose and `Swift` (with some `objective-C` boilerplate).

Inside the ios folder we can find the `[project name].xcworkspace` file, open it with Xcode. This workspace is where we can modify the iOS native project.

Let's create the module class. For this, we need a new swift file name `ARViewerModule`. For this demo, we will use the same name as our Android module but this is not a must, you can use the name you want, just be careful when you call your module from JS side later on, since you will need to use the module based on the platform. In this demo, we will use the same name so we don't need to make that difference.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687857930881/e759b4a3-ae77-42de-9e09-5d12cfcac614.png align="center")

After you click create, Xcode will ask you if you want to configure an Objective-C bridging header, and yes, **we need this:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687858041491/1d42fd38-fc82-4595-b2a9-8c92d9a31952.png align="center")

After clicking Create Bridging Header xcode will add 2 files to the iOS project: `ARViewerModule.swift` (our native module) and its `ARViewerDemo-Bridging-Header.h` (to import the target's public headers that we would like to expose to Swift.). We need to expose the bridge module from React in the bridging file:

```swift
//
//  Use this file to import your target's public headers that you would like to expose to Swift.
//
#import "React/RCTBridgeModule.h"
```

Now, we can use everything that `RCTBridgeModule` offers in our code.

Let's jump to the logic of our module in the `ARViewerModule.swift` file. In this class we are going to have our `displayInAR` method (same name as Android) and some extra code related to AR Quick Look (I added comments to make it easier to understand):

```swift
import Foundation
import QuickLook
import ARKit

@objc(ARViewerModule) // expose the class to objc code
class ARViewerModule: NSObject, QLPreviewControllerDataSource {
  
  private var previewItem: QLPreviewItem? // Item to be shown in AR
  
  func numberOfPreviewItems(in controller: QLPreviewController) -> Int {
    return 1 // Amount of QLPreviewItem to show
  }
  
  func previewController(_ controller: QLPreviewController, previewItemAt index: Int) -> QLPreviewItem {
    return previewItem! //QLPreviewController will show the returned QLPreviewItem here
  }
  
  @objc // expose the method to objc code
  func displayInAR(_ url: NSString) {
    let url = URL(string: url as String)!
    let task = URLSession.shared.downloadTask(with: url) { localURL, urlResponse, error in
      if let localURL = localURL {
        do {
          // We need to download the file first in order to display it in AR Quicklook
          let httpURLResponse = urlResponse as? HTTPURLResponse
          let fileName = httpURLResponse?.suggestedFilename ?? httpURLResponse?.url?.lastPathComponent ?? url.lastPathComponent
          let destination = FileManager.default.temporaryDirectory.appendingPathComponent(fileName)
          if FileManager.default.fileExists(atPath: destination.path) {
            try FileManager.default.removeItem(at: destination)
          }
          try FileManager.default.moveItem(at: localURL, to: destination)
          
          // We save the local destination file in previewItem, QLPreviewController will show it next
          self.previewItem = destination as NSURL
          DispatchQueue.main.async {
            let previewController = QLPreviewController()
            previewController.dataSource = self //QLPreviewController will use our module as data source
            var rootVC = UIApplication.shared.keyWindow?.rootViewController
            rootVC?.present(previewController, animated: true, completion: nil) //Presenting QLPreviewController
          }
        } catch {
          print(error)
        }
      
      }
    }
    
    task.resume()
  }
}
```

Notice how we **expose the class to Objective-C using** `@objc` **decorator and how we extend this class from** `NSObject` **and** `QLPreviewControllerDataSource`.  
In the `displayInAR` method we are receiving the `url` string parameter from JS side, **the method has to be exposed to Objective-C as well**.

To display the file in AR Quicklook we need to download it first and move it with the right filename and extension (.usdz) after. Then we save the local file location in the `previewItem` variable and invoke the `QLPreviewController`.

We are exposing the class and the methods to objective-c because then we are going to expose them to javascript. **We can’t expose Swift code directly to Javascript**, we need to use Objective-C for that, and that will be our next step.

Don't worry if you don't know Objective-C, we will only use it to expose our native goodies. Let's create a new Objective-C file and name it with the same name as our module class. Right-click `ARViewerDemo` (or the name you used) project and select `New File`, then select `Objective-C File` and click next. Use `ARViewerModule` (same name of the module) as the name of the file and click next:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687936053770/b8a7a8c1-1d03-4ad8-b93a-3db5e7675894.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687936172887/d4fc2d30-f591-4dba-be32-49ff33294eaf.png align="center")

This will create a `.m` file that will expose our module and its methods to Javascript, let's see how can we do that.

We need to use some macros from `React/RCTBridgeModule.h` so let’s import this in our newly created file and use the macros needed:

```objectivec
#import <Foundation/Foundation.h>
#import "React/RCTBridgeModule.h"

@interface RCT_EXTERN_MODULE(ARViewerModule, NSObject) //exposing the module to JS
RCT_EXTERN_METHOD(displayInAR: (NSString)url) //exposing the method to JS
@end
```

The macro `RCT_EXTERN_MODULE` is exposing our module class using its name as the first parameter and notice how we need to specify the `NSObject` inheritance as well. Inside the body of this macro, we need to expose the method as well, we use the `RCT_EXTERN_METHOD`, providing the method name and the received parameter type and name. Don't forget, the class and the method can be exposed here because we use the `@obc` decorator in our Swift class. When a native module method is invoked in JavaScript, React Native converts the arguments from JS types to their Objective-C/Swift object analogues. Here there is a table that helps to understand [how this conversion works](https://reactnative.dev/docs/native-modules-ios#argument-types):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687937498173/fc6d9110-2453-46b3-aa9d-87615051f784.png align="center")

We are ready to [use our Native Module in our React Native App component](https://reactnative.dev/docs/native-modules-ios#test-what-you-have-built).

### **Using Swift methods from RN components (Javascript side)**

Because we used the same name as the Android native module we don't need to make any changes to our existing code since the [previous post](https://osiel.hashnode.dev/creating-an-augmented-reality-viewer-using-native-modules-part-1-android). Let's remember what the code looks like in the `App.tsx` file:

```javascript
import React from 'react';
import {
  SafeAreaView,
  FlatList,
  ListRenderItem,
  NativeModules,
} from 'react-native';
import ARItem from './src/components/ARItem';
import data from './src/data';
import {ItemType} from './src/@types';

const {ARViewerModule} = NativeModules;

function App(): JSX.Element {
  const onTouchAssetHandler = (item: ItemType) => {
    ARViewerModule.displayInAR(item.aRUrl);
  };

  const renderItem: ListRenderItem<ItemType> = ({item}) => {
    return <ARItem item={item} onPress={() => onTouchAssetHandler(item)} />;
  };

  return (
    <SafeAreaView>
      <FlatList data={data} renderItem={renderItem} />
    </SafeAreaView>
  );
}

export default App;
```

We are destructuring the `ARViewerModule` from `NativeModules` and using it in `onTouchAssetHandler` to call `displayInAR` native method. In this call, we pass the `url` string as an argument.

In the case you decide to use different names for each platform's native module you will need to call them based on the `Platform.OS`

Don't forget to re-run the project before trying since we changed the native code. Hot reloading won't be enough. If everything goes well we should see the touched item in AR:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687938564166/c603bff0-df2f-41ab-b1cd-0d61d753358d.jpeg align="center")

You can access the full demo repository [**here**](https://github.com/osielmesa/ARViewerDemo). As I said, corrections, suggestions, comments or any kind of feedback are more than welcome.

Happy coding!