# MobileIron

Introduction
------------
MobileIron is a suite of products aimed at securing and managing corporate data when people access cloud data using mobile devices and modern endpoints. It is a platform based on unified endpoint management (UEM). MobileIron was founded in 2007 and is mobile-centric, zero trust platform built on a unified endpoint management (UEM) foundation. MobileIron’s mobile-centric, zero trust approach ensured that only authorised users, devices, apps and services could access business resources. MobileIron was acquired by Ivanti December 2020.

The Ivanti (formerly MobileIron) products together provide MDM (Mobile Device Managment) or MAM (Mobile Application Managment). It provides a cloud-based service used to manage devices and applications for users. Users can access apps through the Ivanti products while allowing you to manage and secure content on the network. In a MDM (Mobile Device Managment) setup, the entire device is managed, while in MAM (Mobile App Managment) each application is managed separately.

Goals for using MobileIron
---------------------------
Our goals for using MobileIron are to:
1. Secure the app (containerize)
2. Allow users to easily login with a certificate/passcode instead of through the SSO portal
3. Allow VPN connections (i.e to allow access to BNPP resources on BNPP servers)

With these goals in mind, the two main products from the Ivanti suite that we care about here are Ivanti AppConnect and Ivanti AppTunnel. Although some other Ivanti products are requried to make these work - we will mention them when relevant - tangetially as it relates to AppConnect or AppTunnel.

AppConnect protects data on a device -- data-at-rest. [Goal 1]
AppTunnel protects data as it moves between a device and enterprise data sources -- data-in-motion. [Goal 3]
AppConnect (with App Specific Configuration) + AppTunnel - user identification/authorisation. [Goal 2]

AppConnect
----------------------
Ivanti AppConnect containerizes apps to protect corporate data-at-rest. AppConnect essentially wraps mobile apps in a secure container to make it work within the Ivanti/MobileIron ecosystem. Once applications are wrapped with the AppConnect wrapper they become integrated into the secure container on the device. The secure container can include secure email, secure documents, and secure apps, including a secure browser. The apps’ data and documents are encrypted in the container, which requires authorized access. The apps can interact only with other AppConnect apps. The app wrapper provides AppConnect capabilities and security to the app - prevents the app from leaking data outside the secure container. Each app becomes part of a secure container whose data is encrypted, protected from unauthorized access, and removable. On the device, the apps are known as secure apps.

The devices running AppConnect apps (secure apps) are registered to a MobileIron server. MobileIron Core is the on-premise self-hosted server, and MobileIron Cloud is the cloud offering. BNPP runs a MobileIron Core server. The MobileIron server is used to distribute the apps. You cannot distribute AppConnect apps on Google Play. Any AppConnect iOS apps on the app store must be dual-mode (built using the SDK allowing them to work as an AppConnect app, and as a normal app).

For AppConnect apps to work on a device, the mobile device has to have two other apps installed - Mobile@Work, and Secure Apps Manager. Alternatively, these would be AppStation and Secure Apps Manager for AppStation if BNPP was managing only specific apps and not the whole device (MAM instead of MDM).

The first app (Mobile@Work or AppStation) fetches from the server (MobileIron Core) and applies the appropriate policies and AppConnect apps to the device. The second app - Secure Apps Manager (or Secure Apps Manager for AppStation) works with the first app to handle data encryption and the passcode used for logging on to AppConnect apps.

These app combinations can be installed from Google Play/App Store by the device user. Normally, only the first app (Mobile@Work) needs to be installed, as this app will either be bundled with or itself install the Secure Apps Manager (i.e. the device user receives the Secure Apps Manager from the server).

You create an AppConnect app using the Ivanti AppConnect wrapping technology. This wrapping technology transforms an app into a secure app with minimal app development. You can wrap an APK/IPA file that was created using the React Native mobile development framework.

AppConnect wrapping does the following:
1. Examines an app’s APK file for operating system calls that impact security.
2. Replaces these calls with secure AppConnect calls.
3. Generates a replacement APK file.

