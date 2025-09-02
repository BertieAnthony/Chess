# Chess

#Coding chess using OOP - used AI to generate the display board function. Need to improve it a bit more with a better way of typing in moves and need to add a few features such as en passant, signalling when in check, checking for checkmate, castling, turning pawns into queen's, and not being able to move pieces because of check.

#Class: Piece
# - Attributes: colour, position
# - Methods: get_valid_moves(board), move(new_position)
#Subclass: pieces - Pawn, Rook, Knight, Bishop, Queen, King

#Class Board
#Attributes: squares
#Methods: setup_board, move_pieces(start, end), is_square_under_attack, display

#Class game
#Methods: play, switch_turn, is_checkmate, get_player_input, is_stalemate, is_check

import matplotlib.pyplot as plt
import matplotlib.image as mpimg
import os

class Piece:
    def __init__(self, colour, x, y):
        self.colour = colour
        self.position = (x, y)

    def get_valid_moves(self, board):
        raise NotImplementedError

class Rook(Piece):
    def get_valid_moves(self, board):
        moves = []
        x, y = self.position
        directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]

        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            while 0 <= nx < 8 and 0 <= ny < 8:
                target = board.get_piece(nx, ny)
                if target is None:
                    moves.append((nx, ny))
                elif target.colour != self.colour:
                    moves.append((nx, ny))
                    break
                else:
                    break
                nx, ny = nx + dx, ny + dy

        return moves

#Not coded en passant yet
class Pawn(Piece):
    def get_valid_moves(self, board):
        moves = []
        x, y = self.position
        direction = 1 if self.colour == "white" else -1
        start_row = 1 if self.colour == "white" else 6

        if 0 <= y + direction < 8 and board.get_piece(x, y + direction) is None:
            moves.append((x, y + direction))

            if y == start_row and board.get_piece(x, y + 2*direction) is None:
                moves.append((x, y + 2*direction))
                
        for dx in [-1, 1]:
            nx = x + dx
            ny = y + direction
            if 0 <= nx < 8 and 0 <= ny < 8:
                target = board.get_piece(nx, ny)
                if target and target.colour != self.colour:
                    moves.append((nx, ny))

        return moves


class Bishop(Piece):
    def get_valid_moves(self, board):
        moves = []
        x, y = self.position
        directions = [(1, 1), (-1, 1), (-1, -1), (1, -1)]

        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            while 0 <= nx < 8 and 0 <= ny < 8:
                target = board.get_piece(nx, ny)
                if target is None:
                    moves.append((nx, ny))
                elif target.colour != self.colour:
                    moves.append((nx, ny))
                    break
                else:
                    break
                nx, ny = nx + dx, ny + dy
        return moves

class Knight(Piece):
    def get_valid_moves(self, board):
        moves = []
        x, y = self.position
        directions = [(1,2), (1, -2), (-1, 2), (-1, -2), (2, 1), (2, -1), (-2, 1), (-2, -1)]

        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            target = board.get_piece(nx, ny)
            if target is None:
                moves.append((nx, ny))
            elif target.colour != self.colour:
                moves.append((nx, ny))
        return moves

#Not done castling
class King(Piece):
    def get_valid_moves(self, board):
        moves = []
        x, y = self.position
        directions = [(-1, -1), (-1, 0), (-1, 1), (0, -1), (0, 1), (1, -1),  (1, 0),  (1, 1)]

        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            target = board.get_piece(nx, ny)
            if target is None:
                moves.append((nx, ny))
            elif target.colour != self.colour:
                moves.append((nx, ny))
        return moves
        
class Queen(Piece):
    def get_valid_moves(self, board):
        moves = []
        x, y = self.position
        directions = [(1, 1), (-1, 1), (-1, -1), (1, -1), (-1, 0), (1, 0), (0, -1), (0, 1)]

        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            while 0 <= nx < 8 and 0 <= ny < 8:
                target = board.get_piece(nx, ny)
                if target is None:
                    moves.append((nx, ny))
                elif target.colour != self.colour:
                    moves.append((nx, ny))
                    break
                else:
                    break
                nx, ny = nx + dx, ny + dy
        return moves

#Now onto the board

