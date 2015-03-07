# WebSequence
Sequencing for WebAudio.  

### Installation

Include `src/web-worker-factory.js`, `src/web-sequence.js` and `src/web-sequence-worker.js` in your HTML document:  

	<script type="text/javascript" src="<path-to-web-sequence>/src/web-worker-factory.js"></script>
	<script type="text/javascript" src="<path-to-web-sequence>/src/web-sequence.js"></script>
	<script type="javascript/worker" src="<path-to-web-sequence>/src/web-sequence-worker.js"></script>

Notice that the `type` attribute for `web-sequence-worker.js` is set to `javascript/worker` to prevent the browser from parsing this script on load. (WebSequenceTimer uses a WebWorkerFactory instance to create a worker from the contents of this script.)  

### Usage:

##### Creating Sequences:

Create a WebSequence instance. Each sequence accepts _events_ via the `sequence.addEvent(event)` function, where `event` is a function containing arbitrary code that should be executed at a given step. The sequence steps through events in the order that they are added. If you want to skip a step, call `sequence.addEvent(event)` with an empty function or `null`.  

	// Using WebSequence to change the frequency of an oscillator node:

	window.sequence = new WebSequence();
	window.sequence.addEvent(function() { // Step 0:
		window.oscillator.frequency.value = 220.0;
	});
	window.sequence.addEvent(null); // Step 1 (skipped):
	window.sequence.addEvent(function() { // Step 2:
		window.oscillator.frequency.value = 329.63;
	});
	window.sequence.addEvent(function() { // Step 3:
		window.oscillator.frequency.value = 440.0;
	});

You can modify a sequence by replacing or removing events using `sequence.replaceEventAtPosition(event, position)` and `sequence.removeEventAtPosition(position)`:  

	// Replace the first event in the sequence:
	window.sequence.replaceEventAtPosition(function() {
		window.oscillator.frequency.value = 440.0;
	}, 0);

	// Remove the first event in the sequence, adjusting the positions of all following events:
	window.sequence.removeEventAtPosition(0);

Sequences may be safely modified during playback.  

##### Controlling Timing:

WebSequenceTimer controls the pulse of each sequence that is added to it. It runs in a [web worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) so it operates independently of the main thread. It also attempts to correct for timer drift and should therefore provide a relatively stable pulse.  

Create a WebSequenceTimer instance and call its asynchronous `init()` function, passing success and error handlers as the first and second arguments:  

	window.timer = new WebSequenceTimer();
	window.timer.init(
		function(timer) { // Success
			console.log('WebSequenceTimer is ready.');
		}, 
		function(timer, status, textStatus) { // Error
			console.log('Failed to initialize WebSequenceTimer.');
		});

You may add sequences at any time by calling `timer.addSequence(sequence)` where `sequence` is an instance of WebSequence. The interval between pulses can be set by calling `timer.updateInterval(interval)` where `interval` is time in milliseconds, or `timer.updateIntervalWithTempo(tempo, duration)` where `tempo` is time in beats per minute and `duration` is the note length. Note length is usually expressed as a fraction, like `1/4` for a quarter note or `1/8` for an eighth note.  

	window.timer = new WebSequenceTimer();
	window.timer.init(
		function(timer) {
			timer.addSequence(window.sequence);
			// One WebSequenceTimer can control multiple sequences:
			// timer.addSequence(window.sequence2);
			timer.updateIntervalWithTempo(120.0, 1/8);
		}, 
		function(timer, status, textStatus) {
			console.log('Failed to initialize WebSequenceTimer.');
		});

You can also remove a sequence from the timer at any time:  

	window.timer.removeSequence(window.sequence2);

##### Controlling Playback:

Start the timer and play the sequence:  

	window.timer.start();
	window.sequence.play();

Pause the sequence at its current position:  

	window.sequence.pause();

Stop the sequence and timer:  
	
	window.sequence.stop();
	window.timer.stop();

You can also reset a sequence, which stops it and removes all events.  

	window.sequence.reset();

### License:
WebSequence is made available under the MIT License. See LICENSE.txt for details.  

### Credits:
WebSequence was created by [Tony Wallace](http://tonywallace.ca).  