To wrap an app (make it an AppConnect app), you either:
- Upload it to the AppConnect Wrapping Portal.
	* Ivanti signs the wrapped apps with the Ivanti private key. Ivanti also signs the Secure Apps Manager and all secure apps provided by Ivanti with the Ivanti private key. This is the simplest and most common way to wrap apps.
- Use the AppConnect Wrapping Tool.
	* This is a desktop app used to wrap and sign your apps. Use the wrapping tool only if there is a need to sign apps with an enterprise private key instead of the Ivanti/MobileIron private key. Signing apps with an enterprise private key instead of the Ivanti private key is a security decision that BNPP would have already made. The wrapping tool can be used only for apps that are distributed by MobileIron Core. We will use the wrapping tool if BNPP requires that the MobileIron apps are signed with BNPP's own enterprise private key.

When wrapping an app with the AppConnect Wrapping Portal, you select an option that determines whether the app works with the regular Secure Apps Manager or with the Secure Apps Manager for AppStation. Therefore, if you want your app to work with either the regular Secure Apps Manager or the Secure Apps Manager for AppStation, you must wrap it twice: once without the AppStation option and once with the AppStation option.

When you wrap an app without the AppStation option, it has the following package name:
 > forgepond.com.bnpparibas.globalmarkets

When you wrap an app with the AppStation option, it has the following package name:
 > appstation.com.bnpparibas.globalmarkets

There are two variations of the wrapper (Generation 1 and Generation 2). When generating a wrapped app, we have to choose which Generation wrapper to use. The two generations support different functionality that might be used within wrapped apps. For our app, we have to use the Generation 2 wrapper. This is due to it being a React Native app and the functionality that we wish to support. The targetSdkVersion of our app is set to 30 which requires using the Generation 2 wrapper, and AppConnect applies necessary changes to AndroidManifest.xml if the app uses targetSdkVersion=30. Also as we require VPN tunelling from a React Native app, we have to use the Generation 2 Wrapper.

## MobileIron Core server Configuration
Using AppConnect requires that an AppConnect Devices configuration is set up. This configuration specifies settings that are not specific to a particular AppConnect app such as the AppConnect passcode requirements and data loss protection(DLP) requirements. MobileIron provides a default AppConnect Devices configuration. This likely already exists within the BNPP set up.



## Ivanti/MobileIron Components that we use/rely on
### MobileIron Core (now Ivanti Endpoint Manager Mobile (EPMM))
This is one of Ivanti's EMM (Enterprise Mobility Management) solutions. It is the on-premise server that provides security and management for an enterprise’s devices, and for the apps and data on those devices. An administrator configures the security and management features using a web portal. It integrates with backend enterprise IT systems and enables IT to define security and management policies for mobile apps, content and devices. BNPP already has a MobileIron Core server.

MobileIron Core now uses only TLSv1.2 for incoming and outgoing connections with all external servers.

### Standalone Sentry
The server that provides secure network traffic tunneling from the app to enterprise servers. It is an in-line gateway that manages, encrypts, and secures traffic between the mobile device and back-end enterprise systems. Sentry addresses three fundamental needs: mobile security, scalability and user experience.

### Mobile@Work
The Core client app that runs on the device. It interacts with the Core server to apply the appropriate policies and AppConnect apps to the device. It also interacts with the Secure Apps Manager. The device user gets Mobile@Work from Google Play or the AppStore.

Mobile@Work works with Core to:
- Configure VPN and security certificates to create a clear separation between personal and business information.
- Install the enterprise app storefront so that device users can browse and install the mobile applications that administrators make available to them.
- Allow device users to access web resources and content repositories that sit behind the firewall.

### Secure Apps Manager
Another Ivanti app that runs on the device. Working with Mobile@Work, it handles data encryption and the AppConnect passcode for logging on to AppConnect apps. The device user receives the Secure Apps Manager from Core.



App Specific Configuration
-------------------------------
With some additional development, an app can receive app-specific configuration from the Ivanti server.

