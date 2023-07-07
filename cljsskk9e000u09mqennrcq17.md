---
title: "Local Storage in React Native"
seoDescription: "Analysis about local storage usage in react native based on security and data complexity."
datePublished: Fri Jul 07 2023 16:29:11 GMT+0000 (Coordinated Universal Time)
cuid: cljsskk9e000u09mqennrcq17
slug: local-storage-in-react-native
tags: react-native, security, localstorage, sensitive-data

---

When it comes to local storage in React Native there are several criteria to consider. Choosing the right local storage solution is something that cannot be taken lightly and even more when developing offline-first apps. There are a lot of solutions to this topic, that's why I will cover the most common ones in this post.

The first question you need to ask yourself, and not only in the planning phase, is this one: **Is the data I will store sensitive?** This is a recurrent question developers need to consider every time some information will be stored and the main reason is that **you cannot save on the device sensitive information without some security protection.**

When storing sensitive data, we can never be 100% sure that data is secured, there is no more secured information than the one that is never stored. That's why the first thing we need to re-think is this: Is it really necessary to store this particularly sensitive information? if the answer is yes, then you should proceed with it.

This post will not analyze file system access. It's recommended to use this when you are synchronizing files from a server or writing large amounts of data. If you want a recommendation on what library to use, you can check [react-native-fs](https://github.com/itinance/react-native-fs). With it, we can cover the most common solutions.

On Android and iOS, every app runs in a sandbox that no other app has access to, this doesn't mean your data is secure. Nevertheless, the majority of data used in an app can be stored in a plain storage solution. Storing securely comes with a price to pay regarding performance due to encryption/decryption. Only use it for sensitive information to prevent a negative impact on your app.

Just to give some hints about what solution to use and put you on the right track, you can follow the following mini-guide that considers "secure storage" and "data complexity" categories:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688741801911/e12871bc-79ec-4d63-bede-dacc2a3e19e3.png align="center")

To decide between the ones in the same group, let's see the main features of each library so you have more chances on choosing correctly according to your needs. You can visit the links on each title for more info.

### [AsyncStorage](https://react-native-async-storage.github.io/async-storage/docs/usage)

To store non-sensitive data in the local storage react-native provided for a long time a built-in solution but since the RN team at Facebook tried to reduce the React Native core to the minimal (lean core), AsyncStorage was handed over to the community. This solution is very well maintained and supported.

With a very easy-to-use API, AsyncStorage is a simple key-value storage. It can only store primitive data forcing you to serialize/deserialize complex objects to JSON when storing/retrieving them. Also, you have to consider, AsyncStorage cannot be used to run complex queries, there is no database, and it does not support transactions or locking. This comes with extra care from your side when writing/reading from several parts of the app. Here you have small examples of its API:

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';

//Storing string value
const storeData = async (value) => {
  try {
    await AsyncStorage.setItem('my-key', value);
  } catch (e) {
    // saving error
  }
};

//Reading string value
const getData = async () => {
  try {
    const value = await AsyncStorage.getItem('my-key');
    if (value !== null) {
      // value previously stored
    }
  } catch (e) {
    // error reading value
  }
};
```

### [React Native M](https://github.com/mrousavy/react-native-mmkv)MKV

Another common solution is [MMKV](https://github.com/Tencent/MMKV), which is a native storage solution developed by WeChat. It has wrappers for reac native based on Javascript Interface (JSI) and therefore supports synchronous and fast access. MMKV is also a key/value store and because of it, complex objects must be serialized to JSON. You can see examples, more details and benchmarks here: [https://www.npmjs.com/package/react-native-mmkv](https://www.npmjs.com/package/react-native-mmkv). I will leave a small snippet here for a fast look:

```javascript
import { MMKV } from 'react-native-mmkv'

export const storage = new MMKV({
  id: `user-${userId}-storage`,
  path: `${USER_DIRECTORY}/storage`,
  encryptionKey: 'hunter2'
})

//and then you can use it
//Store
storage.set('user.name', 'Marc')
storage.set('user.age', 21)
storage.set('is-mmkv-fast-asf', true)

//Retrieve
const username = storage.getString('user.name') // 'Marc'
const age = storage.getNumber('user.age') // 21
const isMmkvFastAsf = storage.getBoolean('is-mmkv-fast-asf') // true
```

### [React Native MMKV Storage](https://github.com/ammarahm-ed/react-native-mmkv-storage)

About [**react-native-mmkv-storage**](https://github.com/ammarahm-ed/react-native-mmkv-storage) we can mention some more features like indexing and data life cycle methods which can be used for locking and transactions. It offers encryption for secure storage, something they call **full-encryption** and here you have the details about it in their [own words](https://github.com/ammarahm-ed/react-native-mmkv-storage#full-encryption-support): *"The library supports full encryption (AES CFB-128) on Android and iOS. You can choose to store your encryption key securely for continuous usage. The library uses Keychain on iOS and Android Keystore on android (API 23 and above).* Encrypting an instance is simple:*"*

```javascript
const storage = new MMKVLoader()
  .withEncryption() // Generates a random key and stores it securely in Keychain
  .initialize();
