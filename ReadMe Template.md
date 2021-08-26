\* [[Title] eg. Japanese Macdonald's App](#title-eg-japanese-macdonalds-app)

\* [Glossary](#glossary)

\* [Members and roles](#members-and-roles)

\* [Configuration, build and run](#configuration-build-and-run)

\* [Modules](#modules)

\* [The basic technology stack](#the-basic-technology-stack)

\* [Dev workflow](#dev-workflow)

\* [Code style](#code-style)

\* [Specific technical detail](#specific-technical-detail)

\* [How to Release](#how-to-release)

\* [Related Reference](#related-reference)

# [Title] eg. Japanese Macdonald's App

Short description for project. 

eg. This is Japanese Macdonald's App Project, or JMA for short.

This is Android version. IOS version: https://github.com/theplant/mcd-ios



# Glossary

Some professional and unreadable words about project and business. 

eg. **NGLP(next-gen loyalty program)**: mainly for issuing coupons that are used at the counter. The user management module also sits in NGLP. Originally was a standalone app. The backend is developed by Plexure(PLX).



# Members and roles

- Project Manager: 
- Product Manager: 
- Tech Leader: 
- Backend Developers: 
- Frontend Developers: 
- IOS Developers: 
- Android Developers: 



# Configuration, build and run

Steps to help member to build and run the project.

eg. 

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
   - Staging: the development environment
   - UAT: the User Acceptance environment
   - Product: the online product environment	



# Modules

It's useful to help members know and manage this project if it's a multi-module project. 

Introduce modules by short description.



# The basic technology stack

To help newer to catch up this project, list the basic technology stack is necessary.

eg.

- Dev language: Java & Kotlin
- DI: Dagger, Butterknife
- Http/Https: Volley, Retrofit & Okhttp
- Images process: Picasso, UniversalImageLoader
- Data parse: Gson, Moshi
- Data base: Realm
- Flow Control: Rxjava & RxAnroid



# Dev workflow

A standard or unified development workflow is very useful to help members manage process, version control and risk

For reference, please see [JMA project readme file](Japanese%20Macdonald's%20App%20ReadMe.md) or [gitflow](GitFlow.md)



# Code style

Each program language has its own style, but for maintainability, we have to uniform it.

eg.

[Android Open Source Code Style](https://source.android.com/source/code-style.html#java-style-rules) or [Custom](https://github.com/JohnZh/Android-Code-Style-Convention)



# Specific technical detail

Some technical/program detail may be not common or standard, but it need all the programers to follow or notice.

eg.

- Use `Logger`  to print log instead of Android standard log util class `android.util.Log` or `System.out.print*`



# How to Release 

Steps to release project to staging, UAT, Beta and Product environment



# Related Reference

- Design: 
  - JP：
  - EN：
- API doc: 
- REQ doc:
- GA REQ doc: 