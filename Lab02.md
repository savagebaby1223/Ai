# Laborator 2 — Crearea și utilizarea claselor. Construirea proiectului

Acest fișier conține interacțiunea cu asistentul AI (LLM) folosit la realizarea laboratorului: prompturile utilizate și răspunsurile generate, în ordine.

Laboratorul continuă proiectul **Tic-tac-toe** din Lab 1 și se desfășoară în ramura `lab02`.

---

## Prompt: Ce trebuie să fac la Lab 2?

> Dă-mi pas cu pas ce trebuie să fac la laboratorul 2 și explică.

### Răspuns

Pașii laboratorului:

1. Creează ramura `lab02` din `main`.
2. Instalează compilatorul (g++) și utilitarul make.
3. Scrie fișierele `.cpp` cu implementările metodelor declarate în `.hpp`.
4. Scrie `main.cpp` (punctul de intrare).
5. Compilează un fișier din linia de comandă (creează un fișier obiect).
6. Creează un `Makefile` pentru construirea automată a proiectului.
7. Actualizează `.gitignore` (ignoră `.o` și `.exe`).
8. Actualizează `README.md` cu metoda de construcție.
9. Publică pe GitHub în ramura `lab02` și fă pull request în `main`.

Ideea centrală: în Lab 1 ai scris **declarațiile** (ce există, în `.hpp`); acum scrii **implementările** (cum funcționează, în `.cpp`).

---

## Prompt: Instalarea uneltelor

> g++ nu e recunoscut. Ce fac?

### Răspuns

Compilatorul nu era instalat. Instalarea prin MSYS2 (în terminalul MSYS2 MINGW64):

```
pacman -S mingw-w64-x86_64-gcc
pacman -S make
```

Verificare:

```
g++ --version
make --version
```

Pentru Git din terminal:

```
pacman -S git
```

---

## Prompt: Crearea ramurii lab02

> Cum creez ramura lab02?

### Răspuns

În terminalul MINGW64, în directorul proiectului:

```
cd /c/Users/Ovidiu/source/repos/TicTacToe
git checkout main
git pull
git checkout -B lab02
```

- `git checkout main` — trece pe ramura principală.
- `git pull` — aduce ultimele modificări de pe GitHub.
- `git checkout -B lab02` — creează ramura `lab02` și trece pe ea.

Verificare: `git branch` trebuie să arate `* lab02`.

---

## Prompt: Scrierea fișierelor .cpp

> Dă-mi fiecare fișier .cpp și explică.

### Răspuns

Pentru fiecare `.hpp` se creează un `.cpp` cu implementarea metodelor, folosind forma `NumeClasa::numeMetoda`.

### `point.cpp`

```cpp
#include "point.hpp"
```

`Point` este o structură simplă, fără metode, deci fișierul conține doar include-ul.

### `cell.cpp`

```cpp
#include "cell.hpp"
```

`Cell` este o enumerare, fără metode, deci doar include-ul.

### `player.cpp`

```cpp
#include "player.hpp"

Player::Player() {
    _symbol = Cell::Empty;
}

Player::Player(Cell symbol) {
    _symbol = symbol;
}

Cell Player::GetSymbol() const {
    return _symbol;
}
```

Implementează constructorii (implicit și cu parametru) și getter-ul `GetSymbol` (are `const` pentru că doar citește).

### `board.cpp`

```cpp
#include "board.hpp"

Board::Board() {
    Reset();
}

void Board::Reset() {
    for (int r = 0; r < 3; r++) {
        for (int c = 0; c < 3; c++) {
            _cells[r][c] = Cell::Empty;
        }
    }
}

bool Board::PlaceSymbol(const Point& position, Cell symbol) {
    if (_cells[position.row][position.col] != Cell::Empty) {
        return false;
    }
    _cells[position.row][position.col] = symbol;
    return true;
}

Cell Board::GetCell(const Point& position) const {
    return _cells[position.row][position.col];
}

bool Board::IsFull() const {
    for (int r = 0; r < 3; r++) {
        for (int c = 0; c < 3; c++) {
            if (_cells[r][c] == Cell::Empty) {
                return false;
            }
        }
    }
    return true;
}
```

