# Protocol Blueprint

## Framing Rule

Messages are sent as JSON, one per line, ending with a newline (`\n`).

**Why:** TCP sends raw bytes without message boundaries. The newline tells the receiver "this message is complete."

**Example:**
```json
{"msg":"Join","player_name":"Saad","time":"2026-10-07"}\n
{"msg":"Move","player_name":"Saad","grid":{"row":0,"col":2},"time":"2026-10-07"}\n
```

**How to receive:** Read bytes until you see `\n`, then parse that line as JSON.

---

## Message Types

### Join (Client → Server)
Player joins the game.

```json
{
  "msg": "Join",
  "player_name": "Saad",
  "time": "2026-10-07"
}
```

### Idle_Lobby (Server → Client)
Server is waiting for the second player.

```json
{
  "msg": "Idle_Lobby",
  "player_name": "Saad",
  "status": "Waiting for a player",
  "time": "2026-10-07"
}
```

### Start (Server → Both)
Both players connected, game starts.

```json
{
  "msg": "Start",
  "player_name": "Saad",
  "player2_name": "Steve",
  "role": "Player 1",
  "starting_position": "---------",
  "time": "2026-10-07"
}
```

### Move (Client → Server)
Player makes a move.

```json
{
  "msg": "Move",
  "player_name": "Saad",
  "grid": {"row": 0, "col": 2},
  "time": "2026-10-07",
  "role": "Player 1"
}
```

### Error (Server → Client)
Tell player their move was invalid.

```json
{
  "msg": "Error",
  "player_name": "Saad",
  "error_message": "Position already occupied",
  "time": "2026-10-07"
}
```

### Game_Over (Server → Both)
Game ended.

```json
{
  "msg": "Game_Over",
  "match_result": "Player 1 Wins!",
  "winner": "Saad",
  "loser": "Steve",
  "last_position": "XXO---X--",
  "time": "2026-10-07"
}
```

### Left_Lobby (Client → Server)
Player is leaving.

```json
{
  "msg": "Left_Lobby",
  "player_id": "Saad",
  "time": "2026-10-07"
}
```

---

## Connection Termination

### Graceful (Player Quits)
1. Player sends Left_Lobby message
2. Server sends Game_Over with forfeit
3. Server closes both sockets

### Abrupt (Network Fails / Crash)
1. Server's `recv()` returns 0 bytes (EOF)
2. Server sends Game_Over to other player
3. Server closes both sockets
4. Notify other player of error

**Important:** If `recv()` returns 0 bytes, the connection is closed. Do NOT loop forever. Close the socket.

### Exceptions to Catch
- `ConnectionResetError` — second player crashed
- `BrokenPipeError` — can't send to closed socket
- `ConnectionAbortedError` — OS aborted connection

When you catch these, treat it like EOF: notify opponent and close both sockets.
