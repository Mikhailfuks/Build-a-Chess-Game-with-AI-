using System;
using System.Collections.Generic;

public class Program
{
    static void Main(string[] args)
    {
        ChessGame game = new ChessGame();
        game.Start();
    }
}

public class ChessGame
{
    private ChessBoard _board;
    private bool _isWhiteTurn;

    public ChessGame()
    {
        _board = new ChessBoard();
        _isWhiteTurn = true;  // White always starts
    }

    public void Start()
    {
        while (true)
        {
            Console.Clear();
            _board.Display();
            if (_isWhiteTurn)
            {
                Console.Write("White's turn. Enter move (e.g. e2 e4): ");
                string input = Console.ReadLine();
                if (!MakeMove(input))
                    continue;
            }
            else
            {
                Console.WriteLine("Black is thinking...");
                // Simple AI - Randomly selects a legal move
                MakeRandomMove();
            }
            _isWhiteTurn = !_isWhiteTurn; // Switch turns
        }
    }

    private bool MakeMove(string move)
    {
        // Parse move and make it
        // Expected format: "e2 e4"
        string[] parts = move.Split(' ');
        if (parts.Length != 2)
        {
            Console.WriteLine("Invalid move format. Try again.");
            return false;
        }

        // Extract the starting and ending positions
        Position from = new Position(parts[0]);
        Position to = new Position(parts[1]);

        // Validate and make the move
        if (_board.MovePiece(from, to, _isWhiteTurn))
            return true;

        Console.WriteLine("Invalid move. Try again.");
        return false;
    }

    private void MakeRandomMove()
    {
        Random random = new Random();
        List<Move> legalMoves = _board.GetLegalMoves(!_isWhiteTurn);
        if (legalMoves.Count > 0)
        {
            Move randomMove = legalMoves[random.Next(legalMoves.Count)];
            _board.MovePiece(randomMove.From, randomMove.To, !_isWhiteTurn);
        }
    }
}

public class ChessBoard
{
    private Piece[,] _board;

    public ChessBoard()
    {
        _board = new Piece[8, 8];
        SetupPieces();
    }

    private void SetupPieces()
    {
        // Setup Pawns
        for (int i = 0; i < 8; i++)
        {
            _board[1, i] = new Piece(PieceType.Pawn, true); // White
            _board[6, i] = new Piece(PieceType.Pawn, false); // Black
        }
        // Setup Rooks
        _board[0, 0] = _board[0, 7] = new Piece(PieceType.Rook, true);
        _board[7, 0] = _board[7, 7] = new Piece(PieceType.Rook, false);
        // Setup Knights
        _board[0, 1] = _board[0, 6] = new Piece(PieceType.Knight, true);
        _board[7, 1] = _board[7, 6] = new Piece(PieceType.Knight, false);
        // Setup Bishops
        _board[0, 2] = _board[0, 5] = new Piece(PieceType.Bishop, true);
        _board[7, 2] = _board[7, 5] = new Piece(PieceType.Bishop, false);
        // Setup Queens and Kings
        _board[0, 3] = new Piece(PieceType.Queen, true);
        _board[0, 4] = new Piece(PieceType.King, true);


        _board[7, 3] = new Piece(PieceType.Queen, false);
        _board[7, 4] = new Piece(PieceType.King, false);
    }

    public void Display()
    {
        for (int row = 7; row >= 0; row--)
        {
            Console.Write(row + 1 + " ");
            for (int col = 0; col < 8; col++)
            {
                Piece piece = _board[row, col];
                Console.Write(piece?.Display() ?? ". ");
            }
            Console.WriteLine();
        }
        Console.WriteLine("  a b c d e f g h");
    }

    public bool MovePiece(Position from, Position to, bool isWhite)
    {
        Piece piece = GetPiece(from);
        if (piece == null || piece.IsWhite != isWhite)
            return false; // No piece or wrong turn

        if (IsValidMove(from, to, piece))
        {
            _board[to.Row, to.Col] = piece;
            _board[from.Row, from.Col] = null;
            return true;
        }

        return false; // Invalid move
    }

    public List<Move> GetLegalMoves(bool isWhite)
    {
        List<Move> legalMoves = new();
        for (int row = 0; row < 8; row++)
        {
            for (int col = 0; col < 8; col++)
            {
                Piece piece = _board[row, col];
                if (piece != null && piece.IsWhite == isWhite)
                {
                    // Generate legal moves for this piece
                    // Note: This is a simplified version and doesn't implement movement rules
                    // You will need to expand this to check for each piece's unique movement
                    foreach (var move in GenerateMoves(new Position(row, col)))
                    {
                        if (IsValidMove(new Position(row, col), move, piece))
                            legalMoves.Add(new Move(new Position(row, col), move));
                    }
                }
            }
        }
        return legalMoves;
    }

    private bool IsValidMove(Position from, Position to, Piece piece)
    {
        // Add your movement validation logic here
        return true; // Simplified for this example
    }

    private IEnumerable<Position> GenerateMoves(Position from)
    {
        // This should return legal positions for the piece at 'from'
        List<Position> moves = new List<Position>();
        moves.Add(new Position(from.Row + 1, from.Col)); // Example move - 1 step forward
        return moves; // Simplified for this example
    }

    private Piece GetPiece(Position pos)
    {
        return _board[pos.Row, pos.Col];
    }
}

public class Piece
{
    public PieceType Type { get; }
    public bool IsWhite { get; }

    public Piece(PieceType type, bool isWhite)
    {
        Type = type;
        IsWhite = isWhite;
    }

    public string Display()
    {
        return IsWhite ? Type.ToString().Substring(0, 1).ToUpper() : Type.ToString().Substring(0, 1).ToLower();
    }
}

public enum PieceType { Pawn, Rook, Knight, Bishop, Queen, King }

public class Position
{
    public int Row { get; }
    public int Col { get; }

    public Position(string pos)
    {
        Col = pos[0] - 'a'; // Convert 'a'-'h' to 0-7
        Row = pos[1] - '1'; // Convert '1'-'8' to 0-7
    }

    public Position(int row, int col)
    {
        Row = row;
        Col = col;
    }
}

public class Move
{
    public Position From { get; }
    public Position To { get; }

    public Move(Position from, Position to)
    {
        From = from;
        To = to;
    }
}
