# Game State Machine (FSM)

## State Diagram

INIT → WAITING_FOR_PLAYERS (Player 1 sends Join)

WAITING_FOR_PLAYERS → GAME_START (Player 2 sends Join)
WAITING_FOR_PLAYERS → CLEANUP (Player 1 disconnects)

GAME_START → PLAYER_1_TURN (Start game)

PLAYER_1_TURN → EVALUATE_MOVE (Player 1 sends Move)
PLAYER_1_TURN → CLEANUP (Player 1 disconnects)

PLAYER_2_TURN → EVALUATE_MOVE (Player 2 sends Move)
PLAYER_2_TURN → CLEANUP (Player 2 disconnects)

EVALUATE_MOVE → INVALID_MOVE (Move is invalid)
EVALUATE_MOVE → CHECK_WIN (Move is valid)

INVALID_MOVE → PLAYER_1_TURN (Send Error, Player 1's turn again)
INVALID_MOVE → PLAYER_2_TURN (Send Error, Player 2's turn again)

CHECK_WIN → GAME_OVER (Someone won or board full)
CHECK_WIN → PLAYER_2_TURN (Switch to Player 2)
CHECK_WIN → PLAYER_1_TURN (Switch to Player 1)

GAME_OVER → CLEANUP (Send Game_Over message)

CLEANUP → End

---

## States In Detail

### INIT
The server has started, but there are no players.

### WAITING_FOR_PLAYERS
Player 1 or 2 has joined, and we are waiting for the other!
- Send the idle player to the lobby.

### GAME_START
Both players joined and are ready to play.
- Send Start to both with their roles
- Start with Player 1's turn.

### PLAYER_1_TURN
Waiting for Player 1 to move.
- Send state that shows it's Player 1's turn
- If Player 2 sends Move: send Error "not your turn"
- If Player 1 sends Move: go to EVALUATE_MOVE

### PLAYER_2_TURN
Waiting for Player 2 to move.
- Send state update showing it's Player 2's turn
- If Player 1 sends Move: send Error "not your turn"
- If Player 2 sends Move: go to EVALUATE_MOVE

### EVALUATE_MOVE
Check if the move is valid.
- Is the cell empty? Is row/col in range [0,2]? Is it the right player's turn?
- If NO: go to INVALID_MOVE
- If YES: update grid and go to CHECK_WIN

### INVALID_MOVE
This move can't be made.
- Send an Error to the player
- Go back to their turn (they try again)
- Do NOT change the grid

### CHECK_WIN
Check if game is over.
- Did current player get three in a row?
- Is the grid full (draw)?
- If YES to either: go to GAME_OVER
- If NO: switch turns and send state update

### GAME_OVER
Game ended (win, draw, or forfeit).
- Send Game_Over to both players
- Go to CLEANUP

### CLEANUP
Close connections and clean up.
- Close both sockets
- End game

---

## Error Handling

### Invalid Move
- Out of bounds: send Error
- Cell already occupied: send Error
- Out of turn: send Error "not your turn"
- Do NOT update grid
- Player tries again

### Disconnect During Game
- Disconnection (Left_Lobby message): send Game_Over with forfeit to opponent
- Abrupt (EOF or exception): same as Disconnect

### Malformed JSON
- Send Error "Incorrect message type"
- Stay in the same state

---

## Win Conditions

Check for three in a row:
- Rows: (0,1,2), (3,4,5), (6,7,8)
- Columns: (0,3,6), (1,4,7), (2,5,8)
- Diagonals: (0,4,8), (2,4,6)

Grid is 9 cells: "........." (empty)
- . = empty
- X = Player 1
- O = Player 2

If all 9 cells are full with no winner: DRAW
