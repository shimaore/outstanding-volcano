Renderer: the native JS module that binds renderer-side events with the IPC bus towards the main (Node.js) process.

    {ipcRenderer} = require 'electron'

    module.exports =

Receive events from the main process.

      on: (event,cb) ->
        ipcRenderer.on event, (e,args...) ->
          cb args...

Send events to the main process.

      emit: (event,args...) ->
        ipcRenderer.send event, args...
