# imagesLoaded

http://desandro.github.com/imagesloaded/

A jQuery plugin that triggers a callback after all the selected/child images have been loaded. Because you can't do `.load()` on cached images.

## Calling

ImagesLoaded can be called on containers with images within them, images directly, or a combination of both.

```js
$(selector).imagesLoaded( [ callback ] [, defer ] );
```

Both arguments are optional.

### [ callback ]

Callback function to be called when all images has finished with loading.

###### this

The function scope (`this`) is a jQuery object with element or set of elements on which the imagesLoaded has been called (`$(selector)`).

###### arguments

Callback function receives 3 arguments:

+ **$images:** `Object` jQuery object with all images
+ **$proper:** `Object` jQuery object with properly loaded images
+ **$broken:** `Object` jQuery object with broken images

### [ defer ]

Boolean argument to postpone the loaded/proper/broken images determination process.

When `true` is passed, returned object will be extended with `.start()` method that starts the determination process.
Useful when you want to use deferred `.progress()` method.

```js
var dfd = $(selector).imagesLoaded( true );

dfd.progress(function(){
	// do stuff on each loaded image
});

// start the determination process
dfd.start();
```

## Basic usage

```js
$('#my-container, .article img').imagesLoaded( function( $images, $proper, $broken ) {
  console.log( $images.length + ' images total have been loaded' );
  console.log( $proper.length + ' properly loaded images' );
  console.log( $broken.length + ' broken images' );
});
```

## Deferred

As of v1.2.0, `imagesLoaded` returns jQuery deferred object.


### Behaviour

**Resolved**: deferred is *resolved* when all images have been properly loaded

**Rejected**: deferred is *rejected* when at least one image is broken

**Notified**: deferred is *notified* every time an image from stack has finished loading

### Usage
```js
// Save a deferred object with postponed determination process
var dfd = $('#my-container').imagesLoaded( true );

// Always
dfd.always( function(){
  console.log( 'all images has finished with loading, do some stuff...' );
});

// Resolved
dfd.done( function( $images ){
  // callback provides one argument:
  // $images: the jQuery object with all images
  console.log( 'deferred is resolved with ' + $images.length + ' properly loaded images' );
});

// Rejected
dfd.fail( function( $images, $proper, $broken ){
  // callback provides three arguments:
  // $images: the jQuery object with all images
  // $proper: the jQuery object with properly loaded images
  // $broken: the jQuery object with broken images
  console.log( 'deferred is rejected with ' + $broken.length + ' out of ' + $images.length + ' images broken' );
});

// Notified
dfd.progress( function( isBroken, $images, $proper, $broken ){
  // function scope (this) is a jQuery object with image that has just finished loading
  // callback provides four arguments:
  // isBroken: boolean value of whether the loaded image (this) is broken
  // $images:  jQuery object with all images in set
  // $proper:  jQuery object with properly loaded images so far
  // $broken:  jQuery object with broken images so far
  console.log( 'Loading progress: ' + ( $proper.length + $broken.length ) ' out of ' + $images.length );
});

// Start determination process
dfd.start();
```

When using `.progress()` method, you have to call imagesLoaded with truthy **defer** argument, and trigger determination process with `.start()`
method manually after you've attached progress callback. That's because in some browsers (when loading a page with cached images)
imagesLoaded is so fast that by the time you'll attach a callback to a progress method, deferred object is already resolved/rejected, and there is nothing to report.

### Requirements

+ Deferred is being used only when present, so having older versions of jQuery doesn't break the plugin, just removes the functionality.
+ For using any Deferred method, you need jQuery **v1.5** and higher.
+ For using Deferred progress method, you need jQuery **v1.7** and higher.
+ For availability of other Deferred methods, read the [jQuery Deferred object documentation](http://api.jquery.com/category/deferred-object/).

## Behavior notes

### Determination process

Every browser is handling image elements differently. We are trying to check for state in image attributes when possible & reliable, but in some browsers (especially older ones)
we have to reset `img.src` attribute to re-trigger load/error events. That is the only possible way how to check for an image status,
as well as differentiate between proper and broken images. We are resetting the src with blank image data-uri to bypass webkit log warning (thx doug jones).

Fortunately, since v2.0.0 we are in a state where imagesLoaded is both cross-browser reliable and fast, with only a few minor [known issues](#known-issues).

### Caching

The state of all once checked images is cached, so the calls repeated on the same images don't have to go through a determination process for each image again.
Determining might be slow in older browsers in which we have to reset `src`, and also might introduce image flickering.

Image state is stored in `$.data` associated to that particular image DOM element. That means that everything is stored per page load,
so you don't have to worry that temporarily unavailable images will be considered as broken on a next page load as well. This is just for a multiple calls within one page load.

If, however, you need it from some reason, you can remove this data from an image with:

```js
$.removeData( img, 'imagesLoaded' );
```

## Known issues

+ Unreliable differentiation between proper and broken images in Opera versions lower than 11 (market share ~0.1%), but callback still fires.
+ In IE (particularly in older versions) you might see images flicker as plugin has to refresh all `src` attributes to catch event types.
  Thankfully, this flickering is invisible most of the time.

## Contribute

It ain't easy knowing when images have loaded. [Every browser has its own little quirks](https://github.com/desandro/imagesloaded/wiki/Browser-quirks) that make this a difficult task to develop a cross-browser solution. Pull requests, testing, [issues](https://github.com/desandro/imagesloaded/issues), and commentary are all highly encouraged (pleasepleaseplease) and very much appreciated.

### Contributors

+ [**View contributors**](https://github.com/desandro/imagesloaded/contributors)
+ [ajp](http://groups.google.com/group/jquery-dev/browse_thread/thread/eee6ab7b2da50e1f)
+ Oren Solomianik

