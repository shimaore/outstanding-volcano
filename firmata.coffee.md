    seem = require 'seem'
    {debug,hand} = (require 'tangible') 'outstanding-volcano:firmata'
    Board = require 'firmata'

    sleep = (timeout) ->
      new Promise (resolve) -> setTimeout resolve, timeout

Connect to the Firmata device
-----------------------------

This is currently hard-coded for an Arduino Nano.

    module.exports = (w) ->
      w.on 'connect', (serial) ->
        connect serial, w

    connect = (serial,w) ->
      led = 13

      board = new Board serial
      board.on 'error', (error) ->
        w.emit 'error', error

      board.on 'ready', ->
        board.digitalWrite led, board.HIGH

        digital_pins = []
        for pin, number in board.pins when board.MODES.OUTPUT in pin.supportedModes and board.MODES.PULLUP in pin.supportedModes
          digital_pins.push number

        for pin in digital_pins
          board.pinMode pin, board.MODES.PULLUP
          board.reportDigitalPin pin, 1

        w.emit 'digital_pins', digital_pins
        return

      debouncing = {}
      last_value = {}
      board.on 'digital-read', ({pin,value}) ->
        # debug 'digital-read', pin, value

Ignore any changes while we are debouncing a previous change event.

        return if debouncing[pin]

Ignore any non-changes event.

        is_on = value is board.LOW
        return if is_on is last_value[pin]

Ignore any first-time events.

        last = last_value[pin]
        last_value[pin] = is_on
        return unless last?

We got a valid change event while we were not debouncing a previous event.

        # debug 'changed', pin, value
        debouncing[pin] = true
        setTimeout ->
          debouncing[pin] = false
        , 32

We debounce with prompt detection (name is from https://github.com/thomasfredericks/Bounce2/blob/master/Bounce2.cpp#L61).

        w.emit 'changed', {pin,value:is_on}
        w.emit "changed-#{pin}", is_on
        return
