# Ingredient Substitute Microservice

This microservice provides a list of substitute ingredients based on a single input string. It is designed to support another program (e.g., a Recipe Manager backend) via ZeroMQ using the REQ–REP pattern.

---

## 🔗 Communication Contract

### Requesting Data (from your backend)

Your program must send a message to the microservice using ZeroMQ in the format:

- `<ingredient>` must be a single-word or phrase describing the ingredient you want substitutes for (e.g., `"milk"`).
- All requests should be sent using `socket.send_string(...)`.

### Example (Python pseudocode):

```python
socket.send_string("substitute:milk")
```
### Connection details

- Protocol: ZeroMQ REQ–REP
- Address: `tcp://localhost:5556`
- Socket binding (on microservice): `socket.bind("tcp://*:5556")`
- Client connection (main program): `socket.connect("tcp://localhost:5556")`
- Both programs must agree on the same port (5556)


### Receving Data

The microservice will return one of the following:

- A JSON list of substitute strings if substitutes are found (e.g., `["almond milk", "soy milk", "oat milk"]`)
- A plain string `"No substitute found"` if there is no match

### Example (Python pseudocode):

```python
reply = socket.recv()

try:
    substitutes = json.loads(reply.decode())
    print("Substitutes:", substitutes)
except json.JSONDecodeError:
    print("Message:", reply.decode())
```
---

## UML Sequence Diagram

Below is a high-level representation of the communication pattern between your backend and this microservice:

```sql
+-----------------+         +------------------------------+         +---------------------------+
| Recipe Manager  |         | Ingredient Sub Microservice |         | Substitute Dictionary/API |
+-----------------+         +------------------------------+         +---------------------------+
        |                                |                                  |
        |  send_string("substitute:milk")|                                  |
        |------------------------------->|                                  |
        |                                |                                  |
        |                                |  lookup("milk")                  |  If substitute is found, send string of
        |                                |--------------------------------> |   replacement ingridients
        |                                |                                  |
        |                                |                                  |
        |                                |  return ["almond milk", ...]     |
        |                                |<-------------------------------  |
        |                                |                                  |
        |  send_json(["almond milk", "oat milk", "soy milk"])               |
        |<-------------------------------|                                  |
        |                                |                                  |

      OR if no match is found:

        |  send_string("substitute:xyz")|                                     |
        |------------------------------->|                                     
        |                                |  lookup("xyz")                     |
        |                                |----------------------------------->|  If no substitute ingridients found
        |                                |                                    |   return string of "No substitute found"
        |                                |  return null                       |
        |                                |<---------------------------------- |
        |                                |  send_string("No substitute found")|
        |<-------------------------------|                                    |

```
