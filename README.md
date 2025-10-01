# wwwww
1 import os
     2 import sys
     3 import termios
     4 import tty
     5
     6 def getch():
     7     """Reads a single character from stdin without needing Enter."""
     8     fd = sys.stdin.fileno()
     9     old_settings = termios.tcgetattr(fd)
    10     try:
    11         tty.setraw(sys.stdin.fileno())
    12         ch = sys.stdin.read(1)
    13     finally:
    14         termios.tcsetattr(fd, termios.TCSADRAIN, old_settings)
    15     return ch
    16
    17 def clear_screen():
    18     """Clears the console screen."""
    19     os.system('cls' if os.name == 'nt' else 'clear')
    20
    21 def print_game_state(maze, steps_left, message=""):
    22     """Prints the current game state (map, steps, and message)."""
    23     clear_screen()
    24     print("--- 限步迷宮 (Limited Step Maze) ---")
    25     print(f"Steps Left: {steps_left}")
    26     print("-" * 20)
    27     for row in maze:
    28         print("  ".join(row))
    29     print("-" * 20)
    30     print(f"Controls: W(Up), A(Left), S(Down), D(Right), Q(Quit)")
    31     if message:
    32         print(f"\nMessage: {message}")
    33
    34 def find_char_position(maze, char):
    35     """Finds the position of a specified character on the map."""
    36     for r, row in enumerate(maze):
    37         for c, cell in enumerate(row):
    38             if cell == char:
    39                 return r, c
    40     return None, None
    41
    42 def main():
    43     """Main game function."""
    44     # --- Game Setup ---
    45     maze = [
    46         list("P.#.."),
    47         list(".#.#."),
    48         list(".#..#"),
    49         list("...#E"),
    50     ]
    51     steps_left = 15
    52
    53     original_maze = [row[:] for row in maze] # Keep a copy to place the '.' path correctly
    54     player_r, player_c = find_char_position(maze, 'P')
    55     exit_r, exit_c = find_char_position(maze, 'E')
    56
    57     if player_r is None or exit_r is None:
    58         print("Error: Maze must contain 'P' and 'E'.")
    59         return
    60
    61     message = "Find the exit (E) before you run out of steps!"
    62
    63     # --- Main Game Loop ---
    64     while True:
    65         print_game_state(maze, steps_left, message)
    66         message = "" # Clear message after displaying it once
    67
    68         # Check for win condition
    69         if (player_r, player_c) == (exit_r, exit_c):
    70             print("\nCongratulations! You reached the exit!")
    71             break
    72
    73         # Check for loss condition
    74         if steps_left <= 0:
    75             print("\nGame Over! You ran out of steps.")
    76             break
    77
    78         # Get player input
    79         move = getch().upper()
    80
    81         if move == 'Q':
    82             print("\nQuitting game.")
    83             break
    84
    85         if move not in ['W', 'A', 'S', 'D']:
    86             message = "Invalid input! Please use W, A, S, D."
    87             continue
    88
    89         # Calculate new position
    90         new_r, new_c = player_r, player_c
    91         if move == 'W':
    92             new_r -= 1
    93         elif move == 'S':
    94             new_r += 1
    95         elif move == 'A':
    96             new_c -= 1
    97         elif move == 'D':
    98             new_c += 1
    99
   100         steps_left -= 1 # A move is attempted, consume a step
   101
   102         # Check if the new position is valid
   103         if not (0 <= new_r < len(maze) and 0 <= new_c < len(maze[0])):
   104             message = "You hit the outer wall! You lose a step."
   105         elif maze[new_r][new_c] == '#':
   106             message = "You hit a wall! You lose a step."
   107         else:
   108             # Update player position
   109             # Restore the original character at the old position
   110             maze[player_r][player_c] = original_maze[player_r][player_c]
   111
   112             player_r, player_c = new_r, new_c
   113             maze[player_r][player_c] = 'P'
   114             message = f"You moved. {steps_left} steps left."
   115
   116 if __name__ == "__main__":
   117     # Note: The getch() function for direct key presses works on Linux/macOS.
   118     # On Windows, you might need a different library like `msvcrt`.
   119     # This version is simplified for broader terminal compatibility.
   120     try:
   121         main()
   122     except (KeyboardInterrupt, SystemExit):
   123         print("\nGame exited.")
   124     except Exception as e:
   125         # Restore terminal settings if something goes wrong
   126         # This is important for the getch function
   127         import termios, sys
   128         fd = sys.stdin.fileno()
   129         termios.tcsetattr(fd, termios.TCSADRAIN, termios.tcgetattr(fd))
   130         print(f"\nAn error occurred: {e}") 
