# Redis-like In-Memory Key-Value Store

This project implements a Redis-like in-memory key-value store with a custom protocol for communication between the client and server. The implementation includes both a server and client written in Python, using `gevent` for concurrency.

---

## Features

- Custom protocol for encoding and decoding client-server communication.
- Supports common Redis-like commands: `GET`, `SET`, `DELETE`, `FLUSH`, `MGET`, and `MSET`.
- Handles multiple client connections using `gevent`'s `StreamServer`.
- Lightweight and easy to extend.

---

## How It Works

The project includes the following components:

### **ProtocolHandler**

The `ProtocolHandler` class handles:
- Encoding requests and responses using a custom protocol.
- Decoding client requests received by the server.

Supported protocol types include:
- **Simple Strings** (`+`): `+OK\r\n`
- **Errors** (`-`): `-ERROR MESSAGE\r\n`
- **Integers** (`:`): `:123\r\n`
- **Bulk Strings** (`$`): `$6\r\nfoobar\r\n`
- **Arrays** (`*`): `*2\r\n$3\r\nfoo\r\n$3\r\nbar\r\n`
- **Dictionaries** (`%`): `%2\r\n$3\r\nkey\r\n$5\r\nvalue\r\n`

### **Server**

The `Server` class implements a multithreaded server using `gevent`. It supports the following commands:
- `GET key`: Retrieve the value of a key.
- `SET key value`: Set the value of a key.
- `DELETE key`: Delete a key.
- `FLUSH`: Remove all keys.
- `MGET key1 key2 ...`: Retrieve multiple keys.
- `MSET key1 value1 key2 value2 ...`: Set multiple key-value pairs.

#### Example Command Handlers:
```python
def get(self, key):
    return self._kv.get(key)

def set(self, key, value):
    self._kv[key] = value
    return 1



Client
The Client class connects to the server and provides a convenient interface for sending commands.

Example Usage:
python
Copy code
client = Client()

# Set a key-value pair
client.set('foo', 'bar')

# Get the value for a key
print(client.get('foo'))  # Output: 'bar'

# Delete a key
client.delete('foo')

# Flush all keys
client.flush()

# Perform multi-get and multi-set
client.mset('key1', 'value1', 'key2', 'value2')
print(client.mget('key1', 'key2'))  # Output: ['value1', 'value2']


Prerequisites
Python 3.8+
gevent library
Install dependencies with:

bash
Copy code
pip install gevent
Running the Server
Start the server with:

bash
Copy code
python redis.py
The server will start listening on 127.0.0.1:31337 by default.

Using the Client
Interact with the server using the Client class:

python
Copy code
from redis import Client

client = Client()

# Set and get a value
client.set('foo', 'bar')
print(client.get('foo'))  # Output: 'bar'

# Delete a key
client.delete('foo')
Protocol Details
The custom protocol used for communication encodes requests and responses in a manner similar to the Redis protocol. Examples:

Request: SET key value
Encoded: *3\r\n$3\r\nSET\r\n$3\r\nkey\r\n$5\r\nvalue\r\n
Response: 1 (indicating success)
Encoded: :1\r\n
Extending the Server
To add new commands:

Define a handler method in the Server class.
Register the command in the get_commands() method.
Example:

python
Copy code
def incr(self, key):
    if key not in self._kv or not self._kv[key].isdigit():
        raise CommandError('Key does not exist or is not an integer')
    self._kv[key] = str(int(self._kv[key]) + 1)
    return self._kv[key]

def get_commands(self):
    return {
        'GET': self.get,
        'SET': self.set,
        'DELETE': self.delete,
        'FLUSH': self.flush,
        'MGET': self.mget,
        'MSET': self.mset,
        'INCR': self.incr,  # New command
    }
Copy code





