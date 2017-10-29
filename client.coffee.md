Client: the webpack-compiled module that runs inside the Renderer.

    {debug} = (require 'tangible') 'outstanding-volcano:renderer'
    $ = dom = require 'component-dom'

    module.exports = (y,electron) ->
      y.on 'ready', ->

        $ 'div.main'
          .append '<p>Looking for ports…</p>'

        y.emit 'get-available-ports'

      y.on 'digital-pins', (device,digital_pins) ->
        $ 'div.main'
          .empty()
          .append "<h2>Device #{device}</h2>"

        pin_line = (pin) ->
          """
          <p>Pin #{pin} →
            Channel <input type="number" name="pin#{pin}_channel" width="2"/>,
            Note    <input type="number" name="note#{pin}_note" width="2"/>
          </p>
          """

        $ 'div.main'
          .append digital_pins.map(pin_line).join ''

      y.on 'error', (error) ->
        console.log 'Remote Error', error
        alert error

      y.emit 'client-ready'
