    seem = require 'seem'
    {debug,hand} = (require 'tangible') 'outstanding-volcano:firmata'
    Board = require 'firmata'

    fs = require 'fs'
    {join} = require 'path'

    DEVICE_PATH = process.env.DEVICE_PATH ? '/dev'

    sleep = (timeout) ->
      new Promise (resolve) -> setTimeout resolve, timeout

Connect to the Firmata device
-----------------------------

This is currently hard-coded for an Arduino Nano.

    module.exports = (w) ->

      w.on 'connect', (device) ->
        connect device, w
        w.on 'error', -> connect device, w

      w.on 'get-available-ports', ->
        fs.readdir DEVICE_PATH, (error,files) ->
          debug "readdir #{DEVICE_PATH}", error
          if err?
            w.emit 'error', "readdir #{DEVICE_PATH}: #{error}"
            return
          unless files?
            w.emit 'available_ports', []
            return
          available_ports = files
            .filter (x) -> x.match /^ttyUSB/
            .map (x) -> join DEVICE_PATH, x
          debug 'available-ports', available_ports
          w.emit 'available-ports', available_ports

    boards = {}
    digital_pins = {}

    connect = (device,w) ->
      led = 13

      if device of boards
        if digital_pins[device]
          w.emit 'digital-pins', device, digital_pins[device]
        return boards[device]

      board = new Board device
      boards[device] = board

      board.on 'error', (error) ->
        w.emit 'error', "Device #{device}: #{error}"

      board.on 'ready', ->
        debug 'Board ready', device
        board.digitalWrite led, board.HIGH

        my_pins = []
        for pin, number in board.pins when board.MODES.OUTPUT in pin.supportedModes and board.MODES.PULLUP in pin.supportedModes
          my_pins.push number

        for pin in my_pins
          board.pinMode pin, board.MODES.PULLUP
          board.reportDigitalPin pin, 1

        digital_pins[device] = my_pins
        debug 'digital-pins', device, my_pins
        w.emit 'digital-pins', device, my_pins
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

        w.emit 'changed', {device,pin,value:is_on}
        return

      return board
