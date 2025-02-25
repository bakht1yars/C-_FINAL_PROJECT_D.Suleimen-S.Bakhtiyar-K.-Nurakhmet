# C-_FINAL_PROJECT_D.Suleimen-S.Bakhtiyar-K.-Nurakhmet
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <cstdlib>
#include <ctime>
#include <map>
#include <algorithm>

using namespace std;

// Типы клеток
enum CellType { START, PROPERTY, TAX, CHANCE, CHEST, JAIL, BANK };

struct Cell {
    string name;
    CellType type;
    int cost;
    int rent;
    string owner;
};

struct Player {
    string name;
    int position = 0;
    int money = 1500;
    vector<string> properties;
    bool inJail = false;
    bool bankrupt = false;
};

vector<Cell> board = {
    {"Старт", START, 0, 0, ""},
    {"Astana", PROPERTY, 60, 2, ""},
    {"Шанс", CHANCE, 0, 0, ""},
    {"Almaty", PROPERTY, 100, 6, ""},
    {"Налог", TAX, 0, 200, ""},
    {"Shymkent", PROPERTY, 200, 25, ""},
    {"Тюрьма", JAIL, 0, 0, ""},
    {"Kostanai", PROPERTY, 120, 8, ""},
    {"Казна", CHEST, 0, 0, ""},
    {"Банк", BANK, 0, 0, ""}
};

void displayBoard(const vector<Cell>& board, const vector<Player>& players) {
    cout << "\nТекущее состояние игрового поля:\n";
    cout << " ------------------------------------- \n";
    cout << "| " << board[0].name << " | " << board[1].name << " | " << board[2].name << " | " << board[3].name << " |\n";
    cout << "|-------------------------------------|\n";
    cout << "| " << board[7].name << " |             | " << board[4].name << " |\n";
    cout << "|-------------------------------------|\n";
    cout << "| " << board[6].name << " | " << board[5].name << " | " << board[8].name << " | " << board[9].name << " |\n";
    cout << " ------------------------------------- \n";
    
    cout << "\nИгроки на поле:\n";
    for (const auto& player : players) {
        if (!player.bankrupt) {
            cout << player.name << " находится на " << board[player.position].name << "\n";
        }
    }
    
    cout << "\nБаланс игроков:\n";
    for (const auto& player : players) {
        if (!player.bankrupt)
            cout << player.name << ": " << player.money << " монет\n";
    }
    cout << "\n";
}

void saveGame(const vector<Player>& players) {
    ofstream file("savegame.txt");
    for (const auto& player : players) {
        file << player.name << " " << player.money << " " << player.position << " " << player.inJail << " " << player.bankrupt << "\n";
    }
    file.close();
    cout << "Игра сохранена.\n";
}

int rollDice() {
    return rand() % 6 + 1;
}

void movePlayer(Player& player) {
    int dice = rollDice();
    player.position = (player.position + dice) % board.size();
    cout << player.name << " бросает кубик и перемещается на " << board[player.position].name << "\n";
}

void handleCell(Player& player, vector<Player>& players) {
    Cell& cell = board[player.position];
    if (cell.type == PROPERTY) {
        if (cell.owner.empty()) {
            cout << "Хотите купить " << cell.name << " за " << cell.cost << "? (y/n): ";
            char choice;
            cin >> choice;
            if (choice == 'y') {
                player.money -= cell.cost;
                cell.owner = player.name;
                player.properties.push_back(cell.name);
                cout << player.name << " купил " << cell.name << "!\n";
            }
        } else if (cell.owner != player.name) {
            int rent = cell.rent;
            player.money -= rent;
            for (auto& p : players) {
                if (p.name == cell.owner) {
                    p.money += rent;
                    cout << player.name << " заплатил " << rent << " " << cell.owner << " за аренду!\n";
                }
            }
        }
    } else if (cell.type == TAX) {
        player.money -= cell.cost;
        cout << player.name << " заплатил налог " << cell.cost << "!\n";
    }
    if (player.money < 0) {
        cout << player.name << " банкрот и выбыл из игры!\n";
        player.bankrupt = true;
    }
}

int main() {
    srand(time(0));
    int numPlayers;
    cout << "Введите количество игроков: ";
    cin >> numPlayers;
    vector<Player> players(numPlayers);
    
    for (int i = 0; i < numPlayers; i++) {
        cout << "Введите имя игрока " << i + 1 << ": ";
        cin >> players[i].name;
    }

    int activePlayers = numPlayers;
    int turn = 0;
    while (activePlayers > 1) {
        Player& currentPlayer = players[turn % numPlayers];
        if (currentPlayer.bankrupt) {
            turn++;
            continue;
        }
        
        displayBoard(board, players);
        cout << "\nНажмите Enter, чтобы бросить кубик...";
        cin.ignore();
        cin.get();
        
        movePlayer(currentPlayer);
        handleCell(currentPlayer, players);
        
        if (currentPlayer.bankrupt) {
            activePlayers--;
        }
        saveGame(players);
        turn++;
    }
    
    for (const auto& player : players) {
        if (!player.bankrupt) {
            cout << "Победитель: " << player.name << "!\n";
            break;
        }
    }
    return 0;
}
