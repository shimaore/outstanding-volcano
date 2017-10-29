Renderer: the native JS module that binds renderer-side events with the IPC bus towards the main (Node.js) process.

    {ipcRenderer} = require 'electron'

    module.exports =

Receive events from the main process.

      on: (event,cb) ->
        ipcRenderer.on event, (e,message) ->
          console.log "on #{event}", e, message
          cb message
        console.log "Registered #{event}"

Send events to the main process.

      emit: (event,message) ->
        console.log "emit #{event}", message
        ipcRenderer.send event, message
        console.log "emitted #{event}", message
