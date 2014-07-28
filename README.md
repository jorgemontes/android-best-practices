# Best practices in Android develoment

Lessons learned from Android developers in Futurice. Avoid reinventing the wheel by following these guidelines.

Feedback and criticism are welcomed, feel free to open an issue or send a pull request.

## Summary

#### Use Gradle and its recommended project structure
#### Put passwords and sensitive data in gradle.properties
#### Don't write your own HTTP client, use a library
#### Use Gson unless you have a reason not to
#### Use Volley or Retrofit+Picasso for networking and images
#### Use Fragments to represent a UI "screen"
#### Use Activities just to manage Fragments and Action Bar
#### Keep your colors.xml short and DRY, just define the palette
#### Also keep dimens.xml DRY, define generic constants
#### Use styles to avoid duplicate attributes in layout XMLs
#### Use multiple style files to avoid a single huge one
#### Do not make a deep hierarchy of ViewGroups
#### Avoid using WebViews as much as you can


----------

### Android SDK

Place your [Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools) in the `/opt` directory or some other application-independent location. Some IDEs include the SDK when installed, and may place it under the same directory as the IDE. This can be bad when you need to upgrade (or reinstall) the IDE, or when changing IDEs. Putting it in `/opt` makes it independent of IDE.

### Build system

Your default option should be [Gradle](http://tools.android.com/tech-docs/new-build-system). Ant is much more limited and also more verbose. With Gradle, it's simple to: 

- Build different flavours or variants of your app
- Make simple script-like tasks
- Manage and download dependencies
- Customize keystores
- And more 

Android's Gradle plugin is also being actively developed by Google while as the new standard build system.

### Project structure

There are two popular options: the old Ant & Eclipse ADT project structure, and the new Gradle & Android Studio project structure. You can make the decision whether to use one or the other, although we recommend the new project structure. If using the old, try to provide also a Gradle configuration `build.gradle`.

Old structure:

    assets
    libs
    res
    src
        com/example/myapp
    AndroidManifest.xml
    build.gradle
    project.properties
    proguard-rules.pro

New structure:

    library-foobar
    app
        libs
        src
            androidTest
                java
                    com/example/myapp
            main
                java
                    com/example/myapp
                res
                AndroidManifest.xml
        build.gradle
        proguard-rules.pro
    build.gradle
    settings.gradle

The main difference is that the new structure explicitly separates 'source sets' (`main`, `androidTest`), a concept from Gradle. You could, for instance, add source sets 'paid' and 'free' into `src` which will have source code for the paid and free flavours of your app.

Having a top-level `app` is useful to distinguish your app from other library projects (e.g., `library-foobar`) that will be referenced in your app. The `settings.gradle` then keeps references to these library projects, which `app/build.gradle` can reference to.

### Gradle configuration

**General structure.** Follow [Google's guide on Gradle for Android](http://tools.android.com/tech-docs/new-build-system/user-guide)

**Small tasks.** Instead of (shell, Python, Perl, etc) scripts, you can make tasks in Gradle. Just follow [Gradle's documentation](http://www.gradle.org/docs/current/userguide/userguide_single.html#N10CBF) for more details.

**Passwords.** In your app's `build.gradle` you will need to define the `signingConfigs` for the release build. Here is what you should avoid:

_Don't do this_. This would appear in the version control system.

    signingConfigs {
        release {
            storeFile file("myapp.keystore")
            storePassword "password123"
            keyAlias "thekey"
            keyPassword "password789"
        }
    }

Instead, make a `gradle.properties` file which should _not_ be added to the version control system:

    KEYSTORE_PASSWORD=password123
    KEY_PASSWORD=password789

That file is automatically imported by gradle, so you can use it in `build.gradle` as such:

    signingConfigs {
        release {
            try {
                storeFile file("myapp.keystore")
                storePassword KEYSTORE_PASSWORD
                keyAlias "thekey"
                keyPassword KEY_PASSWORD
            }
            catch (ex) {
                throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
            }
        }
    }

### IDEs and text editors

Editors are a personal choice, and it's your responsibility to get your editor functioning according to the project structure and build system.

The most recommended IDE at the moment is [Android Studio](https://developer.android.com/sdk/installing/studio.html), because it is developed by Google, is closest to Gradle, uses the new project structure by default, is finally in beta stage, and is tailored for Android development.

You can use [Eclipse ADT](https://developer.android.com/sdk/installing/index.html?pkg=adt) if you wish, but you need to configure it, since it expects the old project structure and Ant for building. You can even use a plain text editor like Vim, Sublime Text, or Emacs. In that case, you will need to use Gradle and `adb` on the command line. If Eclipse's integration with Gradle is not working for you, your options are using the command line just to build, or migrating to Android Studio.

Whatever you use, just make sure Gradle and the new project structure remain as the official way of building the application, and avoid adding your editor-specific configuration files to the version control system. For instance, avoid adding an Ant `build.xml` file. Especially don't forget to keep `build.gradle` up-to-date and functioning if you are changing build configurations in Ant. Also, be kind to other developers, don't force them to change their tool of preference.

### Libraries

**Gson** is a Java library for converting Objects into JSON and vice-versa. It should be your default choice unless you have performance problems with it. Try to avoid reinventing the wheel, writing your own JSON serializer library.

**Networking, caching, and images.** There are a couple of battle-proven solutions for performing requests to backend servers. Use [Volley](https://android.googlesource.com/platform/frameworks/volley) or [Retrofit](http://square.github.io/retrofit/). Volley also provides helpers to load and cache images. If you choose Retrofit, consider [Picasso](http://square.github.io/picasso/) for loading and caching images. Both Picasso and Retrofit are made by the same company, so they complement each other nicely.

**RxJava** is a library for Reactive Programming, in other words, handling asynchronous events. It is a powerful and promising paradigm, which can also be confusing since it's so different. We recommend to take some caution before using this library to architect the entire application. There are some projects done by us using RxJava, if you need help talk to one of these people: Timo Tuominen, Olli Salonen, Andre Medeiros, Mark Voit, Antti Lammi, Vera Izrailit, (any one else?). We have written some blog posts on it: [[1]](http://blog.futurice.com/tech-pick-of-the-week-rx-for-net-and-rxjava-for-android), [[2]](http://blog.futurice.com/top-7-tips-for-rxjava-on-android), [[3]](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754), [[4]](http://blog.futurice.com/android-development-has-its-own-swift).

If you have no previous experience with Rx, start by applying it only for responses from the API. Alternatively, start by applying it for simple UI event handling, like click events or typing events on a search field. If you are confident in your Rx skills and want to apply it to the whole architecture, then write Javadocs on all the tricky parts. Keep in mind that another programmer unfamiliar to RxJava might have a very hard time maintaining the project. Do your best to help him understand your code and also Rx.

### Activities and Fragments

[Fragments](http://developer.android.com/guide/components/fragments.html) should be your default option for implementing a UI screen in Android. Fragments are reusable user interfaces that can be composed in your application. We recommend using fragments instead of [activities](http://developer.android.com/guide/components/activities.html) to represent a user interface screen, here are some reasons why:

- **Solution for multi-pane layouts.** Fragments were primarily introduced for extending phone applications to tablet screens, so that you can have both panes A and B on a tablet screen, while either A or B occupy an entire phone screen. If your application is implemented in fragments from the beginning, you will make it easier later to adapt your application to different form-factors.

- **Proper screen-to-screen communication.** Android's API does not provide a proper way of sending complex data (e.g., some Java Object) from one activity to another activity. With fragments, however, you can use the instance of an activity as a channel of communication between its child fragments.

- **Many Android classes expect fragments.** The [API for tabs on the ActionBar](http://developer.android.com/reference/android/app/ActionBar.html#newTab()) expects the tab contents to be fragments. Also, if using Google Maps API v2, the [recommended solution](https://developers.google.com/maps/documentation/android/) is [MapFragment](http://developer.android.com/reference/com/google/android/gms/maps/MapFragment.html), even though [MapView](http://developer.android.com/reference/com/google/android/gms/maps/MapView.html) exists.

- **Easy to implement swiping transitions between screens.** If a UI screen is a fragment in your application, you can use `FragmentPagerAdapter` to contain a collection of fragments and implement smooth and interactive transitioning between them. For instance, horizontal swiping of screens.

- **Intelligent behavior for "back".** The [FragmentManager](http://developer.android.com/reference/android/app/FragmentManager.html) allows you to perform transactions to change the fragments inside an activity. Hence, the FragmentManager manages the "state" of fragments. The back button/action will be handled by the FragmentManager to go back to the previous "fragments state". This is more advanced than the [activity stack](http://developer.android.com/guide/components/tasks-and-back-stack.html).

- **Fragments are generic enough to not be UI-only.** You can have a [fragment without a UI](http://developer.android.com/guide/components/fragments.html#AddingWithoutUI) that works as background workers for the activity. You can take that idea further to create a [fragment to contain the logic for changing fragments](http://stackoverflow.com/questions/12363790/how-many-activities-vs-fragments/12528434#12528434), instead of having that logic in the activity.

That being said, we advise not to use [nested fragments](https://developer.android.com/about/versions/android-4.2.html#NestedFragments) extensively, because [matryoshka bugs](http://delyan.me/android-s-matryoshka-problem/) can occur. Use nested fragments only when it makes sense (for instance, fragments in a horizontally-sliding ViewPager inside a screen-like fragment) or if it's a well-informed decision.

On an architectural level, your app should have a top-level activity that contains most of the business-related fragments. You can also have some other supporting activities, as long as their communication with the main activity is simple and can be limited to [`Intent.setData()`](http://developer.android.com/reference/android/content/Intent.html#setData(android.net.Uri)) or [`Intent.setAction()`](http://developer.android.com/reference/android/content/Intent.html#setAction(java.lang.String)) or similar.

### Java packages architecture

TODO

### Resources

TODO

### Thanks to

Antti Lammi, Joni Karppinen, Peter Tackage, Timo Tuominen, Vera Izrailit, Vihtori Mäntylä, and other Futurice developers for sharing their knowledge on Android development.