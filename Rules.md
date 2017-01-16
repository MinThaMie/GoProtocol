# Rules
This files describes the rules that should be implemented by the server.

1. The board of configurable square size is empty at the start of the game
2. Black makes the first move, after which White and Black alternate
3. A move consists of placing one stone of one's own color on an empty inter-section on the board.
4. A player may pass their turn at any time.
5. A stone or solidly connected group of stones of one color is captured and re-moved from the board when all the intersections directly adjacent to it are occupied by the enemy.
6. Self-capture/suicide is allowed (but pretty useless)
7. When a suicide move results incapturing a group,theg roup is removed from the board first and the suiciding stone stays alive.
8. No stone may be played so as to recreate any previous board position (ko rule).
9. Two consecutive passes end the game. However, since black begins, white must end the game.
10. A player's territory consists of all the points the player has either occupied orsurrounded.
11. The player with more territory wins.
