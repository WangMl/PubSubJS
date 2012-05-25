# PubSubJS

PubSubJS is a dependency free library for doing [publish/subscribe](http://en.wikipedia.org/wiki/Publish/subscribe)
messaging in [JavaScript](https://developer.mozilla.org/en/JavaScript).

In order to not have surprising behaviour where the execution chain generates more than one message, 
publication of messages with PubSub are done asyncronously (this also helps keep your code responsive, by 
dividing work into smaller chunks, allowing the event loop to do it's business).

If you're feeling adventurous, you can also use syncronous message publication (speedup in browsers), which can lead 
to some very confusing conditions, when one message triggers publication of another message in the same execution chain.
Don't say I didn't warn you.

## Goals

* No dependencies
* No modification of subscribers (jQuery custom events modify subscribers)
* No use of DOM for exchanging messages
* No reliance of running in a browser
* Easy to understand (messages are async by default)
* Small(ish)
* Compatible! ES3 compatible, should be able to run everywhere that can execute JavaScript
* AMD / CommonJS module

## Examples

### Basic example

    // create a function to receive the message
    var mySubscriber = function( msg, data ){
        console.log( msg, data );
    };

    // add the function to the list of subscribers to a particular message
    // we're keeping the returned token, in order to be able to unsubscribe 
    // from the message later on
    var token = PubSub.subscribe( 'MY MESSAGE', mySubscriber );

    // publish a message asyncronously
    PubSub.publish( 'MY MESSAGE', 'hello world!' );
    
    // publish a message syncronously, which is faster by orders of magnitude,
    // but will get confusing when one message triggers new messages in the 
    // same execution chain
    // USE WITH CAUTION, HERE BE DRAGONS!!!
    PubSub.publishSync( 'MY MESSAGE', 'hello world!' );
    
    // unsubscribe from further messages, using setTimeout to allow for easy 
    // pasting of this code into an example :-)
    setTimeout(function(){
        PubSub.unsubscribe( token );
    }, 0);

### Hierarchical addressing

    // create a subscriber to receive all messages from a hierarchy of topics
    var myToplevelSubscriber = function( msg, data ){
        console.log( 'top level: ', msg, data );
    }
 
    // subscribe to all topics in the 'car' hierarchy
    PubSub.subscribe( 'car', myToplevelSubscriber );


    // create a subscriber to receive only leaf message from hierarchy op topics
    var mySpecificSubscriber = function( msg, data ){
        console.log('specific: ', msg, data );
    }

    // subscribe only to 'car.drive' topics
    PubSub.subscribe( 'car.drive', mySpecificSubscriber );


    // Publish some topics
    PubSub.publish( 'car.purchase', { name : 'my new car' } );
    PubSub.publish( 'car.drive', { speed : '14' } );
    PubSub.publish( 'car.sell', { newOwner : 'someone else' } );

    // In this scenario, myToplevelSubscriber will be called for all 
    // topics, three times in total
    // But, mySpecificSubscriber will only be called once, as it only
    // subscribes to the 'car.drive' topic

## Tips

Use "constants" for topics and not string literals. PubSubJS uses strings as topics, and will happily try to deliver 
your messages with ANY topic. So, save yourself from frustrating debugging by letting the JavaScript engine complain
when you make typos.

### Example of use of "constants"

	// BAD
	PubSub.subscribe("hello", function( msg, data ){ 
		console.log( data ) 
	});
	
	PubSub.publish("helo", "world");
	
	// BETTER
	var MY_TOPIC = "hello";
	PubSub.subscribe(MY_TOPIC, function( msg, data ){ 
		console.log( data ) 
	});
	
	PubSub.publish(MY_TOPIC, "world");
	

## Testing
The tests are done using [BusterJS](http://busterjs.org) and the excellent [Sinon.JS](http://cjohansen.no/sinon/). 

## Future of PubSubJS

* Build script to create the following wrappers
	* jQuery plugin
	* Ender.js wrapper
* Better and more extensive usage examples

## More about Publish/Subscribe

* [The Many Faces of Publish/Subscribe](http://www.cs.ru.nl/~pieter/oss/manyfaces.pdf) (PDF)
* [Addy Osmani's mini book on Patterns](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#observerpatternjavascript)

## Versioning

PubSubJS uses [Semantic Versioning](http://semver.org/) for predictable versioning.

## Changelog

* v1.1.0
    * Hierarchical addressing of topics ("namespacing") (@jgauffin, @fideloper)
* v1.0.3
	* AMD / CommonJS module support (@fernandogmar)
	
## License

MIT: http://mrgnrdrck.mit-license.org

## Alternatives

These are a few alternative projects that also implement topic based publish subscribe in JavaScript.

* http://www.joezimjs.com/projects/publish-subscribe-jquery-plugin/
* http://amplifyjs.com/api/pubsub/