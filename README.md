Angular-stateManager
============

A simple state manager implementation for Angular.js which enables nested views, the browser back button, proper history, and deep linking.

Live demo
------------

http://angular-statemanager-demo.herokuapp.com

Wordy intro
------------

This state manager allows your app to transition to different states as the result of user actions, and conceptually links these states to different URLs in the app. The state itself is encoded in the URL, there are no objects or variables that the state manager maintains or lets your write to. 

This follows the philosophy of the web in the sense that a URL should be enough to describe the state of an app. Consequently when your app does something significant, e.g. you go from say the "Inbox" to the "Sent" folder (in say a webmail client) your App's URL will change from www.mymailapp.com/inbox to www.mymailapp.com/sent. If the user then decides to go back to the Inbox, the URL will change back to www.mymailapp.com/inbox and this should be enough for your app to be able to restore the state of the inbox. This is a contrived example and the user will probably be in a deeper nested URL structure like www.mymail.com/inbox/message/123/edit.

This is where this state manager comes in handy in that it lets each controller in the stack (root > inbox > message > edit) initialise itself according to the state represented by the URL. So the root controller will load the inbox subview/controller pair. The inbox controller will load the message. And the message will load the editor. This can happen asynchronously, i.e. the full stack is never known to any one controller, and each controller just has to worry about itself and its subviews. The subviews worry about themselves, and their subviews worry about themselves, etc. The state manager makes sure that each controller is initialised properly to reflect the current state once it is ready.

Sounds complicated but all you have to do is write an initialiser function in your controller which will get called when the URL (=state) changes. That's it. The state manager takes care of when to call the initialiser, and when to destroy the controller. If as a result of your initialisation function another (sub-)controller gets loaded after the fact, then as long as this sub-controller implements his own initialiser, the state manager will call his initialiser function once he's loaded. This is all asynchronous and each controller just has to worry about itself, resting safe in the knowledge that it can change the controller hierarchy and all will be well (by either adding a new controller, or replacing an old one, through say `ng-include` or by manually inserting a compiled element containing the `ng-controller` directive into the DOM).

Usage
------------

This walkthrough uses an "Animals" theme with dogs and cats.

The state manager itself comes as a module inside `stateManager.js`, which you just link to in your main html file:

```html
<script src='/js/stateManager.js'></script>
```

And then add a dependency in your app (assuming your app is called "Animals"):

```javscript
angular.module("Animals", ["stateManager"])
```

Then in your controllers provide the state manager with an initialiser which it can call when the state changes. `pathComponents` is an array of path components so `http://www.google.com/some/path/on/the/root/domain` would pass in `["some", "path", "on", "the", "root", "domain"]`. The state manager returns a function which you must call with the $scope as the first parameter:

```javascript
function HuskyCtrl($scope, stateManager) {
	stateManager.registerInitialiser(function (pathComponents) {
		//Do whatever you like here to respond to state changes: load subviews via ng-include, load content via AJAX, whatever...
		//If you load in new subviews via ng-include and if doing so causes an ng-controller directive to be compiled, that new controller's initialiser will also get called once it's loaded.
	})($scope);
}
```

You can push state as the user navigates your app. You pass in the URL split up into an array of strings:

```javascript
stateManager.pushState(["some", "other", "path"]);
```

Or if you just want to change a small part of the URL, pass `null` for the parts which should remain unchanged:

```javascript
stateManager.pushState([null, null, "edit"]);
```

Replacing state works the same way (same thing but without adding another entry in the browser history):

```javascript
stateManager.replaceState(["some", "other", "path", "which", "is", "even", "deeper"]);
```

The state manager uses the angular $location service, so you can configure it to use hashbang or HTML5 style URLs:

```javascript
angular.module("Animals", ["stateManager"])
	.config(function($locationProvider) {
		// $locationProvider.html5Mode(false).hashPrefix('!');	//turn html5 mode off
		$locationProvider.html5Mode(true);					//turn html5 mode on
	});
```

Including in your project
------------

The statemanager itself is just over 100 lines of code, and is implemented in `public/js/stateManager.js`. To use it, simply add `stateManager.js` to your project and link either directly using a `script` tag, or feed it to your build system (e.g. Closure Compiler, Yeoman, etc.).

Running demo locally
------------

Make sure you have Ruby and the following gems installs:

* Sinatra
* haml

This project comes with a `Gemfile` so if you use `Bundler` you can just do:

```sh
bundle install
```

(optional) Alternatively you can install the gems manually:

```sh
gem install sinatra
gem install haml
```

To run the demo on your computer:

```sh
ruby demo.rb
```

Dependencies
------------

The service depends Angular's own services:

* $location
* $rootScope

There are no other external dependencies for this service*. The demo uses Ruby, Sinatra and haml to bootstrap it and provide routes for the partials as well as route all other requests to the main javascript app; but the state manager itself has no dependency on this.

*The state manager does depend on at least the `onhashchange` function being implemented by the browser; this is if using it in hash fragment mode. If you use HTML5 mode, then the browser must support the history API.

Copyright & License
------------

Copyright 2013 Luka Mirosevic

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this work except in compliance with the License. You may obtain a copy of the License in the LICENSE file, or at:

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/lmirosevic/angular-statemanager/trend.png)](https://bitdeli.com/free "Bitdeli Badge")
