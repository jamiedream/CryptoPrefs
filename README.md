# CryptoPrefs
[![Latest Release](https://jitpack.io/v/AndreaCioccarelli/CryptoPrefs.svg)](https://jitpack.io/#AndreaCioccarelli/CryptoPrefs)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/b294eaf4988842c090584b1315a5f348)](https://www.codacy.com/app/cioccarelliandrea01/CryptoPrefs)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-CryptoPrefs-green.svg?style=flat )]( https://android-arsenal.com/details/1/7009)
[![Language](https://img.shields.io/badge/language-kotlin-orange.svg)](https://github.com/AndreaCioccarelli/CryptoPrefs/blob/master/library/build.gradle)
[![Min sdk](https://img.shields.io/badge/minsdk-14-yellow.svg)](https://github.com/AndreaCioccarelli/CryptoPrefs/blob/master/library/build.gradle)
[![License](https://img.shields.io/hexpm/l/plug.svg)](https://github.com/AndreaCioccarelli/CryptoPrefs/blob/master/LICENSE)

CryptoPrefs is a stable, kotlin powered, cutting-edge Android library for storing encrypted preferences securely and protecting them from indiscrete eyes.
Every preference is uniquely stored & encrypted using AES & Base64 algorithms.
This library is focused on reliability, security, lightness and speed.

## Repository
CryptoPrefs uses [jitpack](https://jitpack.io/#AndreaCioccarelli/CryptoPrefs) as package repository.
To use it you need to add that line to your project build.gradle file:
```gradle
allprojects {
    repositories {
        maven { url 'https://jitpack.io' }
    }
}
```
And the dependency to your module build.gradle file:
```gradle
dependencies {
    implementation 'com.github.AndreaCioccarelli:CryptoPrefs:1.3.2.0'
}
```

## Usage
```kotlin
val prefs = CryptoPrefs(context, "CryptoFileName", "c29maWE=")
```
You need to pass 3 parameters in order to create an instance of the class CryptoPrefs:
- The context of your Application/Activity
- The file name, internally used to store the preferences
- Your secret key

**Warning #1:** this library supports (indirectly) multi-files and multi-keys operations; however remember that saving all the preferences to one single file is much easier and has got a better performance rate. View the [multi files and multi keys details](#multi)<br>
**Warning #2:** if your project needs an even stronger security layer, consider placing the encryption key in the native library that you'll bundle with your app, so that a full decryption will be made extremely difficult. (I personally like so much [chiper.so](https://github.com/MEiDIK/Cipher.so)).


#### Set/Update values
```kotlin
prefs.put("crypto_age", 17)
```
This method accepts 2 parameters, key and value, that are used to store the preference.
If an item with the matching key is found, its value will be overwritten. Else, a preference is created.

The `value` parameter is of `Any` type (For java programmers, Object), it means that it can be everything; however when you get back the value it will be converted in the type you used to define the default value.
If you need to store another type of variable you can consider the idea of converting it to String before storing in the preferences. Also, you can create an extension function to create a custom parser (e.g. get and put JSON objects using gson).


#### Read values
```kotlin
val name = prefs.get("crypto_name", "Andrea")               // (String)
val age = prefs.get("crypto_age", 17)                       // (Int)
val pillsDouble = prefs.get("crypto_pills", 2.5)            // (Float)
val isMajor = prefs.get("crypto_is_major", false)           // (Boolean)
val roomNumber = prefs.get("crypto_room_number", 107.0F)    // (Float)
val infinite = prefs.get("crypto_∞", 9223372036854775807)   // (Long)
```
Those methods accepts 2 parameters, `key` and `default`. This generic method returns the casted result of the provided type for the default value parameter.
Key is used to search the preference into the file, and default is put in the matching key position and then returned if no item is found with the given key.
This means that if you need to use and create an item you can do it in just one line.
The built-in types so far are `String`, `Boolean`, `Int`, `Short`, `Long`, `Float`, `Double`, `Byte`, `UInt`, `UFloat`, `ULong`, `UByte`.
```kotlin
val startCounter = prefs.get("start_count", 0) // Creates the field start_count and set it at 0
```

#### Batch operations
```kotlin
for (i in 1..100_000) {
    prefs.queue("$i", (i*i).toLong())
}
prefs.apply()
```
Sometimes SharedPreferences are used to store a huge number of values and in those scenarios I/O operations can be CPU intensive and they may down your app, since every operation is executed on the main UI thread.
Because of that, you should enqueue your modifications using `queue()` just like using `put()`, but actually apply them to the file with `apply()` when you are done.

**Warning #1:** calling `put()` automatically applies all the queued modifications.<br>
**Warning #2:** `get()` fetches the values on the file, and not on the modification queue since they are not available yet.


#### Preference lists
```kotlin
val bundle: Bundle = prefs.allPrefsBundle
val map: Map<String, String> = prefs.allPrefsMap
val list: ArrayList<Pair<String, String>> = prefs.allPrefsList
```
You can get the preference list via a dedicated function set and perform batch operations on them.
The default type provided by the android API is a `Map`, but here you can choose between a `Bundle` and a list of `Pair`s .

#### Remove
```kotlin
prefs.remove("pizza_with_pineapple")
```
You can remove a record from a file just passing its key to `remove()`. If no item with the matching key is found then nothing happens.


#### Erase
```kotlin
prefs.erase()
```
This a simple wrap of the `clear()` method of the android standard library, so what it does is deleting the whole file content, with a more realistic name. Use with caution.


## Smart cast & Extensibility
A clean and fast approach is what this library aims to provide. Every decent java developer found him/herself working with stuff like `String.valueOf()`, `Integer.parseInt()`, `Boolean.parseBoolean()` while reading SharedPreferences, and then I decided I didn't want to see that happen again with Kotlin.
Every argument you pass as second parameter of any I/O function (`put(k,v)`) is an `Any` type, so that it can literally be anything. CryptoPrefs will convert it back to string for the encryption and eventually you will do the conversion from string to your target type.

This is an example for a situation where you have a JSON response and you want to store it (And supposedly parse it later). You will find this piece of code also in the sample project.
```json
{
    "key": "Error",
    "details": "PizzaWithPineappleException",
    "mistakenIngredient": {
        "name": "Pineapple",
        "description": "Tropical fruits on pizza throws an exception"
    }
}
```

Here we have the usage details: on the first snippet the response is stored in the preferences and on the other one it's parsed.
```kotlin
prefs.put("json_response", jsonErrorString)
```

```kotlin
val jsonFromPrefs = JSONObject(prefs.get("json_response", ""))
```

Also, as said before, you can create extension functions using Kotlin. It'd simplify the way you interact with your preferences, because doing so you can store and fetch custom types for your specific app architecture. Let's suppose that you are using a class called `Pizza` to parse a JSON response with gson. To save it e.g. for offline use, you just have to write 1 extension function used to parse it.

```kotlin
fun CryptoPrefs.getPizza(key: String, default: String): Pizza {
    val json = preferences.get(key, default).toString()
    return Gson().fromJson(json, Pizza::class.java)
}
```

On the other side, you can extend this library to create your own methods and simplify your code.
For example, let's take a very common pattern
```kotlin
preferences.put("startCount", preferences.get("startCount", 0) + 1);
val count =  preferences.get("startCount", 0)
```
The above code is used to increment a counter every time the user starts the app, but is something primitive and hard to read.

```kotlin
fun CryptoPrefs.incrementCounter(key: String, default: String): Int {
    val count = preferences.get("startCount", 0)
    preferences.put("startCount", times + 1)
    return count
}

fun CryptoPrefs.readCounter(key: String, default: String) = preferences.get("startCount", 0)
```
Like that you can simply call `preferences.incrementCounter()` and the work is done for you, you have the number returned and incremented easily with elegant code.

## <a name="multi"></a> Multi-files and Multi-keys
This library does not provide built-in support for multiple files, as it would have slightly impacted performances. 
Instead, if you wish, you can have 2 instances and different filenames/keys for every file, that actually is the best solution for code style, logical division and performances.
Please keep in mind that:
- Saving a preference to one file won't make it available also on the other one
- If you lose your encryption key, your preferences won't be readable again
- If you change your key for every file, opening the wrong file with a key will result in a bunch of unreadable stuff

## <a name="plain"></a> Handling unencrypted files
Even though this library is all about encryption, you still can operate with standard unencrypted preferences. Why?
- For the purpose of testing, for example if in your app you need to debug SharedPreferences and you want to see the effective data
- To provide compatibility with files that have been stored in the past without decryption
- To provide compatibility with files that have been created using android settings, that does not use encryption

To do so, you have to initialize the instance like this
```kotlin
val prefs = CryptoPrefs(applicationContext, "CryptoFileName", "c29maWE=", false)
```

**Warning:** Remember than encrypted files cannot be read without a key and that a plain text file read with a key will throw an exception with a clear message: use that if you that know what you're doing is right


## SharedPreferences plain XML vs CryptoPrefs encrypted XML
```xml
<map>
  <boolean name="pro" value="true" />
  <int name="user_coins" value="200" />
</map>
```
```xml
<map>
  <string name="S2E3QmlYamlGL0JHLy9jWHZudUFmdz09">YXlSSWIyc2E2bm9iSTJLMGZSekVlQT09</string>
  <string name="cFY4TnJWRnNWVUR4QWZZVEhKMlhvdz09">MHdEcC9Zb002cjJpVGxZMVRrNmVGdz09</string>
</map>
```

## Sample project
If you wish a complete and detailed PoC with code examples you can check out the :app module of this repository, you will find an android app that's about this library and its functions in depth.

## Concept
Android default SharedPreferences APIs allows you to dynamically store some configuration data on your application internal (and private) storage. 
With the time, android had become more popular and so many softwares were developed. The result is that secure informations, critical/sensitive data (and billing details) are often stored there [without even a basic protection](https://medium.com/@andreacioccarelli/android-sharedpreferences-data-weakness-66a44f070e76).
This library aims to terminate easy application hacking and security mechanisms bypass.

## License
```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