The MobileIron server passes the app-specific configuration to the MobileIron client app (Mobile@Work). The MobileIron client app in turn passes the configuration to the AppConnect wrapper around the app, which passes it to the app.

The app can receive these key-value pairs. Specifically, after implementing configuration handling in the app, the app:
-requests the current configuration when it first runs, receives an asynchronous response containing the key-value pairs.
-receives updates to the configuration.

We can implement app-specific configuration in a React Native app by using MobileIron-provided files that make
up a React Native package called ConfigServicePackage. The files provide the necessary APIs to receive the app specific configuration from the MobileIron server.

The code we add to the app to receive app-specific configuration is simple as the AppConnect wrapper
around the app and the MobileIron client do most of the work. The focus is in applying the configuration to the app according to our requirements.

To handle app-specific configuration in the app, we need to do the following high-level tasks:
- Check at runtime if the app is wrapped.
	> This check is typically necessary if using the same source code to create a Google Play app and an in-house AppConnect app. Only wrapped AppConnect apps can receive app-specific configuration from a MobileIron server. If developing an app that will be distributed only as an in-house app, not from Google Play/AppStore, there will be no need to use this check.
- Create a callback method to receive configuration updates
	Provide a callback method that receives app-specific configuration updates.
	IMPORTANT: The callback thread runs on the main thread.
- Request the configuration when your app starts.
	> When the app starts, request the app-specific configuration, which the app will receive asynchronously in the callback method you provide.
- Add callback information to AndroidManifest.xml
	> You provide a callback method that receives app-specific configuration updates. Add information about your callback method to your app’s AndroidManifest.xml file. You add this information as a <meta-data> element in your <application> element.
- Specify app configuration and policies in .properties files.
	> It is possible to include .properties files in the app that list the app’s key-value pairs and data loss prevention (DLP) policies. When the MobileIron server administrator uploads the app to the server, these files cause the server to automatically configure the key-value pairs and DLP policies.


We determine the app-specific configuration that the app requires from the Ivanti server.
Only unwrapped version of the app should send user to SSO to login.

Examples of what may be provided are/app specific configuration we may want to consider:
- the address of a server that the app interacts with
- whether particular features of the app are enabled for the user
- user-related information from LDAP, such as the user’s ID and password
- certificates for authenticating the user to the server that the app interacts with
- The stuff in env files

Each configurable item is a key-value pair. Each key and value is a string. A server administrator specifies the keyvalue pairs on the server for each app. The administrator applies the appropriate set of key-value pairs to a set of devices.

# Configuration
The following configuration files are included in the app through this package:
- appconnectconfig.properties
  This file specifies the app’s configuration keys and their default values, if any. This .properties file causes the MobileIron server to automatically configure the keys and their default values on the server.
- AppConnect.plist
  Core uses the values entered in the AppConnect.plist to create and populate an AppConnect app configuration and AppConnect container policy  

If the app contains these .properties/plist files, the MobileIron server automatically configures the key-value pairs specified. This automatic configuration occurs when the MobileIron server administrator uploads the app to the server’s App Catalog.

The administrator can then change the default values on the server as necessary for BNPP and for this app.

File location of the .properties file within the app: <application root directory>/res/raw

## AppConnect apps authentication
AppConnect apps can provide device users a seamless authentication experience where users do not have to enter any credentials when accessing enterprise applications. 

The following methods are available to support this capability:

