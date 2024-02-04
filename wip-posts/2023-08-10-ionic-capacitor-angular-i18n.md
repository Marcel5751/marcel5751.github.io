---
layout: post
title: "Making Angular Internationalization work with Ionic Capacitor"
---


We also encountered this issue in our organization. We preferred to use the [Angular Internationalization](https://angular.io/guide/i18n-overview#angular-internationalization) over using the NGX Translate package, but ran into the issue that `The web directory (/path/to/project/www) must contain an "index.html".`

With the following approach, we managed to build the app with both English and German languages packaged in the same APK.

The ionic `build:after` hook allows to execute custom javascript as part of the capacitor build process but after the angular build and therefore after the localization.
We wrote a small js script, which creates a custom `index.html` file in the root of the `www` directory, which redirects to the correct language-specific version of the app within the corresponding folder (in our case either "de" or "en-GB").

Add the following to the `ionic.config.json`:

{% highlight json %}
  "hooks": {
    "build:after": "src/scripts/after-script.js"
  }
{% endhighlight %}

The contents of the "after-script.js" look as follows (for "de" and "en-GB" locales):

{% highlight javascript %}
// File needs to be CommonJs
module.exports = function () {
  console.log("creating index.html...");

  const fs = require("fs");
  const indexHtmlPath = "./www/index.html"
  const htmlContent =
    '<script type="text/javascript">var currentLocale = navigator.language; if (currentLocale === "de") { window.location.href = "./de/index.html"; } else { window.location.href = "./en-GB/index.html"; } </script>';

  fs.writeFile(indexHtmlPath, htmlContent, (error) => {
    if (error) {
      console.error('Error writing to file:', error);
    } else {
      console.log('Successfully wrote to file:', indexHtmlPath);
    }
  });
};
{% endhighlight %}

In order to ensure the Android app is served in the correct language, we needed to add some code to the `MainActivity` in the Android source folder of our ionic project.

{% highlight java %}
public class MainActivity extends BridgeActivity {
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setLocaleFromDeviceLanguage();
  }

  private void setLocaleFromDeviceLanguage() {
    Locale deviceLocale;
    // The getResources().getConfiguration().locale method has been deprecated in Android N (API level 24).
    // It's better to use getResources().getConfiguration().getLocales().get(0) to get the current locale.
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
      deviceLocale = getResources().getConfiguration().getLocales().get(0);
    } else {
      deviceLocale = getResources().getConfiguration().locale;
    }

    // Set the app's locale based on the detected language
    if (deviceLocale.getLanguage().equals("de")) {
      setLocale("de");
    } else {
      setLocale("en-GB");
    }
  }

  private void setLocale(String localeCode) {
    Locale locale = new Locale(localeCode);
    Locale.setDefault(locale);

    Configuration config = new Configuration();
    config.locale = locale;

    getBaseContext().getResources().updateConfiguration(config, getBaseContext().getResources().getDisplayMetrics());
  }
}
{% endhighlight %}

Build the app e.g. with the `ionic cap sync android` command and then run as a native app with `ionic cap run android`.
When switching the system language of the Android phone, the app should now be loaded in that language.
