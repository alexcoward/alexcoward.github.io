---
layout: post
title:  "Mac App Notarization"
date:   2019-10-17 14:00:30 -0700
---

## I Learned how to sign and notarize a Mac app this week!

<!--break-->

This week I learned how to package, sign and notarize a macOS Application (OSX 10.14). By signing and notarizing and application
we can 'help' ensure both the quality and security of our software, as through the notarization process Apple scans the submitted files for malicious software
and then allows Gatekeeper to find information related to our app, when a user launches it.

### Signing the .app file

1: First we must sign the app using our developer certificate stored in our keychain

{% highlight ruby %}

codesign --timestamp --options runtime -s "Developer ID Application: <org name>" /path/to/my.app
# We may need to use the --deep flag if we have nested folders that contains binaries, etc. (Note: this is discouraged by Apple)
# We must provide the --timestamp to generate a secure time stamp
# We must pass the --options runtime in order to allow a hardened runtime for our app for security purposes

{% endhighlight %}

2: Next we can verify the signature

{% highlight ruby %}

codesign --verify -vvvv Application.app
or
spctl -a -vvvv Application.app

{% endhighlight %}

### Uploading for notarization

The trick here is you need to first set up two factor authentication on your Apple account, and then you need to generate an app 
specific password to be used by altool for the below command.

1: Next we need to zip and upload the .app to apple

{% highlight ruby %}

 xcrun altool --notarize-app --primary-bundle-id "com.example.app"\ #bundle id from Info.plist
 --username "AC_USERNAME"\ #your Apple ID username
 --password "@keychain:AC_PASSWORD"\ #the one time password you generated online (saved in your keychain)
 --asc-provider <ProviderShortname>\ #ProviderShortname from your certificate
 --file Application.zip # The zip file where your .app file is
 
{% endhighlight %}

2: If notarization succeeds you will get back something like the following

{% highlight ruby %}

altool[16765:378423] No errors uploading 'Application.zip'.
RequestUUID = 2EFE2717-52EF-43A5-96DC-0797E4CA1041

{% endhighlight %}


3: Apple will email you the status of your notarization request, or you can check it yourself from the command line

{% highlight ruby %}

xcrun altool --notarization-history 0 -u "AC_USERNAME" -p "@keychain:AC_PASSWORD"

or 

xcrun altool --notarization-info 2EFE2717-52EF-43A5-96DC-0797E4CA1041 -u "AC_USERNAME"

{% endhighlight %}

4: Lastly we also attach the notarization ticket to our .app file using the stapler tool, so that future distributions include the ticket.
This ensures that Gatekeeper can find the ticket even when a network connection isn’t available. First be sure to unzip the .app

{% highlight ruby %}

 xcrun stapler staple "Application.app"

{% endhighlight %}

<br><br>

## Next Steps

Now you have signed and notarized .app file, next you need to:

1: Create a .dmg file from the folder containing the .app file or add the .app file to the .dmg

2: Sign the .dmg file using the above command

3: Notarize the .dmg file using the above command (you can send the .dmg file, no need to compress)

4: Staple the notarization to the .dmg using the above command

5: Distribute your app via Jamf, or via the Apple store!

