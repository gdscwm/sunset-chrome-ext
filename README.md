# Sunset Chrome Extension

Today we're going to be building a simple Chrome extension with options to keep your display awake, allow the display to
turn off but not let the system go to sleep, or default to your system settings. We'll explore the Chrome Extensions
API, learn about callback functions, and code some JavaScript.

A quick note: this is a Chrome extension, so it won't work most other browsers. It will work on Brave (the best
browser).

## Setup

We'll be using some images and starting with some skeleton code that you'll need to have in a directory somewhere on
your system. The easiest way to do this is to clone this repository. If you have an tokens set up to clone with either
HTTP or SSH, go this route. If you have no idea what that means, no worries - just manually download the files from this
repo into a single directory.

First, make sure you have an IDE where you're going to edit your code. If you don't have one, you can download the
popular VSCode [here](https://code.visualstudio.com/).

#### Cloning

The best way to clone is with SSH. If you don't have an SSH key set up, you can use HTTP if you have a personal access
token set up. If you have neither of these and still want to try cloning, call one of us over and we'll help you set up
a token.

1. On the main page of this repository, click the green "Clone" button near the top right.
2. In the SSH tab, copy the URL it gives you.
3. Open up your shell (Terminal on Mac, Powershell on Windows, or your IDE's built-in shell) and `cd` into the directory
   you want your project to live. Where it is doesn't really matter, as long as you know where on your system you put
   it.
4. Type `git clone [url]`, where `url` is the cloning URL you copied in step 2. If you cloned with SSH, this should
   successfully create a directory `sunset-chrome-ext` with the same filestructure as in the Git repo. If you cloned
   with HTTP, it will prompt you for your Git username and a password, which should be your personal access token.

#### Manual cloning

If you don't have access keys set up, you can add our files to your system manually.

1. Create a directory called `sunset-chrome-ext`. Make sure you remember where you put it.
2. From the Git repo, download the files `background.js` and `manifest.json` into your `sunset-chrome-ext` directory.
3. In your `sunset-chrome-ext` directory, create a directory `images`. From Git, download all the image files
   into `sunset-chrome-ext/images`

## Chrome API

An API, or Application Programming Interface, is how software communicate and interact with each other. Think of an API
as a waiter at a restaurant: you look at the menu and give the waiter your order, the waiter takes it back to the
kitchen, waits for the dish to be made, then delivers it back to your table.

Chrome has a large set of APIs for extensions, of which we will be using three: power, storage, and action.

### Power

`chrome.power` allows us to override the system's power management features. <sup>4</sup> It has two methods we will
use:

- `requestKeepAwake()` takes `level` as a parameter. `level` is the degree to which power management should be disabled
  and must be part of the `Level` enum predefined by Chrome. The `Level` enum has only two values: "system", which
  prevents the system from sleeping, and "display", which prevents the display from being turned off or dimmed.

- `releaseKeepAwake()` releases a request previously made by `requestKeepAwake()`.

### Storage

`chrome.storage` allows us to store, retrieve, and track changes to user data. <sup>5</sup> There are a few different
layers of storage, but the only one we'll need to use is `local`. It has two methods we care about:

- `set()` takes as a parameter a series of JSON objects (for our purposes, the same thing a Python dictionary) and
  stores them in local storage.
- `get()` takes a key as a parameter and fetches the object associated with that key from storage, if it exists. It also
  will hold a function in its parameters, but we'll get more into that later.

### Action

`chrome.action` is used to control the extension's icon in the toolbar. <sup>6</sup> Using `action`, we'll be able to
set a default name and image for our extension in our `manifest.json`, and use two of the API's methods:

- `setIcon()` to set the extension icon.
- `onClicked.addListener()` that will tell the extension what to do when someone clicks on it.

## manifest.json

The `manifest.json` file is the only file that every extension using WebExtension APIs must contain.

Using `manifest.json`, you specify basic metadata about your extension such as the name and version, and can also
specify aspects of your extension's functionality (such as background scripts, browser actions, and API
permissions).<sup>1</sup>

We're starting with a half filled in manifest. Let's go through each key in the file (the string before each colon), see
what they're doing for us, and fill in what's missing.

# TODO expand if have time

## background.js

Code time! Enter your `background.js` file.

Let's start with defining some constants. First, we'll need a variable to act as a key to use for storing the current
state in local storage. Recall JavaScript syntax for variable definitions:

```javascript
const STATE_KEY = 'state';
```

(We're only putting this variable in all caps because it's a global constant.)

We also need to define our three possible states: display on, system on, and disabled (revert to system settings). What
kind of data structures could we use for this?

```javascript
const States =
...
    ?
```

Next, we're going to write a function to switch to a new state.

```javascript
function setState(newState) {
    // The prefix for the image we're going to use. By default it should be system settings, which is night
    let imagePrefix = 'night';
    // Define what text will appear when you hover over the icon
    let title = '';

    // Switch between possible states. 
    // For each, set the appropriate image prefix, title, and make the appropriate chrome.power API request

    // Set the current state in storage using the chrome.storage API

    // Set the new icon and title using the chrome.action API
}
```

Awesome. This one function is going to take care of most of what we want our program to do.

This is where things get a little spicy. Because we defined our extension to be a background extension (as it should be;
we don't want to take up valuable CPU time with something that doesn't need to be done immediately), we have to make it
asynchronous. If you recall previous workshops, we typically do this by defining a function with the `async` keyword,
and `await`s and `Promise`s to get a result. This time, we're instead going to use something called a **callback
function**.

A callback function is any function that is called by another function that takes the first function as a parameter.
It's typically called when some event happens from the user, like when you click on the extension.
Instead of telling the program to wait until the user clicks on the extension, which would block anything else from
being executed, we can pass a whole function as a parameter to a listener that will only call it when something happens.

Okay. So this means we need some event listeners and a function for our listeners to call when their event happens.

Let's call this function `loadSavedState`. It's going to take a function as a parameter, making whatever function gets
passed in the callback function.

```javascript
function loadSavedState(callback) {
    // Get the current state from storage
    chrome.storage.local.get(STATE_KEY, function (items) {
        let savedState = items[STATE_KEY];
        // Check that the state we stored is part of our States dictionary
        for (let key in States) {
            if (savedState == States[key]) {
                callback(savedState);
                return;
            }
        }
        // Default case is disabled
        callback(States.DISABLED);
    });
}
```

Next, let's write our listener functions. We need one that's going to launch on start, that will call `loadSavedState`.
What should be passed into `loadSavedState`'s parameters?

```javascript
chrome.runtime.onStartup.addListener(function () {
    loadSavedState(?);
});
```

Finally, we need an onClick listener to change the state of the extension when you click on it. This one is going to be
a little more complicated, because we have to switch states. We can do this by defining a new function
in `loadSavedState`'s parameters!

```javascript
chrome.action.onClicked.addListener(function () {
    loadSavedState(?);
});
```

## Running your extension

Follow [this guide](https://developer.chrome.com/docs/extensions/get-started/tutorial/hello-world#load-unpacked) to load
your extension. Happy never sleeping!

## Sources

1. https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json
2. https://www.w3schools.com/js/js_callback.asp
3. https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/storage/StorageArea/get
4. https://developer.chrome.com/docs/extensions/reference/api/power
5. https://developer.chrome.com/docs/extensions/reference/api/storage#property-local
6. https://developer.chrome.com/docs/extensions/reference/api/action
7. https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/api-samples/power