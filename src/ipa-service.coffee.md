IP Access Service
=================

The service is responsible for interaction with the far-end TCP connection. It handles new connections from multiple BTS.

    module.exports = class ipAccessService

      constructor: (@server) ->

        @server.on 'error', (err) =>
          console.error err

        @server.on 'listening', =>
          console.log 'Server ready'

        @server.on 'close', =>
          console.error 'All connections closed'
          @end()

        @server.on 'connection', (c) =>
          c.pause()
          @init c
          c.resume()

Build the higher layers for the IPA connection.

      init: (connection) ->

Base handler for the IPA protocol

        ipaProtocol = require 'osmo-ipa'
        @ipa = new ipaProtocol connection

Higher-level handler for the IPAccess protocol (over IPA)

        ipAccessProtocol = (require 'osmo-ipaccess').protocol
        @ipaccess = new ipAccessProtocol @ipa

Finally, final handler for IPAccess protocol.

        ipAccessHandler = (require 'osmo-ipaccess').handler
        @ipaccess_handler = new ipAccessHandler @ipaccess

Higher-level handler for SCCP (over IPA)

        sccpProtocol = (require 'osmo-itu').sccp
        @sccp = new sccpProtocol @ipa
        # TODO: bing higher layers that rely on @sccp

      end: ->
        delete @ipaccess_parser
        delete @sccp_parser
