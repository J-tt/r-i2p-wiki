Using the Self-Installing i2p Browser Profile for Windows
=========================================================

In order to make i2p simpler to use and to encourage safer practices with web
browsers, I've created a browser profile for use with i2p and Firefox, coupled
with an easy and reliable installer and launcher that will be familiar to
Windows users. This profile endeavors to make everyday browsing with i2p safer
and to automatically separate your clearnet browser use and your i2p browser
user, while also disabling features of Firefox which are irrelevant or risky
in the context of browsing i2p.

Installation/Use:
-----------------

First, in order to use this browser profile, you will need to install Firefox if
you have not already. You can get the latest version from [Mozilla's web site](https://www.mozilla.org/en-US/firefox/new/).

Next, you should download this [this .exe installer](https://github.com/eyedeekay/firefox.profile.i2p/releases/download/current/install-i2pbrowser.exe)(URL: https://github.com/eyedeekay/firefox.profile.i2p/releases/download/current/install-i2pbrowser.exe)
and run it. It should detect your Firefox application automatically and
configure the profile and launcher scripts.

You're done! Now you can launch the i2p browser profile with the short-cuts that
have been made on your Start Menu and Desktop.

Notes:
------

With the standard launcher, which is labeled "I2PBrowser-Launcher" the profile
is used in regular browsing mode, and some information like logins can persist
between sessions. By launching "Private Browsing-I2PBrowser-Launcher" Firefox
will automatically delete session information when you close the browser.

For more information on it's development, see [the development repository](https://github.com/eyedeekay/firefox.profile.i2p/).

In the next major release of i2p, this profile/installer will be integrated with
java i2p automatically, with it's relevant launchers.
