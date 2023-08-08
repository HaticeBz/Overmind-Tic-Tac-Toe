// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TicTacToe {
    enum Player { None, X, O }
    
    struct Game {
        address playerX;
        address playerO;
        Player turn;
        Player[3][3] board;
    }
    
    mapping(uint256 => Game) public games;
    uint256 public totalGames;
    
    constructor() {
        totalGames = 0;
    }
    
    function createGame() external {
        require(totalGames < 100, "Maximum game limit reached");
        
        games[totalGames] = Game({
            playerX: msg.sender,
            playerO: address(0),
            turn: Player.X,
            board: createEmptyBoard()
        });
        
        totalGames++;
    }
    
    function markSpace(uint256 gameId, uint8 row, uint8 col) external {
        require(gameId < totalGames, "Invalid game ID");
        Game storage game = games[gameId];
        require(game.playerO != address(0), "Game is not ready yet");
        require(game.board[row][col] == Player.None, "Space is already marked");
        require(msg.sender == getPlayerOnTurn(game), "Not your turn");
        
        game.board[row][col] = game.turn;
        
        if (checkWin(game, game.turn)) {
            game.turn = Player.None;
        } else {
            game.turn = (game.turn == Player.X) ? Player.O : Player.X;
        }
    }
    
    function getPlayerOnTurn(Game storage game) internal view returns (address) {
        return (game.turn == Player.X) ? game.playerX : game.playerO;
    }
    
    function createEmptyBoard() internal pure returns (Player[3][3] memory) {
        return [
            [Player.None, Player.None, Player.None],
            [Player.None, Player.None, Player.None],
            [Player.None, Player.None, Player.None]
        ];
    }
    
    function checkWin(Game storage game, Player player) internal view returns (bool) {
        // Check rows, columns, and diagonals for a win
        for (uint8 i = 0; i < 3; i++) {
            if (game.board[i][0] == player && game.board[i][1] == player && game.board[i][2] == player) {
                return true; // Row win
            }
            if (game.board[0][i] == player && game.board[1][i] == player && game.board[2][i] == player) {
                return true; // Column win
            }
        }
        if (game.board[0][0] == player && game.board[1][1] == player && game.board[2][2] == player) {
            return true; // Diagonal win
        }
        if (game.board[0][2] == player && game.board[1][1] == player && game.board[2][0] == player) {
            return true; // Reverse diagonal win
        }
        return false;
    }
}
