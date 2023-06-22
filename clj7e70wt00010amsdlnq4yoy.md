---
title: "Creating an Augmented Reality files viewer using Native Modules - part 1:  Android"
seoTitle: "Augmented Reality in React Native with Native Modules - Android"
seoDescription: "This React Native demo shows AR assets in Augmented Reality using Scene Viewer without any third-party library."
datePublished: Thu Jun 22 2023 17:03:36 GMT+0000 (Coordinated Universal Time)
cuid: clj7e70wt00010amsdlnq4yoy
slug: creating-an-augmented-reality-files-viewer-using-native-modules-part-1-android
tags: react-native, android, augmented-reality, sceneviewer, nativemodules

---

In this article, We will create a small React Native (RN) demo that has a list of AR file items. When an item is pressed, it will be displayed in the native platform's Augmented Reality. The main purpose of this demo is to have a better understanding of how we can create native modules and send data from Javascript to them. We will be able to communicate the Javascript side with the native platform API and we won't need any third-party library.

The entire demo will be divided into two articles, one to cover the Android platform and the other for iOS. The final code will be available with free access on GitHub.

Corrections, suggestions, ideas or comments are more than welcome. You are free to express yourself ;)

What's the plan? In a newly created RN project, I will **prepare the user interface and the event handlers** of the demo, and then I will **communicate it with the native API** providing the file location of the pressed item, finally the **native API will trigger the AR viewer** using the received file location. Let's get started. I will skip the boilerplate of creating, setup and cleaning a new RN project. I will start preparing all the javascript side parts.

### **User interface (javascript side)**

This small demo uses a very basic structure that fulfils all its needs. The key parts of the structure look as follows:

* `App.tsx`: It will contain the only screen of the demo with the FlatList of the AR items.
    
* `src`
    
    * `@types`
        
        * `index.ts`: it will hold the project types
            
    * `components`
        
        * `ARItem.tsx`: a simple and reusable component that will show an item in the AR files list
            
    * `data`
        
        * `index.ts`: AR files data. In a real scenario, this data could come from different sources, in this case, it will be hardcoded.
            

Let's start by creating a folder named `src` with a subfolder `@types` with an `index.ts` file inside that will contain the types to use. We will only need one type now: `ItemType` with the model representation of the AR files:

```typescript
export type ItemType = {
  name: string;
  description: string;
  aRUrl: string;
  imageUrl: string;
};
```

For this demo, we are going to use a simple and reusable component that will show an item in the AR files list. Create a file named `ARItem.tsx` under the `components` folder. The content of this file looks like this:

```typescript
import React from 'react';
import {View, StyleSheet, Text, Image, TouchableOpacity} from 'react-native';
import {ItemType} from '../@types';

type ARItemProps = {
  item: ItemType;
  onPress: () => void;
};

function ARItem(props: ARItemProps): JSX.Element {
  const {item, onPress} = props;
  const {name, description, imageUrl} = item;
  return (
    <TouchableOpacity style={styles.container} onPress={onPress}>
      <Image source={{uri: imageUrl}} style={styles.image} />
      <View style={styles.textContainer}>
        <Text style={styles.title}>{name}</Text>
        <Text style={styles.description}>{description}</Text>
      </View>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 10,
    flexDirection: 'row',
    alignItems: 'center',
  },
  image: {
    width: 80,
    height: 80,
  },
  textContainer: {
    flex: 1,
    marginLeft: 5,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  description: {
    fontSize: 15,
  },
});

export default ARItem;
```

As you can see this reusable component is expecting to receive as props the AR item and the onPress function to call when the TouchableOpacity is pressed. It also contains some styles needed to improve just a little the user interface.

In the `data` folder let's create an `index.ts` file that will contains the data related to the AR resources we are going to use. The content of the file looks like follows:

```typescript
import {ItemType} from '../@types';

const data: ItemType[] = [
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

export default data;
```

Now we can use `ARItem` in the AR files list inside the App component:

```typescript
import React from 'react';
import {SafeAreaView, FlatList, ListRenderItem} from 'react-native';
import ARItem from './src/components/ARItem';
import data from './src/data';
import {ItemType} from './src/@types';

function App(): JSX.Element {

  const onTouchAssetHandler = (item: ItemType) => {
    console.log(item);
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

It's a very simple screen: just a `FlatList` which uses the ARItem component to render the items data array. When a list item is pressed the onTouchAssetHandler function is called with the item object passed as a parameter. For now, we are just logging the item, we will return to this function later in the article.

After running the RN project on Android device or emulator, it should look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687450095012/f0330a40-0d8e-4c14-9383-8b86a276d219.png align="center")

Now let's prepare the [Android native API with the native module](https://reactnative.dev/docs/native-modules-android) that will display the pressed file in Augmented Reality. For that, we need to open the `android` folder of our project in Android Studio. This folder already contains a standard native Android project.

### Native Module (Android side)

Now we are going to create the module that will receive the file's URL and will open the Augmented Reality viewer. In this demo, we are going to use [Scene Viewer](https://developers.google.com/ar/develop/scene-viewer#java) for this purpose.

Create a java class under your android package (eg. `java/com.arviewerdemo`). Let’s call it `ARViewerModule`.

In order to use this class from our JS code we need to import several packages at the top of the file:

```java
import android.content.Context;
import android.content.Intent;
import android.net.Uri;

