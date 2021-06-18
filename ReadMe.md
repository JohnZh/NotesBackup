[TOC]

# Japanese Macdonald's App

This is Japanese Macdonald's App Project, or JMA for short.

This is Android version. IOS version: https://github.com/theplant/mcd-ios



# Members and roles

- Project Manager: james@theplant.jp
- Product Manager: nate@theplant.jp
- Tech Leader: qinjunbin@theplant.jp
- IOS Developers: qinjunbin@theplant.jp, neo@theplant.jp, yantong@theplant.jp
- Android Developers: lin@theplant.jp, john@theplant.jp, faqianglv@theplant.jp



# Configuration, build and run

1. Dev Env

   - Android Studio 4.0+

   - JDK 1.8+

2. Get KeyStore file from related developers or admin and put it into the project root directory	

3. Get the 'Store Password' and 'Key Password' from related developers or admin and configure them into system env

> ~ MacOs ~/.bash_profile
>
> export STOREPASSWORD= 'Store Password'
>
> export KEYPASSWORD='Key Password'

4. Choose one build variant to build and run
   - Dev: not to be used anymore
   - Staging: the development env
   - UAT: the User Acceptance Testing env
   - Product: the online product env	



# Modules



# The basic technology stack

- Dev language: Java & Kotlin
- DI: Dagger, Butterknife
- Http/Https: Volley, Retrofit & Okhttp
- Images process: Picasso, UniversalImageLoader
- Data parse: Gson, Moshi
- Data base: Realm
- Flow Control: Rxjava & RxAnroid



# Dev workflow

For example: [gitflow](GitFlow.md)



# Code style

For example: Android Open Source Code Style or [Custom](https://github.com/JohnZh/Android-Code-Style-Convention)



# Specific technical detail



# How to Release 



# Related Reference

- Design: 
  - JP：https://www.sketch.com/s/mWAmJ
  - EN：https://www.sketch.com/s/KrRVe
- API doc: 
- REQ doc:
- GA REQ doc: 