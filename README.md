# i3-mock-server
Mock the server/daemon behavior of the [i3 window manager](https://github.com/i3/i3).
Useful for automated testing of IPC client code.

## About
Source is heavily copied from [node-i3](https://github.com/sidorares/node-i3), an IPC
client library, and adapted to mimic IPC server behavior.
Source annotated with exact links to node-i3 code blocks for reference.

Developed for automated testing of [i3-shade](https://github.com/ctemplin/i3-shade).

## Usage Example
~~~js
const { I3MockServer,
        encodeCommand,
        commandNameFromCode,
        commandCodeFromName,
        eventCodeFromName
} = require('i3-mock-server')

const handleMessage = function(server) {
  let comCode = server._message.code
  let payload = server._message.payload?.toString()
  switch(commandNameFromCode[comCode]) {
    case 'GET_TREE':
      let response = JSON.stringify(
        {,"id": 6875648, "type": "root", "name": "root", "nodes": [...], ...}
      )
      server._stream.write(encodeCommand(comCode, response))
      break;
    case 'SUBSCRIBE':
      server._stream.write(
        encodeCommand(
          commandCodeFromName['SUBSCRIBE'],
          '[{"success": true}]'
        )
      )
      break;
    case 'COMMAND':
      if (payload.startsWith('workspace')) {
        // Write the output of workspace event
        resp = ...// load mock json response
        server._stream.write(encodeCommand(
          eventCodeFromName['workspace'],
          JSON.stringify(resp))
        )
      }
      if (payload.startsWith("focus")) {
        ...
      }
      ... // handle other COMMAND payloads
      // Respond to COMMAND with success
      server._stream.write(encodeCommand(
        commandCodeFromName['COMMAND'],
        '[{"success": true}]'
      ))
      break;
    case ... // other command type(s)
  }
}

new I3MockServer(
  require('path').join(process.cwd(), 'i3test.sock'),
  handleMessage
)
// Communicate with this server with node-i3 using the same socket path
~~~