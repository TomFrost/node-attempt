# Attempt
Retries functions that throw or call back with an error, in crazily
customizable ways.

[![Build Status](https://secure.travis-ci.org/TomFrost/node-attempt.png?branch=master)](http://travis-ci.org/TomFrost/node-attempt)

## Installation
In your project folder, type:

	npm install attempt

## Usage
Whenever you have code that looks like this:

	function flakyApiCall(function(err, result) {
	    if (err)
	        console.err('Flaky API died AGAIN!');
	    else
	        doSomething(result);
	});

Just drop that code in an attempt!

	var attempt = require('attempt');
	attempt(function() {
		flakyApiCall(this);
	},
	function(err, result) {
        if (err)
            console.err('Flaky API failed 5 times.');
        else
            doSomething(result);
    });

## Details
Attempt will re-run your attempted function if it throws an error or if it
calls back with a non-empty first argument (following the first-arg-is-an-error
standard Node.js convention).  The function call looks like this:

**attempt(tryFunc, _[options]_, _[callback]_)**

tryFunc is called with one argument: attempts.  It is the number of times the
tryFunc has been run before.

	attempt(function(attempts) {
		if (attempts)
			console.log('This is retry #' + attempts);
		else
			console.log('This is the first attempt!');
	});

### Options
The following options are set per request:

#### retries
*Default: 5.* The number of times to retry the tryFunc before
giving up and sending the error to the callback.

	attempt(function() {
		flakyApiCall(this);
	}, { retries: 15 }, function(err, result) {
		if (err)
			console.err('Failed 16 times.');
		else
			doSomething(result);
	});

#### interval
*Default: 0.* The number of milliseconds to wait between attempts.

	attempt(function() {
        flakyApiCall(this);
    }, { interval: 5000 }, function(err, result) {
        if (err)
            console.err('5 retries * 5 seconds = 25 seconds of failure.');
        else
            doSomething(result);
    });

#### factor
*Default: 1.* The factor by which the interval should be multiplied per
attempt.  If set to 2 with an interval of 5, the first retry will execute after
5 seconds, the second after 10, the third after 20, and so on.

This allows an exponential retry scheme.  For a smaller gap between retries,
floats like 1.2 can be used to grow the interval at a slower rate.

#### onError
*Default: null.* Function to call when the tryFunc fails with an
error.  The first argument is the error.

	attempt(function() { flakyApiCall(this); },
		{ onError: function(err) { console.err(err); } },
		function(err, result) { /* ... */ });

The second argument, if it exists, is a callback function that will need to be
called in order for the next attempt to continue.  This is useful if you have
to do something asynchronous to fix the error.

	attempt(function() { flakyApiCall(this); },
		{ onError: function(err, done) {
			log.write(err, done);
		} },
		function(err, result) { /* ... */ });

By default, calling done() will ignore the retry interval.  If you still want
it to be observed, call *done(true)*.

#### attempts
*Default: 0.* The number of attempts to fake Attempt into believing were
already completed.  This is mostly used by Attempt internally, but can be
useful for hacking interval times.

#### max
*Default: Infinity* the maximum timeout to delay the next attempt.

#### random
*Default: 0* random factor to scale timeout by. `(Math.random() * options.random) + 1`
setting random to 1 will scale the timeout a random amount between 1 and 2.

## License
Attempt is distributed under the MIT license.

## Credits
Attempt was created by Tom Frost in 2012.  Because webservice APIs be flaky,
man.
