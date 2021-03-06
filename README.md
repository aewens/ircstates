# ircstates

[![Build Status](https://travis-ci.org/jesopo/ircstates.svg?branch=master)](https://travis-ci.org/jesopo/ircstates)

## rationale

I wanted a bare-bones reference implementation of taking byte input, parsing it
into tokens and then managing an IRC client session state from it.

with this library, you can have client session state managed for you and put
additional arbitrary functionality on top of it.


## usage

### socket to state
```python
import ircstates, irctokens, socket

NICK = "nickname"
CHAN = "#chan"
HOST = "127.0.0.1"
POST = 6667

server = ircstates.Server("freenode")
sock   = socket.socket()

sock.connect((HOST, POST))

def _send(s):
    line = irctokens.tokenise(s)
    server.send(line)

_send("USER test 0 * :test")
_send("NICK test321")

while True:
    while server.pending():
        send_lines = server.sent(sock.send(server.pending()))
        for line in send_lines:
            print(f"> {line.format()}")

    recv_lines = server.recv(sock.recv(1024))
    for line in recv_lines:
        print(f"< {line.format()}")

        # user defined behaviors...
        if line.command == "001" and not "#test321" in server.channels:
            _send("JOIN #test321")
```

### get a user's channels
```python
>>> server.users
{'nickname': User(nickname='nickname')}
>>> user = server.users["nickname"]
>>> user
User(nickname='nickname')
>>> server.user_channels[user]
{Channel(name='#chan')}
```

### get a channel's users
```python
>>> server.channels
{'#chan': Channel(name='#chan')}
>>> channel = server.channels["#chan"]
>>> channel
Channel(name='#chan')
>>> server.channel_users[channel]
{User(nickname='nickname'): ChannelUser(user='nickname', channel='#chan', modes='ov')}
```

### get a user's modes in channel
```python
>>> user = server.users["nickname"]
>>> channel = server.channels["#chan"]
>>> channel_user = server.channel_users[channel][user]
>>> channel_user
ChannelUser(user='nickname', channel='#chan', modes='ov')
>>> channel_user.modes
{'o', 'v'}
```
