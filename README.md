# SignalK trigger plugin

This package is designed to allow other signalk plugins to receive notifications when certain values exceed a threshold.

## Installation

To install this package clone it from git and run npm link.

```
git clone https://github.com/codekilo/signalk-trigger.git
cd signalk-trigger
sudo npm link
```

Then go to the SignalK configuration directory (probably `~/.signalk`)  and link the module again:

```
$ cd .signalk 
$ npm link signalk-trigger
```

## configuration

for each trigger the following values are required:

#### condition
The condition that should trigger the event, uses [jexl](https://github.com/TomFrost/jexl) to evaluate the expression. The full signak tree for the specified vessel is available. The following expression would create an event when the vessel is going faster than 3 m/s : `navigation.speedOverGround.value > 3`

#### context
The vessel for which this condition should be checked.

#### event
The name of the event that should be triggered when the condition is true.

#### trigger type
When to fire the event, has four available options

- rising edge
    + only sends the event when the condition becomes true
- falling edge
    + only sends the event when the condition becomes false
- both edges 
    + sends the event when the condition changes
- for all deltas
    + send the event for all deltas while the conditon is true, including an event on the falling edge

## use
After setting this plugin up, other plugins can use the configured events to trigger actions by listening to them.

The following example sets up a simple listener that will write to the plugins debug log

```js 
var unsubscribes = [];
module.exports = function(app) {
  var plugin = {};

  plugin.id = 'your-plugin';
  plugin.name = 'your plugin';
  plugin.description = 'A plugin to send notifications when an event occurs';

  plugin.start = function(options, restartPlugin) {
    let _notify = function(event) {
        app.debug(`Event received: ${event.event}, type: ${event.type}`);
    }
    app.on('eventName', _notify);
    unsubscribes.push(() => {
      eventEmitter.removeListener(event, _notify);
    });

    app.setProviderStatus('Running');
  };

   plugin.stop = function() {
    unsubscribes.forEach(f => f());
    app.setProviderStatus('Stopped');
  };
  return plugin;
}
```
