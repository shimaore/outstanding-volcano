Index: the Node.js module that handles interaction between the firmata/midi modules and the UI.

    {debug} = (require 'tangible') 'outstanding-volcano'
    {Menu,MenuItem} = require 'electron'

    module.exports = main = (w) ->
      w.on 'ready', (mainWindow,y) ->

        menu = new Menu()

        devices_menu = new Menu()
        menu.append new MenuItem label: 'Devices', submenu: devices_menu
        menu.append new MenuItem label: 'Rescan', click: -> w.emit 'get-available-ports'

        Menu.setApplicationMenu menu

        w.on 'error', (error) ->
          debug.dev 'on error → rendered', error
          y.emit 'error', "#{error}"

Start Firmata

        y.on 'connect', (serial) ->
          w.emit 'connect', serial

        y.on 'client-ready', (message) ->
          debug.dev 'Client Ready', message
          y.emit 'ready'

        y.on 'get-available-ports', -> w.emit 'get-available-ports'
        w.on 'available-ports', (devices) ->
          devices.forEach (device) ->
            devices_menu.append new MenuItem
              label: device
              click: -> w.emit 'connect', device
            w.emit 'connect', device
          if not devices.length
            y.emit 'no-devices'

        w.on 'digital-pins', (device,digital_pins) ->
          y.emit 'digital-pins', device, digital_pins
        w.on 'midi-map', (device,map) ->
          y.emit 'midi-map', device, map
        y.on 'set-value', (device,pin,field,value) ->
          w.emit 'set-value', device, pin, field, value

        debug 'on ready'