```

You can create many database instances. This helps greatly if you have separate logics/modules in the same app that use data differently, It also helps in better performance since each database instance is small instead of a single bulky database which makes things slower as it grows. Simple indexer and data querying: For each database instance, there is one global key index and then there are indexes of each type of data. So querying is easy and fast. Support for redux persist is also added.

### [React Native Keychain](https://github.com/oblador/react-native-keychain)

Well-maintained library with easy API. It supports Android, iOS and macOS Catalyst. React Native Keychain uses hardware-backed encryption provided by the operating system to protect stored data. This ensures that the data is encrypted at rest and can only be accessed by the app that originally stored it.

React Native Keychain handles key management internally, abstracting away the complexity of managing encryption keys for developers. It provides an API to securely store and retrieve data using simple key-value pairs.

React Native Keychain supports biometric authentication (such as fingerprint or face recognition) to provide an additional layer of security for accessing stored data.

A small snippet of react-native-keychain usage:

```javascript
import * as Keychain from 'react-native-keychain';

async () => {
  const username = 'zuck';
  const password = 'poniesRgr8';

  // Store the credentials
  await Keychain.setGenericPassword(username, password);

  try {
    // Retrieve the credentials
    const credentials = await Keychain.getGenericPassword();
    if (credentials) {
      console.log(
        'Credentials successfully loaded for user ' + credentials.username
      );
    } else {
      console.log('No credentials stored');
    }
  } catch (error) {
    console.log("Keychain couldn't be accessed!", error);
  }
  await Keychain.resetGenericPassword();
};
```

### [Expo Secure Store](https://docs.expo.dev/versions/latest/sdk/securestore/)

Uses iOS Keychain and Android SharedPreferences combined with Keystore System. Expo Secure Store uses encryption mechanisms provided by the operating system to protect stored data. This ensures that the data is encrypted at rest and can only be accessed by the app that originally stored it.

Expo Secure Store abstracts away the complexity of key management by handling it internally. Developers can simply use the provided API to securely store and retrieve data.

Expo Secure Store supports biometric authentication (where available) to add an extra layer of security for accessing stored data.

```javascript
import * as SecureStore from 'expo-secure-store';

// Storing data
const storeData = async () => {
  try {
    await SecureStore.setItemAsync('key', 'value');
    console.log('Data stored successfully!');
  } catch (error) {
    console.log('Error storing data:', error);
  }
};

// Retrieving data
const retrieveData = async () => {
  try {
    const data = await SecureStore.getItemAsync('key');
    if (data) {
      console.log('Retrieved data:', data);
    } else {
      console.log('No data found.');
    }
  } catch (error) {
    console.log('Error retrieving data:', error);
  }
};

// Deleting data
const deleteData = async () => {
  try {
    await SecureStore.deleteItemAsync('key');
    console.log('Data deleted successfully!');
  } catch (error) {
    console.log('Error deleting data:', error);
  }
};
```

### [React Native Sensitive Info](https://mcodex.dev/react-native-sensitive-info/docs/)

It supports Android, iOS and Windows. It also adds another layer of security which can protect the data even on rooted devices. We should consider this library is getting behind regarding maintenance.

React Native Sensitive Info uses encryption to protect stored data. It encrypts the data using a randomly generated encryption key and stores it securely within the device's storage.

It internally manages the encryption key and abstracts away the complexities of key management for developers. It provides an API to store and retrieve sensitive information securely.

It offers additional security features such as automatic clearing of stored data when the app is uninstalled or when the device is locked. This helps to further protect sensitive information.

```javascript
import * as ReactNativeSensitiveInfo from 'react-native-sensitive-info';

// Storing data
const storeData = async () => {
  try {
    await ReactNativeSensitiveInfo.setItem('key', 'value', {
      sharedPreferencesName: 'my_shared_prefs',
      keychainService: 'my_keychain_service',
    });
    console.log('Data stored successfully!');
  } catch (error) {
    console.log('Error storing data:', error);
  }
};

// Retrieving data
const retrieveData = async () => {
  try {
    const data = await ReactNativeSensitiveInfo.getItem('key', {
      sharedPreferencesName: 'my_shared_prefs',
      keychainService: 'my_keychain_service',
    });
    if (data) {
      console.log('Retrieved data:', data);
    } else {
      console.log('No data found.');
    }
  } catch (error) {
    console.log('Error retrieving data:', error);
  }
};

