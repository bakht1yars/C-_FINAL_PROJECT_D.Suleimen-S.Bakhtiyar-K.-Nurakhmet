# C-_FINAL_PROJECT_D.Suleimen-S.Bakhtiyar-K.-Nurakhmet
#include <iostream>
#include <string>
#include <vector>
#include <cstdlib>
#include <ctime>
#include <clocale>

// Типы ячеек на игровом поле
enum CellType { START, PROPERTY, TAX, CHANCE, CHEST };

// Структура для описания ячейки игрового поля
struct Cell {
    std::string name;
    CellType type;
    int cost;
    int rent;
    int amount;
    std::string owner;
};

// Структура для игрока
struct Player {
    std::string name;
    int position;
    int money;
    std::vector<std::string> properties;
};

// Функция для броска кубика
int rollDice() {
    return rand() % 6 + 1;
}

// Перемещение игрока
void movePlayer(Player& player, int steps, int boardSize) {
    int oldPosition = player.position;
    player.position = (player.position + steps) % boardSize;
    if (player.position < oldPosition) {
        player.money += 200;
        std::cout << player.name << " прошёл старт и получил 200!\n";
    }
}

// Отображение игрового поля
void displayBoard(const std::vector<Cell>& board, const std::vector<Player>& players) {
    std::cout << "\nТекущее состояние игрового поля:\n";
    for (size_t i = 0; i < board.size(); ++i) {
        std::cout << "[" << board[i].name << "]";
        for (const auto& player : players) {
            if (player.position == i) {
                std::cout << " <" << player.name << ">";
            }
        }
        std::cout << "\n";
    }
    std::cout << "\n";
}

// Обработка событий на ячейке
void handleCell(Player& player, std::vector<Cell>& board, std::vector<Player>& players) {
    Cell& cell = board[player.position];
    std::cout << player.name << " попал на клетку: " << cell.name << "\n";

    switch (cell.type) {
    case START:
        std::cout << "Это стартовая клетка.\n";
        break;

    case PROPERTY:
        if (cell.owner.empty()) {
            std::cout << "Эта собственность стоит " << cell.cost << ".\n";
            if (player.money >= cell.cost) {
                std::cout << "Купить? (y/n): ";
                char choice;
                std::cin >> choice;
                if (choice == 'y' || choice == 'Y') {
                    player.money -= cell.cost;
                    cell.owner = player.name;
                    player.properties.push_back(cell.name);
                    std::cout << player.name << " купил " << cell.name << "!\n";
                }
                else {
                    std::cout << "Собственность осталась свободной.\n";
                }
            }
            else {
                std::cout << "Недостаточно средств для покупки.\n";
            }
        }
        else if (cell.owner != player.name) {
            int randomRent = rand() % 201 + 300;
            std::cout << "Собственность принадлежит " << cell.owner
                << ". Нужно заплатить аренду " << randomRent << ".\n";
            player.money -= randomRent;
            for (auto& p : players) {
                if (p.name == cell.owner) {
                    p.money += randomRent;
                    std::cout << cell.owner << " получает " << randomRent << "!\n";
                    break;
                }
            }
        }
        else {
            std::cout << "Вы уже владеете этой собственностью.\n";
        }
        break;

    case TAX:
        std::cout << "Налог! Нужно заплатить " << cell.amount << ".\n";
        player.money -= cell.amount;
        break;

    case CHANCE:
    case CHEST: {
        int eventType = rand() % 2;
        if (eventType == 0) {
            int bonus = rand() % 101 + 50;
            std::cout << "Везёт! Получаете бонус " << bonus << ".\n";
            player.money += bonus;
        }
        else {
            int penalty = rand() % 101 + 50;
            std::cout << "Не повезло! Выпадает штраф " << penalty << ".\n";
            player.money -= penalty;
        }
        break;
    }
    }
}
int main() {
    setlocale(LC_ALL, "RU");
    srand(static_cast<unsigned int>(time(0)));

    std::vector<Cell> board = {
        {"Старт", START, 0, 0, 0, ""},
        {"Astana", PROPERTY, 60, 2, 0, ""},
        {"Community Chest", CHEST, 0, 0, 0, ""},
        {"Almaty", PROPERTY, 60, 4, 0, ""},
        {"Income Tax", TAX, 0, 0, 200, ""},
        {"Shymkent", PROPERTY, 200, 25, 0, ""},
        {"Kostanai", PROPERTY, 100, 6, 0, ""},
    };

    int numPlayers;
    std::cout << "Введите количество игроков: ";
    std::cin >> numPlayers;
    std::vector<Player> players(numPlayers);
    std::cin.ignore();

    for (int i = 0; i < numPlayers; i++) {
        std::cout << "Введите имя игрока " << i + 1 << ": ";
        std::getline(std::cin, players[i].name);
        players[i].position = 0;
        players[i].money = 1500;
    }

    int turn = 0;
    while (true) {
        Player& currentPlayer = players[turn % numPlayers];
        displayBoard(board, players);

        std::cout << "\nХод игрока " << currentPlayer.name << "\n";
        std::cin.ignore();
        std::cin.get();

        int dice = rollDice();
        movePlayer(currentPlayer, dice, board.size());
        handleCell(currentPlayer, board, players);

        if (currentPlayer.money <= 0) {
            std::cout << currentPlayer.name << " банкрот! Игра окончена.\n";
            break;
        }
        turn++;
    }
    return 0;
}
