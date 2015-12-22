
# MockEvent

MockEvent simulates [Server Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events).

Useful if you want to build the client side portion without having the server side portion available to you. Also useful when writing unit tests.

### Install

1. Install 1 dependency: [underscore](https://github.com/jashkenas/underscore/releases/tag/1.8.3) by [Jeremy Ashkenas](https://github.com/jashkenas).
2. Then download the mockevent.js in the root directory.
3. Place them in the same directory and follow the examples.

### Examples

All examples include which libraries to include. How to instantiate a `MockEvent` which streams the data and how to instantiate an `EventSource` which consumes the data.


#### Example 0

Getting started with the least amount of work.
```javascript

// In the head of your HTML
<script src='static/underscore-1.8.3.min.js'></script>
<script src='static/mockevent.js'></script>

<script>
// Instantiate a `MockeEvent` ----------
MockEvent({
    url: '/tweets',
    responses: [
        {name: 'tweet', data: 'a tweet'},
        {name: 'tweet', data: 'another tweet'}
    ]
});

// Instantiating an `EventSource` ----------
var evtSource = new EventSource('/tweets');

// Listening to specific event names and handling them
evtSource.addEventListener('tweet', function(e){ console.log(e)}, false);
</script>
```


#### Example 1

How to create a basic MockEvent aka simulate Server Sent Events.
```javascript

// In the head of your HTML
<script src='static/underscore-1.8.3.min.js'></script>
<script src='static/mockevent.js'></script>

<script>
// Instantiate a `MockeEvent` ----------
var mockEvent = new MockEvent({
    url: '/tweets',
    setInterval: 100,
    responses: [
        {name: 'first_tweet', data: 'first tweet'},
        {name: 'tweet', data: 'a tweet'},
        {name: 'tweet', data: 'another tweet'},
        {name: 'tweet', data: 'fourth tweet'},
        {name: 'last_tweet', data: 'last tweet'}
    ],
});

// Instantiate an `EventSource` ----------
var evtSource = new EventSource('/tweets');

// Listening to specific event names and handling them
evtSource.addEventListener('first_tweet', function(e){ console.log('first tweet', e)}, false);
evtSource.addEventListener('tweet', function(e){ console.log('tweet', e)}, false);
evtSource.addEventListener('last_tweet', function(e){ console.log('last tweet', e)}, false);

// Listening for open and error events
evtSource.onopen = function(e) {console.log('TWEET CONNECTION IS OPEN')}
evtSource.onerror = function(e){log('TWEET CONNECTION GO BOOM', e.message)};
</script>
```

* `url`: The relative URL for your Server Sent Event API.  This is the URL we will subscribe to via `EventSource`.
* `setInterval`: Miliseconds to wait before sending the next event/response.
* `responses`: A list of event/responses to send and the order in which to send them.
    - `name`: Server Sent Events can have a name that you directly subscribe to in case you want to handle that name differently. In the above example we require you to subscribe to 3 different names `first_tweet`, `tweet`, and `last_tweet`.  Making it easier to provide your logic on the client side.  Maybe you want to do some initializing when the first tweet comes in so you put that logic in the that handler.  Maybe you want to tear down some logic or disconnect for the URL on the last tweet. The point is that you can make and subscribe to as many names as you want to make your logic easier to write and undertand.
    - `data`: The data you want to send, in this case we're just sending clear text, but you could also send a hash type or which ever data type you want to send.


#### Example 2

Dynamically make responses and then stream them.
```javascript

// In the head of your HTML
<script src='static/underscore-1.8.3.min.js'></script>
<script src='static/mockevent.js'></script>

<script>
// Instantiate a `MockeEvent` ----------
var mockEvent = new MockEvent({
    url: '/tweets',
    /* If you would like to somehow customize the responses
    dynamically, this is one way. */
    setInterval: [100, 1000],
    response: function(self, evtSource){
        /* If you would like to somehow customize the responses
        dynamically, this is one way. */
        var data = [
            {name: 'tweet', data: 'a tweet'},
            {name: 'tweet', data: 'another tweet'}
        ];

        /* In this case we have static data, but you can build that data
        however you choose and then stream it when you're ready. */
        self.stream(data);
    }
});

// Instantiate an `EventSource` ----------
var evtSource = new EventSource('/tweets');

// Listening to specific event name and handle
evtSource.addEventListener('tweet', function(e){ console.log('tweet', e)}, false);

// Listening for open and error events
evtSource.onerror = function(e){log('TWEET CONNECTION GO BOOM', e.message)};
</script>
```

This uses the `response` attribute instead of the plural `responses` method.  Here you can build build a list of responses to send and then stream them when you're ready.

The `stream` method respects the `setInterval` property you specified. Also notice that `setInterval` can be set to an `int`, `float`, or `array` representing a min/max range of milliseconds.

Technically you can use any method and then call `this.stream`. The benefit of using the `response` hook is that you get the `mockEvent` and `evtSource` object in case you need those values for anything.  Example 3 shows how you can benefit from these objects.

#### Example 3

Use the `MockEvent` and `EventSource` object when creating your responses.
```javascript

// In the head of your HTML
<script src='static/underscore-1.8.3.min.js'></script>
<script src='static/mockevent.js'></script>

<script>
// Instantiate a `MockeEvent` ----------
var mockEvent = new MockEvent({
    url: '/tweets',
    setInterval: 100,
    response: function(self, evtSource){
        /* If you would like to somehow customize the responses
        dynamically, this is one way. */
        var data = [
            {name: 'tweet', data: 'a tweet'},
            {name: 'tweet', data: 'another tweet'}
        ];

        /* Here we use the `self.send` and `self.setInterval` attributes
        from the mockEvent object. To build our own streaming method. */
        var intervalId = setInterval(function(){
            var val = data.shift() || clearInterval(intervalId);
            if(val) self.send(val);
        }, self.setInterval)
    }
});

// Instantiate an `EventSource` ----------
var evtSource = new EventSource('/tweets');

// Listening to specific event name and handle
evtSource.addEventListener('tweet', function(e){ console.log('tweet', e)}, false);

// Listening for open and error events
evtSource.onerror = function(e){log('TWEET CONNECTION GO BOOM', e.message)};
</script>
```

Here we write our own `stream` method that loops through data and sends.  Still respecting the `setInterval` attribute and leveraging the internal `send` method of the `mockEvent`.  This is not recommended, it's just used as an example to show the amount of access you have to all attributes.  Using the `send` method directly overwrites the `stream` attribute and will not respect other queues that maybe set by you later.

### Roadmap

* Unit tests: Long story short, I had them, updated code and then didn't want to prolong the publishing of this library any longer.
* Data from a file: You can set your responses in the javascript file, but reading from a JSON file would be nice.

### Special Thanks

Special thanks to [DecisioHealth](http://decisiohealth.com).  Where I work with talented developers and where they allow me to publish libraries for open source use.

Also thankful for [Jordan Kasper and his MockJax project](https://github.com/jakerella/jquery-mockjax). I used his library for mocking AJAX requests and it was a HUGE help in getting me started on this library.

### History

All we do at work is stream data and mocking Server Sent Events was essential.  I looked for a library and when I didn't find one I did what any open source developer does. Built one.