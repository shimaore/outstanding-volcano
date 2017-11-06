    seem = require 'seem'
    {debug,hand} = (require 'tangible') 'outstanding-volcano:midi'
    midi = require 'midi'

    NOTE_ON = 0x90
    NOTE_OFF= 0x80

    int = (x) ->
      if x?
        parseInt x
      else
        null

    module.exports = (w) ->
      output = new midi.output()

      default_channel = (int process.env.MIDI_CHANNEL) ? 10
      basekey = (int process.env.MIDI_BASEKEY) ? 64
      default_velocity = (int process.env.MIDI_VELOCITY) ? 90
      default_off_velocity = (int process.env.MIDI_OFF_VELOCITY) ? 40
      note_off = process.env.MIDI_NOTEOFF not in ['false']

      output.openVirtualPort "Firmata"

      maps = {}

      w.on 'digital-pins', (device,pins) ->
        maps[device] = {}
        for pin in pins
          maps[device][pin] ?=
            channel: default_channel
            note: basekey+pin
            velocity: default_velocity
            off_velocity: default_off_velocity
        debug 'midi-map', device, maps[device]
        w.emit 'midi-map', device, maps[device]
        return

      w.on 'set-value', (device,pin,field,value) ->
        debug 'set-value', device, pin, field, value
        pin = int pin
        value = int value
        return unless pin? and value?
        maps[device] ?= {}
        maps[device][pin] ?= {}
        maps[device][pin][field] = value

MIDI specs, see https://www.midi.org/specifications/item/table-1-summary-of-midi-message

      w.on 'changed', ({device,pin,value}) ->
        map = maps[device]?[pin]
        return unless map?
        {channel,note,velocity,off_velocity} = map
        if value
          msg = [NOTE_ON+channel-1,note,velocity]
          output.sendMessage
        else
          if note_off
            msg = [NOTE_OFF+channel-1,note,off_velocity]
        output.sendMessage msg if msg?
        return
