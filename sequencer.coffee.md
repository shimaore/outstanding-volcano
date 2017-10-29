Sequencer: a dummy demo for a sequencer.

    seem = require 'seem'
    {debug,hand} = (require 'tangible') 'outstanding-volcano:midi'
    midi = require 'midi'

    sleep = (timeout) -> new Promise (resolve) -> setTimeout resolve, timeout

    NOTE_ON = 0x90
    NOTE_OFF= 0x80

    int = (x) ->
      if x?
        parseInt x

    do seem ->
      output = new midi.output()

      channel = (int process.env.MIDI_CHANNEL) ? 1
      basekey = (int process.env.MIDI_BASEKEY) ? 24
      maxkey  = (int process.env.MIDI_MAXKEY) ? 107

      serial = process.env.FIRMATA_SERIAL ? '/dev/ttyUSB0'
      output.openVirtualPort "Sequencer"

MIDI specs, see https://www.midi.org/specifications/item/table-1-summary-of-midi-message

      note = basekey
      direction = +1
      while true
        console.log note
        output.sendMessage [NOTE_ON+channel-1,note,90]
        yield sleep 150
        output.sendMessage [NOTE_OFF+channel-1,note,40]
        switch
          when direction > 0 and note < maxkey
            note += direction
          when direction < 0 and note > basekey
            note += direction
          else
            direction = -direction
            note += direction
      return
