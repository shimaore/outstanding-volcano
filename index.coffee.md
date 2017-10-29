Index: the Node.js module that handles interaction between the firmata/midi modules and the UI.

    {debug} = (require 'tangible') 'outstanding-volcano'

    module.exports = main = (w) ->
      w.on 'ready', (mainWindow,y) ->
        serial = process.env.FIRMATA_SERIAL ? '/dev/ttyUSB0'

        w.on 'digital_pins', (digital_pins) ->

        w.on 'error', (error) ->
          debug.dev 'on error → rendered', error
          y.emit 'error', error

Start Firmata

        y.on 'connect', (serial) ->
          w.emit 'connect', serial

        y.on 'client-ready', (message) ->
          debug.dev 'Client Ready', message
          y.emit 'ready'

        debug 'on ready'
