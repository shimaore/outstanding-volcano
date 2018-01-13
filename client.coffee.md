Client: the webpack-compiled module that runs inside the Renderer.

    {debug} = (require 'tangible') 'outstanding-volcano:client'
    $ = dom = require 'component-dom'

    module.exports = (y,electron) ->
      y.on 'ready', ->

        $ 'div.main'
          .append '<p>Looking for ports…</p>'

        y.emit 'get-available-ports'

      y.on 'no-devices', ->
        $ 'div.main'
          .empty()
          .append '<p>No devices</p>'

      device_name = null

      y.on 'digital-pins', (device,digital_pins) ->

        device_name = device

        $ 'div.main'
          .empty()
          .append """
            <table>
              <tbody>
                <tr><th>Pin</th><th>Channel</th><th>Note</th><th>Velocity</th></tr>
              </tbody>
            </table>

          """

        pin_line = (pin) ->
          """
          <tr>
            <th>#{pin}</th>
            <td><input type="number" name="pin#{pin}.channel" min="1" max="16"/></td>
            <td><input type="number" name="pin#{pin}.note" min="0" max="127"/></td>
            <td><input type="number" name="pin#{pin}.velocity" min="0" max="127"/></td>
          </tr>
          """

        $ 'div.main tbody'
          .append digital_pins.map(pin_line).join ''

        $ 'div.main'
          .on 'change', 'input', (e) ->
            input = $ e.target
            return unless m = input.name().match /^pin(\d+)\.(\S+)$/
            [pin,field] = [m[1],m[2]]
            y.emit 'set-value', device, pin, field, input.value()

      y.on 'midi-map', (device,map) ->
        debug 'midi-map', device
        fillin = ->
          $ 'div.main input'
          .each (input) ->
            return unless m = input.name().match /^pin(\d+)\.(\S+)$/
            [pin,field] = [m[1],m[2]]
            input.value map[pin]?[field] ? '0'

Wait, because `digital-pins` was probably sent _right before_ and the DOM is not ready yet.

        setTimeout fillin, 1000
        return

      y.on 'error', (error) ->
        $ 'div.main'
          .append "<p>#{error}</p>"
        debug 'Remote Error', error

      y.emit 'client-ready'