-Authentication using Kerberos Constrained Delegation
-Certificate authentication for Android and iOS AppConnect apps
    Setting up certificate authentication from an AppConnect app requires the following two main steps:
    1. Configure MobileIron Cloud with the certificate that the app will use to authenticate to the enterprise service.
      a. Add a certificate authority in Admin > Certificate Management > Certificate Authority
      b. Create an Identity Certificate setting, in Configurations > Add > Identity Certificate. For Certificate Distribution, select Dynamically Generated and for Source, select the certificate you configured in Admin > Certificate Management > Certificate Authority.
    2. Add two sets of key-value pairs to the AppConnect Custom Configuration.
      > The two key-value pairs in each set specify:
          * a certificate
          * a URL matching rule

    When the app makes a web request to a URL that matches a URL matching rule, the connection uses the certificate.

    The user certificate presented to the enterprise server by the app can be specifically for the enterprise server only, or a default user certificate if we do not require a specific certificate for the service. One other option is to use the same certificate that the app presents to the Standalone Sentry. The certificate is either an identity certificate or a group certificate. The AppConnect library, which is part of every AppConnect app, makes sure the connection uses the certificate.


    -----EXAMPLE CONFIG BLOCK-----
    *==== Based on page 83 of https://help.ivanti.com/mi/legacypdfs/AppConnect%20Guide%20for%20Cloud.pdf *====

    Standalone Sentry Configuration (sentry1.bnpparibas.com?)
      App Tunnel Configuration:
        - Service Name: TCP_GLOBALMARKETS
        - Server List: echonet.bnpparibas.com:443
      AppConnect App Configuration for the Globalmarkets secure app:
        - AppTunnel Rules
          * URL Wildcard: echonet.bnpparibas.com
          * Port: 80
          * Sentry: sentry1.bnpparibas.com
          * Service: TCP_GLOBALMARKETS
          * Identity Certificate: {certificate goes here}
        - App-specific Configurations
          * ES_CERT_AUTH_SERVICES: TCP_GLOBALMARKETS
          * TCP_GLOBALMARKETS_CERT: {certificate goes here}


The user would authenticate to the Standalone Sentry using the identity certificate defined under the AppTunnel rules.

In the app’s AppConnect app configuration, the value of ES_CERT_AUTH_SERVICES lists the service that uses certificate authentication.

The app can use a specific certificate (defined as TCP_GLOBALMARKETS_CERT), or a default certificate (defined as ES_CERT_DEFAULT), to authenticate to its enterprise
server.

So essentially, the app would just need to send what is stored in TCP_GLOBALMARKETS_CERT (or ES_CERT_DEFAULT).

To customize the app behavior or add certificates in the app, create an AppConnect Custom Configuration.




AppTunnel with TCP tunneling
---------------------------------
Tunneling is a way to move packets from one network to another. Tunneling works via encapsulation: wrapping a packet inside another packet. 

AppTunnel is particularly useful when an organization does not want to open up VPN access to all apps on the device. This feature requires a Standalone Sentry configured to support app tunneling. A Standalone Sentry is necessary to support AppTunnel with TCP tunneling.

There are two types of AppTunnel provided by AppConnect:
1. AppTunnel with HTTP/S tunneling
2. AppTunnel with TCP tunneling

AppTunnel with HTTP/S tunneling is not supported for React Native apps. React Native apps, when making network requests do not use the HTTP/S APIs that `AppTunnel with HTTP/S tunneling` supports (and is another reason the Generation 2 wrapper has to be used for React Native Apps). Therefore AppTunnel with TCP tunneling is what we will be using for VPN tunelling.



Once configured on the MobileIron Core server, the AppConnect wrapper, the Secure Apps Manager, and the MobileIron client app, manage TCP tunneling. No additional app development is necessary.


AppConnect apps for iOS support TCP tunneling using the MobileIron Tunnel app. Therefore, to use TCP tunneling for iOS AppConnect apps, in addition to Standalone Sentry, also deploy and install MobileIron Tunnel on iOS devices and apply the Tunnel VPN profile to the iOS AppConnect app.


When an app uses AppTunnel with TCP tunneling, the traffic between the device and the Standalone Sentry is secured using an Secure Sockets Layer (SSL) session

Apps relying on AppTunnel with TCP tunneling can use only passthrough authentication or Certificate authentication. Due to the authentication credentials being sent by the sentry server to the enterprise server, not from the client device, AppTunnel with TCP tunneling does not support Kerberos authentication to the enterprise server.

TCP tunneling for AppConnect apps is set up differently for iOS and Android.
	- Android AppConnect apps
		AppConnect apps for Android that are wrapped with the Generation 2 wrapper support TCP tunneling using AppTunnel.
	- iOS AppConnect apps
		AppConnect apps for iOS support TCP tunneling using the MobileIron Tunnel app. Therefore, to use TCP tunneling for iOS AppConnect apps, in addition to Standalone Sentry, also deploy and install MobileIron Tunnel on iOS devices and apply the Tunnel VPN profile to the iOS AppConnect app.