class Board:
    def __init__(self):
        self.squares = [[None for a in range(8)] for b in range(8)]

    def place_piece(self, piece, x, y):
        self.squares[y][x] = piece
        piece.position = (x, y)

    def get_piece(self, x, y):
        return self.squares[y][x]

    def move_piece(self, x1, y1, x2, y2):
        piece = self.get_piece(x1, y1)
        if piece:
            self.squares[y1][x1] = None
            self.squares[y2][x2] = piece
            piece.position = (x2, y2)

    def display(self):
        fig, ax = plt.subplots(figsize=(6, 6))
        # Draw squares
        for y in range(8):
            for x in range(8):
                color = '#f0d9b5' if (x + y) % 2 == 0 else '#b58863'
                ax.add_patch(plt.Rectangle((x, 7-y), 1, 1, color=color))

        # Draw pieces
        piece_images = {
            'white_pawn': 'white_pawn.png',
            'white_rook': 'white_rook.png',
            'white_knight': 'white_knight.png',
            'white_bishop': 'white_bishop.png',
            'white_queen': 'white_queen.png',
            'white_king': 'white_king.png',
            'black_pawn': 'black_pawn.png',
            'black_rook': 'black_rook.png',
            'black_knight': 'black_knight.png',
            'black_bishop': 'black_bishop.png',
            'black_queen': 'black_queen.png',
            'black_king': 'black_king.png',
        }

        for y in range(8):
            for x in range(8):
                piece = self.get_piece(x, y)
                if piece:
                    key = f"{piece.colour}_{piece.__class__.__name__.lower()}"
                    if key in piece_images and os.path.exists(piece_images[key]):
                        img = mpimg.imread(piece_images[key])
                        ax.imshow(img, extent=(x, x+1, 7-y, 8-y), zorder=1)

        ax.set_xlim(0, 8)
        ax.set_ylim(0, 8)
        ax.set_xticks([])
        ax.set_yticks([])
        ax.set_aspect('equal')
        plt.show()

#Now coding the game
class Game:
    def __init__(self):
        self.board = Board()
        self.turn = "white"
        self.setup_pieces()

    def setup_pieces(self):
        for x in range(8):
            self.board.place_piece(Pawn("white", x, 1), x, 1)
            self.board.place_piece(Pawn("black", x, 6), x, 6)
            
        self.board.place_piece(Rook("white", 0, 0), 0, 0)
        self.board.place_piece(Rook("white", 7, 0), 7, 0)
        self.board.place_piece(Rook("black", 0, 7), 0, 7)
        self.board.place_piece(Rook("black", 7, 7), 7, 7)

        self.board.place_piece(Knight("white", 1, 0), 1, 0)
        self.board.place_piece(Knight("white", 6, 0), 6, 0)
        self.board.place_piece(Knight("black", 1, 7), 1, 7)
        self.board.place_piece(Knight("black", 6, 7), 6, 7)

        self.board.place_piece(Bishop("white", 2, 0), 2, 0)
        self.board.place_piece(Bishop("white", 5, 0), 5, 0)
        self.board.place_piece(Bishop("black", 2, 7), 2, 7)
        self.board.place_piece(Bishop("black", 5, 7), 5, 7)

        self.board.place_piece(Queen("white", 3, 0), 3, 0)
        self.board.place_piece(Queen("black", 3, 7), 3, 7)

        self.board.place_piece(King("white", 4, 0), 4, 0)
        self.board.place_piece(King("black", 4, 7), 4, 7)

    def switch_turn(self):
        if self.turn == "white":
            self.turn = "black"
        else:
            self.turn = "white"

    def make_move(self, x1, y1, x2, y2):
        piece = self.board.get_piece(x1, y1)
        if piece is None:
            print("No piece at that square")
            return False
        if piece.colour != self.turn:
            print(f"It's {self.turn}'s turn")
            return False
        
        valid_moves = piece.get_valid_moves(self.board)
        if (x2, y2) not in valid_moves:
            print("Invalid move")
            return False

        self.board.move_piece(x1, y1, x2, y2)
        self.switch_turn()
        return True
    
    def play(self):
        while True:
            self.board.display()
            print()
            print(f"{self.turn}'s move")

            move = input("Enter move as x1 y1 x2 y2 (or 'quit'): ")
            if move == "quit":
                break

            try:
                x1, y1, x2, y2 = map(int, move.split())
                self.make_move(x1, y1, x2, y2)
            except ValueError:
                print("Invalid input, try again!")

game = Game()
game.play()