import androidx.annotation.NonNull;

import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
```

We need to extend our Java class from `ReactContextBaseJavaModule` and implement the `getName` method that will return the name of our module. This name will be used to load the module in JS side. Also, we need to create its constructor as follows:

```java
public class ARViewerModule extends ReactContextBaseJavaModule {
    private Context context;

    ARViewerModule(@NonNull ReactApplicationContext context) {
        super(context);
        this.context = context;
    }

    @NonNull
    @Override
    public String getName() {
        return "ARViewerModule";
    }
}
```

Pay attention how this class is extending from `ReactContextBaseJavaModule`, it will allow to expose this class as a Native Module to JS side. We also need to receive the `ReactApplicationContext` instance in the constructor and pass it to the father class for initialization, we will use it to open an activity that's why we keep it in our class. Now let's implement a method that will receive the pressed item's URL and show it in AR:

```java
@ReactMethod
    public void displayInAR(String url) {
        Intent sceneViewerIntent = new Intent(Intent.ACTION_VIEW);
        Uri intentUri = Uri.parse("https://arvr.google.com/scene-viewer/1.1")
                .buildUpon().appendQueryParameter("file", url)
                .appendQueryParameter("mode", "ar_only")
                .build();
        sceneViewerIntent.setData(intentUri);
        sceneViewerIntent.setPackage("com.google.ar.core");
        sceneViewerIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(sceneViewerIntent);
    }
```

When we want to expose a function to JS we need to use the `@ReactMethod` decorator. In this method, we will start the activity using the scene viewer intent. If you want to go deeper you can check the [scene viewer documentation](https://developers.google.com/ar/develop/scene-viewer#java).

When a native module method is invoked in JavaScript, React Native converts the arguments from JS types to their Java/Kotlin analogues. Here there is a table that helps to understand [how this conversion works](https://reactnative.dev/docs/native-modules-android#argument-types):

![](https://lh5.googleusercontent.com/vuRs09aWBDfOAG2rzLxyGakbfbkODNtRhrrMv3p8XY0qPAe3tWQnk8sWDa28_rVU_HxRwvsO1BIrq-An32n5XsoNgbedBjKrpKQzyy1H8Gk3VAToi0paLi4rNDUqWXoqicDMOYuITXVFeyIh8fs97iA align="left")

Now let’s register our ARViewerModule class. For this, we will need to create a class that will extend from ReactPackage class. This class will register our modules and let React Native knows about them. Let’s create a Java class inside our package and name it `AppPackage`:

```java
import androidx.annotation.NonNull;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class AppPackage implements ReactPackage {
    @NonNull
    @Override
    public List<ViewManager> createViewManagers(@NonNull ReactApplicationContext reactApplicationContext) {
        return Collections.emptyList();
    }

    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();

        modules.add(new ARViewerModule(reactContext));

        return modules;
    }
}
```

This class implements `ReactPackage` which provides all we need to register modules in react context. We need to implement two methods `createViewManagers` and `createNativeModules`, which is the one we will pay more attention to. For this demo, `createViewManagers` method will return an empty list but `createNativeModules` will return a list that includes an instance of our `ARViewerModule`.

To finish our module registration we still need to do a final step. Let's open the existing class MainApplication and add an instance of our AppPackage to the list returned by `getPackages` method. Check the result code:

```java
@Override
protected List<ReactPackage> getPackages() {
  List<ReactPackage> packages = new PackageList(this).getPackages();
  packages.add(new AppPackage());
  return packages;
}
```

At this point, our module is registered and ready to use.

### Using Android methods from RN components (Javascript side)

It's time to [use our method from JS](https://reactnative.dev/docs/native-modules-android#test-what-you-have-built). We will implement this in our `onTouchAssetHandler` function but first we need to import `NativeModules` from react-native:

`import {SafeAreaView, FlatList, ListRenderItem, NativeModules} from 'react-native';`

Now, using destructuring, we can get our ARViewerModule. It will give us the module if already loaded, or load it if not:

`const { ARViewerModule } = NativeModules;`

Here is important to use the name returned in the getName method of our module.

Finally, we can invoke our Java method from `onTouchAssetHandler` :

`ARViewerModule.displayInAR(item.aRUrl);`

This is how the final version of `App.tsx` looks like:

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

Time to test our app, because we did changes in the Android code, we will probably need to re-run the app again. If everything goes well you should be able to see the AR asset in Scene Viewer after pressing it:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687451081409/07ed9b61-8738-460c-9121-0bf61ce0aabd.png align="center")

You can access the full demo repository [here](https://github.com/osielmesa/ARViewerDemo). This article is covering the Android platform only but in the second part we will be extending it to iOS platform as well. As I said, corrections, suggestions, comments or any kind of feedback are more than welcome.

Happy coding!