With passthrough authentication, the Standalone Sentry passes the authentication credentials, such as the user ID and password (basic authentication) or NTLM, to the enterprise server. 

Certificate authentication with AppTunnel with TCP tunneling.
----------------------------------------------------------------------------
Secure Apps supports certificate authentication with AppTunnel with TCP tunneling. This feature is supported only with AppTunnel with TCP tunneling. An app that uses AppTunnel with TCP tunneling can send a certificate to identify and authenticate the app user to an enterprise server. Depending on the server implementation, this authentication occurs without interaction from the device user beyond entering the AppConnect passcode, if one is required. That is, the device user does not need to enter a user name and password to log into enterprise services. Therefore, this feature provides a higher level of security and an improved user experience.


No additional app development is necessary. However, the feature is supported only if the app:
		> initiates a connection that does not use Secure Socket Layer (SSL) to the enterprise server. For example, the app can initiate the connection with a HTTP request, but not with an HTTPS request.


Certificate authentication with AppTunnel with TCP tunneling is supported only if the app: initiates a connection that does not use Secure Socket Layer (SSL) to the enterprise server (Sentry server maybe?). For example, the app can initiate the connection with a HTTP request, but not with an HTTPS request.

IMPORTANT: The connection that this feature makes to the enterprise server is secure; it uses SSL.



Once configured, the AppConnect wrapper, the Secure Apps Manager, and the MobileIron client app, manage TCP tunneling.


For users, what this all means is that:
-----------------------------------------
A device user can use an AppConnect app only if:
- the device user has been authenticated through the MobileIron server.
	The user must use the Mobile@Work, or MobileIron AppStation to register the device with the MobileIron server. Registration authenticates the device user. Only registered devices can use an AppConnect app.
- the server administrator has authorized the device user to use the AppConnect app.
- the device user has entered the passcode for using AppConnect apps, if required by the server administrator.
	With the AppConnect passcode, the device user can access all the AppConnect apps.








This package is to allow integration of Ivanti (formerly MobileIron) AppConnect into a React Native application, through the use of React Native modules and components.








Automated App Release

App distribution and configuration
 > Apps@Work is an enterprise app storefront that facilitates the secure distribution of mobile apps. iOS Managed Apps and Android Enterprise enable easy configuration of app-level settings and security policies.












Getting Started
---------------

1. 

2. To install internal app dependencies:

   $ npm install

   - This will create the node_modules directory in the mobileiron directory.
     The installation refers to package.json to install dependencies and the target version.

   - The settings.gradle file contains the path to dependency libraries in node_modules.

   - If package.json is updated, run 'npm install' again to update the node_modules directory.

3. To start development and run the debug build on a device:

4. To create the release APK:



## Package setup and usage

This library comes with platform-specific (native) code and takes advantage of autolinking. Autolinking is a mechanism that allows your project to discover and use this code (https://github.com/react-native-community/cli/blob/master/docs/autolinking.md).






Common Code
-----------

JavaScript Files:

- index.js
- ConfigCallback.js

Java Files:

- ConfigCallbackPackage.java
- ConfigCallbackModule.java
- AppConfigCallback.java


- ConfigCallback.js gives access from JavaScript to ConfigCallback implemented in Java.

  Usage:
    ConfigCallback.requestConfig() // Returns a Promise object.
      .then((response) => {       // Resolved with the app-specific configuration object.

        if (response.hasOwnProperty("url")) // The response field names correspond to key names in
                                            // the configuration.
          console.log(response.url)         // The field values correspond to values in
                                            // the configuration.
      }

- ConfigCallback/ConfigCallbackModule/ConfigCallbackPackage provide APIs to request app-specific configurations from the MobileIron server and receive them in a Promise object on the JavaScript side.

To get this to work, we need the autolinking to:
  1. Copy the following files from `/android/app/src/main/java/com/configcallback/` to the corresponding java directory in the app.
     - AppConfigCallback.java
     - ConfigCallbackModule.java
     - ConfigCallbackPackage.java

  2. Add ConfigCallbackPackage in the getPackages method of the MainApplication.java file:

    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
              new MainReactPackage(),
              new ConfigCallbackPackage()
              ...);
    }

  3. Add callback package and method to the AndroidManifest.xml file:

     <meta-data
         android:name="com.mobileiron.appconnect.config.callback_class"
         android:value="com.configcallback.AppConfigCallback" />

     <meta-data
         android:name="com.mobileiron.appconnect.config.callback_method"
         android:value="onConfigReceived" />
