- `Reset` golește tabla (modifică → fără `const`).
- `PlaceSymbol` pune un simbol într-o căsuță liberă, returnează `false` dacă e ocupată (modifică → fără `const`).
- `GetCell` returnează conținutul unei căsuțe (doar citește → `const`).
- `IsFull` verifică dacă tabla e plină (doar citește → `const`).

### `game_engine.cpp`

```cpp
#include "game_engine.hpp"

GameEngine::GameEngine() {
    _currentPlayer = 0;
}

void GameEngine::Init() {
    _board.Reset();
    _players[0] = Player(Cell::X);
    _players[1] = Player(Cell::O);
    _currentPlayer = 0;
}

void GameEngine::Run() {
    // bucla principala a jocului va fi implementata aici
}

bool GameEngine::MakeMove(const Point& position) {
    Cell symbol = _players[_currentPlayer].GetSymbol();
    return _board.PlaceSymbol(position, symbol);
}

void GameEngine::SwitchPlayer() {
    if (_currentPlayer == 0) {
        _currentPlayer = 1;
    } else {
        _currentPlayer = 0;
    }
}

Cell GameEngine::CheckWinner() const {
    // verifica randurile
    for (int r = 0; r < 3; r++) {
        Cell first = _board.GetCell({r, 0});
        if (first != Cell::Empty &&
            first == _board.GetCell({r, 1}) &&
            first == _board.GetCell({r, 2})) {
            return first;
        }
    }
    // verifica coloanele
    for (int c = 0; c < 3; c++) {
        Cell first = _board.GetCell({0, c});
        if (first != Cell::Empty &&
            first == _board.GetCell({1, c}) &&
            first == _board.GetCell({2, c})) {
            return first;
        }
    }
    // verifica diagonala principala
    Cell d1 = _board.GetCell({0, 0});
    if (d1 != Cell::Empty &&
        d1 == _board.GetCell({1, 1}) &&
        d1 == _board.GetCell({2, 2})) {
        return d1;
    }
    // verifica diagonala secundara
    Cell d2 = _board.GetCell({0, 2});
    if (d2 != Cell::Empty &&
        d2 == _board.GetCell({1, 1}) &&
        d2 == _board.GetCell({2, 0})) {
        return d2;
    }
    // nimeni nu a castigat inca
    return Cell::Empty;
}
```

- `Init` golește tabla, creează jucătorii X și O, setează primul la rând.
- `MakeMove` pune simbolul jucătorului curent în căsuța aleasă.
- `SwitchPlayer` schimbă rândul între jucători.
- `CheckWinner` verifică rândurile, coloanele și diagonalele; returnează câștigătorul sau `Empty` (doar citește → `const`).

### `painter.cpp`

```cpp
#include "painter.hpp"
#include <iostream>

void Painter::DrawBoard(const Board& board) {
    for (int r = 0; r < 3; r++) {
        for (int c = 0; c < 3; c++) {
            Cell cell = board.GetCell({r, c});
            if (cell == Cell::X) {
                std::cout << " X ";
            } else if (cell == Cell::O) {
                std::cout << " O ";
            } else {
                std::cout << " . ";
            }
            if (c < 2) {
                std::cout << "|";
            }
        }
        std::cout << std::endl;
    }
}

void Painter::WriteMessage(const Point& position, const char* text) {
    std::cout << text << std::endl;
}

void Painter::ClearScreen() {
    for (int i = 0; i < 30; i++) {
        std::cout << std::endl;
    }
}
```

Componenta de afișare. `DrawBoard` desenează tabla, `WriteMessage` afișează un text, `ClearScreen` curăță ecranul. Folosește `<iostream>` (`std::cout`).

### `listener.cpp`

```cpp
#include "listener.hpp"
#include <iostream>

Point Listener::GetMove() {
    Point move;
    std::cout << "Introduceti randul si coloana (0-2): ";
    std::cin >> move.row >> move.col;
    return move;
}

bool Listener::WantsToQuit() {
    return false;
}
```

Componenta de intrare. `GetMove` citește rândul și coloana de la tastatură și le returnează ca `Point`. Folosește `<iostream>` (`std::cin`).

### `main.cpp` (punctul de intrare)

```cpp
#include "game_engine.hpp"

int main() {
    GameEngine engine;
    engine.Init();
    engine.Run();
    return 0;
}
```

`main` este punctul de intrare al programului: creează motorul, îl inițializează și pornește jocul.

---

## Prompt: Compilarea din linia de comandă