// Deleting data
const deleteData = async () => {
  try {
    await ReactNativeSensitiveInfo.deleteItem('key', {
      sharedPreferencesName: 'my_shared_prefs',
      keychainService: 'my_keychain_service',
    });
    console.log('Data deleted successfully!');
  } catch (error) {
    console.log('Error deleting data:', error);
  }
};
```

What about queries or relations? What about database engines? They are often used for offline-first solutions with more complex structures to store. Fortunately, we have several solutions for react native.

### [React Native Quick Sqlite](https://github.com/margelo/react-native-quick-sqlite)

SQLite uses SQL queries and tables to store your data. To insert data you will have to create a table with a column for each property, then, you will need to use a SQL statement.

React Native Quick SQLite provides a direct interface to the SQLite database engine, allowing you to perform efficient data storage and retrieval operations using SQL queries. It is a lightweight SQLite wrapper designed for React Native, offering efficient and fast database operations. It allows customization of database configurations, such as defining custom table schemas and indexes, and setting PRAGMA options for SQLite. React Native Quick SQLite supports transactions, allowing you to perform atomic operations on the database. It also provides batch operations for executing multiple queries in a single call, improving performance.

Let's have a quick look at [**react-native-quick-sqlite**](https://github.com/margelo/react-native-quick-sqlite):

```javascript
import { open } from 'react-native-quick-sqlite';

try {
  const db = open('myDb.sqlite');

  let { rows } = db.execute('SELECT somevalue FROM sometable');

  rows.forEach((row) => {
    console.log(row);
  });

  let { rowsAffected } = await db.executeAsync(
    'UPDATE sometable SET somecolumn = ? where somekey = ?',
    [0, 1]
  );

  console.log(`Update affected ${rowsAffected} rows`);
} catch (e) {
  console.error('Something went wrong executing SQL commands:', e.message);
}
```

### [WatermelonDB](https://watermelondb.dev/docs)

Now let's talk about a relatively new solution with great and powerful features, **Watermelon DB**. This one uses SQLite behind the scenes and it is great with some key features: Highly scalable, Lazy loaded, Uses JS API, **Reactive** and others.

Unlike using SQLite directly, Watermelon is **fully observable**. So whenever you change a record, all UI that depends on it will automatically re-render. You can find more details about the advantages here: [https://watermelondb.dev/docs](https://watermelondb.dev/docs). Small example:

```javascript
import { Database, Model, field } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';

// Define a model
class Post extends Model {
  static table = 'posts';
  static associations = {
    comments: { type: 'has_many', foreignKey: 'post_id' },
  };

  @field('title') title;
  @field('body') body;
  @field('is_published') isPublished;
}

// Initialize the database
const adapter = new SQLiteAdapter({
  dbName: 'MyAppDB',
  schema: [Post.schema], // Specify the model schemas
});

const database = new Database({
  adapter,
  modelClasses: [Post],
});

// Create a new post
const createPost = async () => {
  await database.action(async () => {
    const newPost = await database.collections.get('posts').create((post) => {
      post.title = 'My First Post';
      post.body = 'This is the content of my first post';
      post.isPublished = true;
    });

    console.log('New post created:', newPost);
  });
};

// Retrieve all posts
const getPosts = async () => {
  const posts = await database.collections.get('posts').query().fetch();

  console.log('All posts:', posts);
};
```

### [Realm](https://realm.io/)

In Realm, Data is directly exposed as objects and queryable by code, removing the need for ORM's riddled with performance & maintenance issues.

Realm supports relationships, generics, and vectorization. Realm is known for its high performance and efficiency. It provides fast data access and manipulation, leveraging a lightweight database engine.

Realm offers real-time data synchronization and updates. It allows you to listen to changes in data in real-time, enabling live updates to UI components.

Realm supports seamless offline data synchronization, allowing data to be stored locally and synced with a remote database when connectivity is available.

Realm offers built-in encryption options to secure local and synchronized data. It allows encryption at the database level and supports encrypting individual properties or entire objects.

```javascript
import Realm from 'realm';

// Define a realm schema
const TaskSchema = {
  name: 'Task',
  properties: {
    id: 'int',
    title: 'string',
    completed: 'bool',
  },
};

// Open a realm instance
const realm = new Realm({ schema: [TaskSchema] });

// Create a new task
realm.write(() => {
  realm.create('Task', { id: 1, title: 'Buy groceries', completed: false });
});

// Query tasks
const tasks = realm.objects('Task');

console.log('All tasks:', tasks);

// Update a task
realm.write(() => {
  const task = realm.objects('Task').filtered('id = 1')[0];
  task.completed = true;
});

// Delete a task
realm.write(() => {
  const task = realm.objects('Task').filtered('id = 1')[0];
  realm.delete(task);
});

// Close the realm instance when you're done
realm.close();
```

In conclusion, choosing the right local storage solution in React Native depends on various factors such as data sensitivity, performance requirements, query capabilities, community support and others. When it comes to secure storage options, libraries like React Native Keychain, Expo Secure Store, and React Native Sensitive Info provide encryption mechanisms and support for secure storage environments. On the other hand, libraries like React Native Quick SQLite, Watermelon DB, and Realm offer powerful query capabilities and advanced data manipulation features along with secure storage options in some cases. It's important to consider your specific requirements and trade-offs when selecting a local storage solution to ensure data security and optimal performance in your React Native application.

**Used sources (big thanks):**

* **Book**: Professional React Native from Alexander Benedikt Kutting
    
* Documentation provided by each package's authors.