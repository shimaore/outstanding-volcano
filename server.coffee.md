Server: the main Node.js process

    {debug} = (require 'tangible') 'outstanding-volcano:server'
    EventEmitter = require 'events'
    midi = require './midi'
    firmata = require './firmata'
    ui = require './ui'
    index = require './index'

    do ->
      debug 'Starting…'
      w = new EventEmitter
      midi w
      firmata w
      ui w
      index w
      debug 'Started.'
