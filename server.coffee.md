    EventEmitter = require 'events'
    midi = require './midi'
    firmata = require './firmata'

    do ->
      w = new EventEmitter
      midi w
      firmata w
