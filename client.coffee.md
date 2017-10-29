Client: the webpack-compiled module that runs inside the Renderer.

    {debug} = (require 'tangible') 'outstanding-volcano:renderer'
    $ = dom = require 'component-dom'

    module.exports = (y) ->
      y.on 'ready', ->
        console.log 'Main ready'

        $ 'div.main'
          . append '<p>Main ready.</p>'

      y.on 'error', (error) ->
        console.log 'Error', error

      y.emit 'client-ready'
