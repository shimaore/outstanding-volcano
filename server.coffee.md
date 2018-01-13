Server: the main Node.js process

    {debug} = (require 'tangible') 'outstanding-volcano:server'
    EventEmitter = require 'events'
    ui = require './lib/ui'

    midi = require './midi'
    firmata = require './firmata'
    index = require './index'

    do ->
      debug 'Starting…'
      w = new EventEmitter
      midi w
      firmata w
      ui w
      index w
      debug 'Started.'
