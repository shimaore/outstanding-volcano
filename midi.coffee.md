    seem = require 'seem'
    {debug,hand} = (require 'tangible') 'outstanding-volcano:midi'
    midi = require 'midi'

    NOTE_ON = 0x90
    NOTE_OFF= 0x80

    int = (x) ->
      if x?
        parseInt x

    module.exports = (w) ->
      output = new midi.output()

      channel = (int process.env.MIDI_CHANNEL) ? 10
      basekey = (int process.env.MIDI_BASEKEY) ? 64
      note_off = process.env.MIDI_NOTEOFF not in ['false']

      output.openVirtualPort 'Firmata'

MIDI specs, see https://www.midi.org/specifications/item/table-1-summary-of-midi-message

      w.on 'changed', ({pin,value}) ->
        # debug 'changed', pin, value
        if value
          output.sendMessage [NOTE_ON+channel-1,basekey+pin,90]
        else
          if note_off
            output.sendMessage [NOTE_OFF+channel-1,basekey+pin,40]
        return