# Required app development














Examples of external servers to which Core makes outgoing connections are:
> Standalone Sentry
> Connector
> SCEP servers
> LDAP servers
> Core Gateway
> Apple Push Notification Service (APNS)
> Content Delivery Network servers
> Core support server (support.ivanti.com)
> Outbound proxy for Gateway transactions and system updates
> SMTPS servers
> Public app stores (Apple, Google, Windows)
> Apple Volume Purchase Program (VPP) servers
> Apple Device Enrollment Program (DEP) servers
> Android for Work servers





Apple requires all customers who wish to manage their iOS devices using an MDM service, such as MobileIron, to set up an iOS MDM Certificate (Admin > MDM Certificate). This certificate allows secure communication between the Apple Push Certificate Portal and the MobileIron service.
 1. DOWNLOAD the Certificate Signing Request file (ApplePushInfo.txt) from the MobileIron Core Portal.
 2. GET the MDM Certificate by logging into Apple’s Push Certificate Portal.
 3. UPLOAD the MDM Certificate acquired in Step 2 to the MobileIron admin portal.


Enroll with Android Enterprise/Register With Google
 Enroll Android enterprise devices with Managed Google Play Accounts

 User devices are enrolled without sending any personal information (such as email addresses) to Google. MobileIron Cloud will provision and manage users automatically with Google.

 You will need an admin Google account to enroll.

 By skipping the Managed Google Play Account enrollment, you will not be able to register Android Enterprise devices.

 Android devices can still be registered with device admin, but key features such as Managed Google Play and App Config will not be available for use.

 You can complete the enrollment with Google later under Admin > Android > Android Enterprise



You can directly search and add apps from the two app stores into MobileIron from the UI.

You can manually invite users to register their devices. Once user(s) register they will receive an email and password configurations and access to all the added apps in the app catalog.

You can create and add users to user groups.




# Steps required
1. Adding AppConnect apps to MobileIron Core server
2. Adding AppConnect Custom Configuration
3. Adding an AppConnect Devices Configuration



# Capabilities and limitations and versions
The following app capabilities are supported by only the Generation 2 wrapper:
-Data Encryption of all file I/O
-AppTunnel with TCP Tunneling as described in AppTunnel with TCP tunneling.
-Java Native Interface (JNI) call support:
	o Calling from Java to native code
	o Calling from native code to Java
-Passing file descriptors with Parcels across processes
-Calls to Runtime.exec are supported with a special flag.
-AccountManager APIs getAccounts(), getAccountsByType(), and getAccountsByTypeAndFeatures() are supported.
-The intents android.intent.action.OPEN_DOCUMENT, CREATE_DOCUMENT, and OPEN_DOCUMENT_TREE are supported for device storage and for SD card storage.
-Calls (by reflection) to the private method java.lang.Runtime.nativeLoad()
-Wrapping apps that use 64-bit native libraries (C or C++ libraries)
-Scoped storage


DownloadManager API
  - If using the Android DownloadManager API, the secure FileManager that MobileIron provides must also be installed on the device
MediaPlayer and MediaMetaDataRetriever Internet permission requirement
 - Wrapper support of these APIs require that wrapped apps set the Android Internet permission.

Latest version of AppConnect for Android: 9.4.0.0
Latest version of AppConnect for iOS: 4.8.1
