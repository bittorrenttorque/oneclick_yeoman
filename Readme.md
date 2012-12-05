```
git submodule init //this is to pull in the .pem file for signing
git submodule update
yeoman install //pull in all the browser extension javascript dependencies
```
Add the `app` directory as an unpacked chrome extension on the [extension page](chrome://chrome/extensions/)
```
npm install //pull in the crx packaging dependencies
```

#Back Story

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

* Loaded the unpacked extension via the [chrome://chrome/extensions/](chrome extensions page)...made sure to select the app directory, not the root.
* The permissions in the manifest were busted and showed a warning. Replaced the array with a single empty string with an empty array. Need to remember to submit a patch to [the Yeoman ChromeApp Generator project](https://github.com/yeoman/generator-chromeapp).

K. I think I'm ready to roll porting everything over. I recently added support for [Bower](http://twitter.github.com/bower/) to both [Btapp.js](https://github.com/bittorrenttorque/btapp/) and [Backbrace.js](https://github.com/bittorrenttorque/backbrace/), so I'm going to attempt to take advantage of that when reassembling this extension.

__(Sun 5:13 PM)__  

Looks like I'm in charge of dinner...I'll be back. 

__(Sun 6:32 PM)__  

Back in the game!

Lets add the btapp and backbrace libraries.
```bs
yeoman install btapp
yeoman install backbrace
```
These libraries and their dependencies are put into a components directory. Lets update manifest.json to add those as background scripts.

__manifest.json__
```json
{
   //...
   "background": {
      "scripts": [
        "components/jquery/jquery.js",
        "components/underscore/underscore.js",
        "components/backbone/backbone.js",
        "components/jStorage/jstorage.js",
        "components/btapp/btapp.min.js",
        "components/backbrace/backbrace.js",
        "main.js"
      ]
   }
   //...
}
```

To test that we can still connect to the underlying Torque client, I updated main.js to the following.

__main.js__
```js
jQuery(function() {
    var btapp = new Btapp();
    btapp.connect({
        mime_type: 'application/x-bittorrent-torquechrome'
    });
    btapp.on('all', _.bind(console.log, console));
});
```
No luck! We need to make this extension a npapi plugin, and provide the dll and plugin directory to load the native mime type handlers. Lets copy the following from the previous manifest.json over to the new one.

__manifest.json__
```js
{
   //...
   "plugins": [
      // { "path": "plugin-linux.so" },
      { "path": "plugins/npTorqueChrome.dll" },
      { "path": "plugins/TorqueChrome.plugin" }
   ]
   //...
}
```

__(Sun 10:21 PM)__  
After porting the javascript in the old application.js file to main.js, everything was up and running. I got a bit distracted trying to figure out how to build on top of bower dependencies, but I'll save that for a blog post, and potentially a Yeoman generator.

I added a `component.json` file so that I wouldn't need to include the component directories in the project, instead pulling them in by running `yeoman install`. The file looked like the following:

__component.json__
```json
{
  "name": "OneClick",
  "version": "0.8.0",
  "dependencies": {
    "btapp": "latest",
    "backbrace": "latest"
  }
}
```

Next up is figuring out how to package the extension. I'm using https://github.com/oncletom/grunt-crx as a starting point.

To ensure we have the grunt-crx node modules, we need to create a `package.json` file with grunt-crx as a dependency. It looks like the following:

__package.json__
```json
{
  "name": "OneClick",
  "version": "0.8.0",
  "description": "OneClick Chrome Extension for easy torrent content downloading.",
  "author": "Patrick Williams <patrick@bittorrent.com>",
  "dependencies": {
    "grunt-crx": "0.1.2"
  }
}
```

Got distracted with a few random patches to projects...but now jstorage is available via bower/yeoman and has a proper component.json file.

__(Mon 2:32 AM)__  
Only a tiny bit left...just create a crx grunt task and call it a day. I have the pem file in its own repo located in the same parent directory as this project.
```
{
   //...
   grunt.loadNpmTasks('grunt-crx');
   //...
   grunt.initConfig({
      crx: {
         oneclick: {
           "src": "app/",
           "dest": "dist/",
           "privateKey": "../oneclick_pem/oneclick.pem",
           "baseURL": "http://torque.bittorrent.com/oneclick/",
           "exclude": [ 
             ".git", 
             ".svn", 
             "components/*/*/**"
           ],
           "filename": "OneClick.crx"
         }
       }
    }
    //...
 }
```

now running `yeoman crx` builds a signed crx file and drops it into a dist directory.

DONE!
