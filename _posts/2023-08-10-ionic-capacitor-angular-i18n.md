---
layout: post
title: "Making Angular Internationalization work with Ionic Capacitor"
tags: Ionic Angular Capacitor
---

In a work project, we used the built-in [Angular Internationalization](https://angular.io/guide/i18n-overview#angular-internationalization) for our ionic app. When trying to build the native version of the app, we ran into the following issue:
`The web directory (/path/to/project/www) must contain an "index.html".`. This is due to the fact that the index.html file is located in the respective language subfolder (e.g. www/en-US) after the angular localize feature did it’s job.

 <!--more-->

It is often suggested to use the [NGX Translate package](https://github.com/ngx-translate/core) instead, as it is able to switch dynamically between languages inside the app instead of building two separate versions of the app. We preferred to use the Angular i18n over the NGX Translate package since it’s the officially supported approach that is guaranteed to work with new angular versions. Especially because Ngx translate was abandoned for a long time and only recently got revived.

## This is our current solution/workaround


We managed to build the native app with both English and German languages packaged in the same APK.

The ionic build:after hook allows to execute custom javascript as part of the capacitor build process. The code is executed after the angular build and therefore after the localization.
We wrote a small js script, that creates a custom `index.html` file in the root of the `www` directory, which redirects to the correct language-specific version of the app within the corresponding folder (in our case either "de" or "en-GB").

We added the following to the ionic.config.json:

{% highlight json %}
  "hooks": {
    "build:after": "src/scripts/after-angular-build-hook.js"
  }
{% endhighlight %}

The contents of the "after-script.js" are as follows (for "de" and "en-GB" locales):

{% highlight javascript %}
// File needs to be CommonJs
const fs = require("fs");

const INDEX_HTML_PATH = "./www/index.html";
const htmlContent =
  '<script type="text/javascript">const currentLocale = navigator.language; if (currentLocale === "de") { window.location.href = "./de/index.html"; } else { window.location.href = "./en-GB/index.html"; } </script>';

module.exports = function () {
  console.log("creating index.html...");
  createIndexHtml();
};

function createIndexHtml() {
  fs.writeFile(INDEX_HTML_PATH, htmlContent, (error) => {
    if (error) {
      throw new Error(`Error writing file: ${INDEX_HTML_PATH}`, { cause: error });
    } else {
      console.log("Successfully created file:", INDEX_HTML_PATH);
    }
  });
}
{% endhighlight %}

In order to ensure the Android app is showing the correct language based on the current system language, we needed to add some code to the `MainActivity` in the Android source folder of our ionic project.

{% highlight java %}
public class MainActivity extends BridgeActivity {
  private static final String LOCALE_DE = "de";
  private static final String LOCALE_EN_GB = "en-GB";

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
    if (deviceLocale.getLanguage().equals(LOCALE_DE)) {
      setLocale(LOCALE_DE);
    } else {
      setLocale(LOCALE_EN_GB);
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

Build the app e.g. with the `ionic capacitor build android` command should now succeed. When running as a native app with `ionic capacitor run android` switching the system language of the Android phone should now lead to the app being loaded in that language.
