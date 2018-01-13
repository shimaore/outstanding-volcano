    module.exports =
      entry: './client.js'
      output:
        path: __dirname
        filename: 'client-bundle.js'
        library: 'Application'
        libraryTarget: 'umd'
