# AI Prompting Guide

## When to Use These Prompts

In Sprint 2, when you ask AI to write code, use these prompts to make sure the code matches your design.

---

## Prompt 1: Message Parser

**Use this when asking AI to write the message parsing code.**

You are writing a message parser for a networked game.

Here are the following requirements that must be followed:

1. Every message ends with a \n (newline)
2. Validate every message against these schemas:

**Message Types:**
- Join: msg, player_name (string), time (int)
- Idle_Lobby: msg, player_name, status ("Waiting for a player"), time
- Start: msg, player_name, player2_name, Role ("Player 1" or "Player 2"), Starting_position
- Move: msg, player_name, grid (row and column), time (int), Role, starting_position
- Error: msg, player_name, error_message, time (int)
- Game_Over: msg, match_result ("Player_name Wins!", "Draw!", "Player_name Lost!"), winner, loser, Last_position, time (int)
- Left_Lobby: msg, player_id, time (int)

**Error Handling:**
- If a JSON is invalid, then raise ValueError("Faulty_Format")
- If fields are missing, then raise ValueError("Missing_Field")
- If a type is wrong, then raise ValueError("Invalid_Type")

**Functions to Write:**
1. Parse_line(line: str) -> dict = parse 1 JSON line
2. Validate_messages(msg: dict) -> boolean = validate against schema
3. Send_message(sock, msg: dict) = send JSON with newline
4. Recieve_messages(sock) = read until \n, parse, and handle EOF

**Handle EOF:**
- When recv() returns 0 bytes (b""), close connection
- Never loop forever

**Include unit tests**

---

## Prompt 2: Game Logic

**Use this when asking AI to write the game loop and logic.**

Here are the requirements:

**Validating Moves:**
- Is this column or row in range [0, 2]?
- Is the cell available?
- Is it this player's turn?
- If a check fails, send an error
- If these are correct, then continue with the move

**Win Conditions:**
- There are 3 of X or O in a column, row, or diagonally = WIN!
- If the board is full = DRAW
- Whoever wins first, send GAME_OVER to the other player

**If the game disconnects:**
- If recv() returns 0 bytes or an exception, then send Game_Disconnected
- Close both sockets

**States:**
- INIT
- WAITING_FOR_PLAYERS
- GAME_START
- PLAYER_1_TURN
- PLAYER_2_TURN
- EVALUATE_MOVE
- INVALID_MOVE
- CHECK_WIN
- GAME_OVER
- CLEANUP

**Transitions:**
- INIT → WAITING_FOR_PLAYERS (when Player 1 sends Join)
- Idle_Lobby → GAME_START (when Player 2 sends Join)
- Start_Game → PLAYER_1_TURN (auto)
- Player_1_Move → EVALUATE_MOVE (when Player 1 sends Move)
- Player_2_Move → EVALUATE_MOVE (when Player 2 sends Move)
- Validate_Move → INVALID_MOVE (if move is bad) or CHECK_WIN (if move is good)
- Invalid_Move → PLAYER_1_TURN or PLAYER_2_TURN (same player tries again)
- Correct_Win → GAME_OVER (if win/draw) or PLAYER_1_TURN/PLAYER_2_TURN (switch turns)
- Any state → CLEANUP (if someone disconnects)

**Crashes:**
- All sockets in try/except
- Send Errors for bad inputs

**Game Grid:**
- Player 1 = X
- Player 2 = O
- 9 spots

**Include unit tests for all methods:** invalid moves, win conditions, and movements

---

## How to Use the Prompts

1. Copy the relevant prompt above
2. Paste it into your AI
3. Ask for written code
4. Review code with the checklist below
5. If something is wrong, ask the AI to fix it

---

## Code Review Checklist

After AI generates code, verify:

- [ ] Parses newline-delimited JSON
- [ ] Validates all required fields
- [ ] Rejects malformed JSON
- [ ] Handles EOF (recv returns 0 bytes)
- [ ] Catches ConnectionResetError, BrokenPipeError
- [ ] Never modifies board for invalid moves
- [ ] Sends ERROR for bad input (doesn't crash)
- [ ] State transitions match FSM exactly
- [ ] Detects win after each move
- [ ] Detects draw (board full)
- [ ] Handles disconnect (graceful and abrupt)
- [ ] Includes unit tests
- [ ] Uses logging for debugging
