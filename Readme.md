
Its been over 6 months since I built and released OneClick, and in that time Google has changed [the requirements for installing extensions](http://developer.chrome.com/extensions/manifestVersion.html#manifest-v1-changes). They've also had a team busy building [Yeoman](http://yeoman.io) which looks like a great development/scaffolding tool. They have a generator for chrome apps, and I know there's a grunt plugin for building crx files, and I want to use all these things!

So I'm going to blog while I do the rewrite using these tools. Here it goes.

__(Sun 4:52 PM)__  
* created https://github.com/pwmckenna/oneclick_yeoman  
* found https://github.com/yeoman/generator-chromeapp#readme  
* ran `yeoman init chromeapp`

```
Please answer the following:
[?] What would you like to call this application? (OneClick) 
[?] How would you like to describe this application? (Download torrent content with the ease of a standard download.) 
[?] Would you like to use Storage in your app (Y/n) n
[?] Would you like to the experimental Identity API in your app (Y/n) n
[?] Would you like to use the Browser Tag in your app (Y/n) n
[?] Would you like to use the Camera in your app (n) n
[?] Would you like to use the Microphone in your app (n) n
[?] Would you like to use USB in your app (n) n
[?] Would you like to use Bluetooth in your app (n) n
[?] Would you like to use the Serial Port in your app (n) n
[?] Would you like to send UDP data in your app (n) n
[?] Would you like to receive UDP data in your app (n) n
[?] Would you like to use TCP in your app? (n) n
[?] Would you like to use the Media Gallery API in your app? (n) n
[?] Do you need to make any changes to the above before continuing? (y/N) 

   create    .editorconfig
   create    .gitattributes
   create    .jshintrc
   create    .npmignore
   create    app/.buildignore
   create    app/_locales/en/messages.json
   create    app/icon-128.png
   create    app/icon-16.png
   create    app/index.html
   create    app/main.js
   create    app/manifest.json
   create    app/styles/main.css
   create    Gruntfile.js
   create    test/index.html
   create    test/lib/chai.js
   create    test/lib/expect.js
   create    test/lib/mocha/mocha.css
   create    test/lib/mocha/mocha.js
   create    test/runner/mocha.js
identical    app/index.html
identical    app/manifest.json
identical    app/_locales/en/messages.json
```