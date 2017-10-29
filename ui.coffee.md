UI: the Node.js (main-side) process that starts the electron UI and binds the event bus to the renderer.

Module to control application life.
Module to create native browser window.

    {app,BrowserWindow,ipcMain} = electron = require 'electron'
    {debug} = (require 'tangible') 'outstanding-volcano:ui'

    path = require 'path'
    url = require 'url'

    module.exports = gui = (w) ->

Keep a global reference of the window object, if you don't, the window will
be closed automatically when the JavaScript object is garbage collected.

      mainWindow = null

      createWindow = ->

Create the browser window.

        mainWindow = new BrowserWindow width: 800, height: 600

and load the index.html of the app.

        index = url.format
          pathname: path.join __dirname, 'index.html'
          protocol: 'file:'
          slashes: true

        debug 'index', index

        mainWindow.loadURL index

Open the DevTools.

        mainWindow.webContents.openDevTools()

Emitted when the window is closed.

        mainWindow.on 'closed', ->

Dereference the window object, usually you would store windows
in an array if your app supports multi windows, this is the time
when you should delete the corresponding element.

          mainWindow = null
          w.emit 'closed'

Receive events from renderers on the `events` bus, and forward them onto the `w` bus:

        y =
          on: (event,cb) ->
            ipcMain.on event, (e,message) ->
              debug "renderer → on #{event}", e, message
              cb message
          emit: (event,message) ->
            debug "emit #{event} → renderer", message
            mainWindow.webContents.send event, message
            debug "emitted #{event} → renderer", message

        w.emit 'ready', mainWindow, y
        return

This method will be called when Electron has finished
initialization and is ready to create browser windows.
Some APIs can only be used after this event occurs.

      app.on 'ready', createWindow

Quit when all windows are closed.

      app.on 'window-all-closed', ->

On OS X it is common for applications and their menu bar
to stay active until the user quits explicitly with Cmd + Q

        if process.platform isnt 'darwin'
          app.quit()

        return

      app.on 'activate', ->

On OS X it's common to re-create a window in the app when the
dock icon is clicked and there are no other windows open.

        if not mainWindow?
          createWindow()

        return

      return
