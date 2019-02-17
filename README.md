# Bitbucket Pipelines For Android Developement (Under construction)
If you are looking for a Continuous Integration tool for your Android Project that hosted on Bitbucket, you should try Bitbucket Pipelines. It's free up to 5 users and gives 50 minutes build time per month.

Enabling Pipelines is easy and all you need is a single YAML file and small adjustments based on the requirements on your project. Let's dive in!

First of all you can find a sample YAML file on above repository. But I'll also paste the lines here and try to explain it in detail. 
```sh
image: mingc/android-build-box:latest

pipelines:
  default:
  - step:
      script:
      - chmod +x gradlew
      - ./gradlew clean
      - ./gradlew test --stacktrace
      - ./gradlew assembleRelease --stacktrace
      artifacts:
      - app/build/outputs/apk/debug/release/*.apk
```
Normally, all you have to do is the above lines. By doing this all of your unit tests will run and right after that the APK files will be created for you to download. Great, right?

But you want more. You want to create your artifacts and download them after merging your pull request to master branch, or better you want to directly upload artifacts to the Google Play Console or somewhere else. In order to do that you need more adjusments than above one. Let's add more complexity to our case. Let's say you have a `debug` and a `production` variants. And this app is being released on Google Play and you need to keep your `keystore` file and it's relavant credentials safe. A way of keeping your keystore files and credentials safe is to make a separate `keystore.properties` file and ignoring that file while pushing to the VCS. And configuring gradle file to handle signing and building the necessarry APK files. Ok, you've already done this and works perfectly on your local machine. But how could your Bitcbucket Pipeline know keystore files and credentials for your project if you don't upload them? Obviously can't.

But we can solve this problem on the runtime of our pipeline. Let's update our basic YAML file.
```sh
image: mingc/android-build-box:latest

pipelines:
  default:
  - step:
      size: 2x
      script:
      - printf 'prod_alias=%s\nprod_password=%s\nprod_keystore=%s\nprod_keystore_password=%s\ndebug_alias=%s\ndebug_password=%s\ndebug_keystore=%s\ndebug_keystore_password=%s' $prod_alias $prod_password $prodKeyStore $prod_keystore_password $debug_alias $debug_password $debugKeyStore $debug_keystore_password > keystore.properties
      - echo $DEBUG_KEYSTORE_BASE64 | base64 -d > $debugKeyStore
      - cd app
      - echo $PROD_KEYSTORE_BASE64 | base64 -d > $prodKeyStore
      - cd ..
      - chmod +x gradlew
      - ./gradlew clean
      - ./gradlew test --stacktrace
      - ./gradlew assembleRelease --stacktrace
      artifacts:
      - app/build/outputs/apk/debug/release/*.apk
      - app/build/outputs/apk/prod/release/*.apk
```
Ouch! Too many lines to explain. Let me explain.
```sh
image: mingc/android-build-box:latest
```
In order to create your work environment, you'll need a Docker image. This image contains required Android SDK files and all the magical setup. You can create your own Docker image but there are many great images out there and [mingc] is one them.
```sh
default:
  - step:
```
`default` is your default branch setup. You can tell your pipeline for which type of branch what type of process you would like to apply. For example you can use different steps for your master branch and development branch. And `step` part is where you define all the stage that will be applied to your pipeline.
```sh
size: 2x
script:
```
If your project is big. Lots of code, lots of test classes to be run, than you may need extra memory on your pipeline. By default the Bitbucket Pipeline memory is limited. So in order to go with full potential, you can use `size: 2x` option. This will help. And the `script` part is where all the magic happen.

# To be continued...

[mingc]: <https://github.com/mingchen/docker-android-build-box>