> Cum compilez un fișier și creez un fișier obiect?

### Răspuns

```
g++ player.cpp -o player.o -c
```

- `-c` — compilează doar, fără linkare (produce un fișier obiect `.o`).
- `-o player.o` — numele fișierului de ieșire.

Rezultat: fișierul `player.o`.

---

## Prompt: Fișierul de construcție (Makefile)

> Dă-mi Makefile-ul și explică.

### Răspuns

```makefile
all: TicTacToe

TicTacToe: point.o cell.o player.o board.o game_engine.o painter.o listener.o main.o
	g++ -o TicTacToe.exe point.o cell.o player.o board.o game_engine.o painter.o listener.o main.o

point.o:
	g++ point.cpp -o point.o -c

cell.o:
	g++ cell.cpp -o cell.o -c

player.o:
	g++ player.cpp -o player.o -c

board.o:
	g++ board.cpp -o board.o -c

game_engine.o:
	g++ game_engine.cpp -o game_engine.o -c

painter.o:
	g++ painter.cpp -o painter.o -c

listener.o:
	g++ listener.cpp -o listener.o -c

main.o:
	g++ main.cpp -o main.o -c

clean:
	rm -f *.o *.exe
```

- Fiecare regulă `nume.o:` compilează un `.cpp` într-un `.o`.
- Regula `TicTacToe` leagă toate `.o`-urile într-un executabil `TicTacToe.exe`.
- Regula `clean` șterge fișierele generate.
- Important: liniile de comandă încep cu TAB, nu cu spații.

Utilizare:

```
make          # compileaza si leaga tot in TicTacToe.exe
make clean    # sterge fisierele .o si .exe
```

---

## Prompt: .gitignore pentru fișierele generate

> Ce pun în .gitignore ca să nu urce .o și .exe?

### Răspuns

La `.gitignore` se adaugă:

```
# Fisiere obiect si executabile (Lab 2)
*.o
*.exe
```

Astfel, fișierele generate la compilare nu sunt urmărite de Git și nu ajung pe GitHub (se pot regenera oricând cu `make`).

---

## Prompt: Actualizarea README.md

> Ce adaug în README despre construcție?

### Răspuns

```markdown
## Construirea proiectului

Proiectul se compilează cu g++ și make.

Pași:

    make

Comanda `make` compilează fiecare fișier `.cpp` într-un `.o` și le leagă
într-un executabil `TicTacToe.exe`.

Pentru a șterge fișierele generate:

    make clean

Compilare manuală a unui fișier:

    g++ player.cpp -o player.o -c
```

---

## Prompt: Publicarea pe GitHub

> Cum public pe lab02?

### Răspuns

Configurarea identității Git (o singură dată):

```
git config --global user.name "savagebaby1223"
git config --global user.email "emailul-de-github"
```

Commit și push:

```
git add .
git commit -m "implementare metode si fisier de constructie"
git push --set-upstream origin lab02
```

Apoi `lab02` a fost unit în `main` printr-un pull request (base: main ← compare: lab02 → Create pull request → Merge).

---

## Concepte explicate (pentru apărare)

- **`.hpp` vs `.cpp`** — header-ul conține declarațiile (ce există), fișierul sursă conține implementările (cum funcționează).
- **`NumeClasa::numeMetoda`** — operatorul `::` leagă implementarea din `.cpp` de clasa din `.hpp`.
- **`const` la finalul metodei** — metoda doar citește obiectul (`GetCell`, `IsFull`, `GetSymbol`, `CheckWinner`); cele care modifică (`Reset`, `PlaceSymbol`, constructorii) nu au `const`.
- **`const Tip&`** — primește obiectul original (eficient, prin `&`) fără a-l modifica (sigur, prin `const`).
- **`const char*`** — un text pe care metoda doar îl citește.
- **`#include <iostream>`** — biblioteca standard pentru intrare/ieșire (`std::cout`, `std::cin`, `std::endl`).
- **Compilare vs linkare** — compilarea transformă fiecare `.cpp` într-un `.o`; linkarea combină toate `.o`-urile într-un executabil.
- **Makefile** — automatizează compilarea și linkarea; `make` construiește, `make clean` curăță.
- **`.gitignore`** — spune lui Git ce fișiere să ignore (fișierele obiect și executabile).
- **`main()`** — punctul de intrare al programului.
