# Laborator 3 — Constructori și Operatori

Acest fișier conține interacțiunea cu asistentul AI (LLM) folosit la realizarea laboratorului: prompturile utilizate și răspunsurile generate, în ordine.

Laboratorul continuă proiectul **Tic-tac-toe** din laboratoarele anterioare și se desfășoară în ramura `lab03`.

---

## Prompt: Ce trebuie să fac la Lab 3?

> Astea sunt cerințele pentru lab 3, te rog să mă ajuți și să îmi explici tot pas cu pas, ca un profesor.

### Răspuns

Pentru fiecare structură de date a jocului (`Point`, `Player`, `Board`) trebuie definite:

1. constructor implicit;
2. constructor de copiere;
3. constructor cu parametri;
4. operator de atribuire (`=`);
5. operatori de comparare (`==`, `!=`);
6. operatori de intrare și ieșire (`>>`, `<<`);
7. pentru clasa de redare (`Painter`), o clasă de bază abstractă.

Scopul: pregătirea proiectului pentru biblioteca standard C++. Containerele standard (`vector`, `set`, sortări) au nevoie ca obiectele să știe să se copieze, să se compare și să se atribuie.

Lucrul se face pe o ramură nouă `lab03`, creată din `main`.

---

## Prompt: Crearea ramurii lab03

> Cum creez ramura lab03?

### Răspuns

În Visual Studio: meniul **Git → Pull** (pe `main`, pentru ultima versiune), apoi **Git → New Branch** → nume `lab03` → based on `main` → Create.

Jos-dreapta în bară trebuie să scrie `lab03`.

---

## Prompt: Dă-mi toate fișierele modificate

> Dă-mi-le pe toate deodată.

### Răspuns

Pentru fiecare structură s-au adăugat constructorii și operatorii ceruți.

### `point.hpp`

```cpp
#pragma once

#include <iostream>

struct Point {
   int row;
   int col;

   // constructor implicit
   Point();
   // constructor cu parametri
   Point(int row, int col);
   // constructor de copiere
   Point(const Point& other);
   // operator de atribuire
   Point& operator=(const Point& other);
   // operatori de comparare
   bool operator==(const Point& other) const;
   bool operator!=(const Point& other) const;
};

// operatori de intrare / iesire (functii libere, nu metode)
std::ostream& operator<<(std::ostream& out, const Point& p);
std::istream& operator>>(std::istream& in, Point& p);
```

### `point.cpp`

```cpp
#include "point.hpp"

Point::Point() {
    row = 0;
    col = 0;
}

Point::Point(int row, int col) {
    this->row = row;
    this->col = col;
}

Point::Point(const Point& other) {
    row = other.row;
    col = other.col;
}

Point& Point::operator=(const Point& other) {
    row = other.row;
    col = other.col;
    return *this;
}

bool Point::operator==(const Point& other) const {
    return row == other.row && col == other.col;
}

bool Point::operator!=(const Point& other) const {
    return !(*this == other);
}

std::ostream& operator<<(std::ostream& out, const Point& p) {
    out << "(" << p.row << ", " << p.col << ")";
    return out;
}

std::istream& operator>>(std::istream& in, Point& p) {
    in >> p.row >> p.col;
    return in;
}
```

### `player.hpp`

```cpp
#pragma once

#include <iostream>
#include "cell.hpp"

class Player {
   Cell _symbol;
public:
   // constructor implicit
   Player();
   // constructor cu parametri
   Player(Cell symbol);
   // constructor de copiere
   Player(const Player& other);
   // operator de atribuire
   Player& operator=(const Player& other);
   // operatori de comparare
   bool operator==(const Player& other) const;
   bool operator!=(const Player& other) const;

   Cell GetSymbol() const;
};

std::ostream& operator<<(std::ostream& out, const Player& p);
std::istream& operator>>(std::istream& in, Player& p);
```

### `player.cpp`

```cpp
#include "player.hpp"

Player::Player() {
    _symbol = Cell::Empty;
}

Player::Player(Cell symbol) {
    _symbol = symbol;
}

Player::Player(const Player& other) {
    _symbol = other._symbol;
}

Player& Player::operator=(const Player& other) {
    _symbol = other._symbol;
    return *this;
}

bool Player::operator==(const Player& other) const {
    return _symbol == other._symbol;
}

bool Player::operator!=(const Player& other) const {
    return !(*this == other);
}

Cell Player::GetSymbol() const {
    return _symbol;
}

std::ostream& operator<<(std::ostream& out, const Player& p) {
    Cell s = p.GetSymbol();
    if (s == Cell::X) out << "X";
    else if (s == Cell::O) out << "O";
    else out << ".";
    return out;
}

std::istream& operator>>(std::istream& in, Player& p) {
    char c;
    in >> c;
    if (c == 'X' || c == 'x') p = Player(Cell::X);
    else if (c == 'O' || c == 'o') p = Player(Cell::O);
    else p = Player(Cell::Empty);
    return in;
}
```

### `board.hpp`

```cpp
#pragma once

#include <iostream>
#include "cell.hpp"
#include "point.hpp"

class Board {
   Cell _cells[3][3];
public:
   // constructor implicit
   Board();
   // constructor de copiere
   Board(const Board& other);
   // operator de atribuire
   Board& operator=(const Board& other);
   // operatori de comparare
   bool operator==(const Board& other) const;
   bool operator!=(const Board& other) const;

   void Reset();
   bool PlaceSymbol(const Point& position, Cell symbol);
   Cell GetCell(const Point& position) const;
   bool IsFull() const;
};

std::ostream& operator<<(std::ostream& out, const Board& b);
std::istream& operator>>(std::istream& in, Board& b);
```

### `board.cpp`

```cpp
#include "board.hpp"

Board::Board() {
    Reset();
}

Board::Board(const Board& other) {
    for (int r = 0; r < 3; r++)
        for (int c = 0; c < 3; c++)
            _cells[r][c] = other._cells[r][c];
}

Board& Board::operator=(const Board& other) {
    for (int r = 0; r < 3; r++)
        for (int c = 0; c < 3; c++)
            _cells[r][c] = other._cells[r][c];
    return *this;
}

bool Board::operator==(const Board& other) const {
    for (int r = 0; r < 3; r++)
        for (int c = 0; c < 3; c++)
            if (_cells[r][c] != other._cells[r][c])
                return false;
    return true;
}

bool Board::operator!=(const Board& other) const {
    return !(*this == other);
}

void Board::Reset() {
    for (int r = 0; r < 3; r++)
        for (int c = 0; c < 3; c++)
            _cells[r][c] = Cell::Empty;
}

bool Board::PlaceSymbol(const Point& position, Cell symbol) {
    if (_cells[position.row][position.col] != Cell::Empty)
        return false;
    _cells[position.row][position.col] = symbol;
    return true;
}

Cell Board::GetCell(const Point& position) const {
    return _cells[position.row][position.col];
}

bool Board::IsFull() const {
    for (int r = 0; r < 3; r++)
        for (int c = 0; c < 3; c++)
            if (_cells[r][c] == Cell::Empty)
                return false;
    return true;
}

std::ostream& operator<<(std::ostream& out, const Board& b) {
    for (int r = 0; r < 3; r++) {
        for (int c = 0; c < 3; c++) {
            Cell cell = b.GetCell({r, c});
            if (cell == Cell::X) out << " X ";
            else if (cell == Cell::O) out << " O ";
            else out << " . ";
            if (c < 2) out << "|";
        }
        out << "\n";
    }
    return out;
}

std::istream& operator>>(std::istream& in, Board& b) {
    b.Reset();
    return in;
}
```

---

## Prompt: Clasa de bază abstractă (punctul 7)

> Ce fac pentru clasa de redare abstractă?

### Răspuns

Se creează un fișier nou `drawable.hpp` cu o clasă abstractă, iar `Painter` moștenește din ea.

### `drawable.hpp` (fișier nou)

```cpp
#pragma once

#include "board.hpp"

// clasa de baza abstracta pentru redare
class Drawable {
public:
    // metoda virtuala pura - orice clasa de redare trebuie sa o implementeze
    virtual void DrawBoard(const Board& board) = 0;

    // destructor virtual (bun stil la clase de baza)
    virtual ~Drawable() {}
};
```

### `painter.hpp` (modificat să moștenească din Drawable)

```cpp
#pragma once

#include "board.hpp"
#include "cell.hpp"
#include "point.hpp"
#include "drawable.hpp"

class Painter : public Drawable {
public:
   void DrawBoard(const Board& board) override;
   void WriteMessage(const Point& position, const char* text);
   void ClearScreen();
};
```

Fișierul `painter.cpp` rămâne neschimbat față de Lab 2.

Explicație:
- `Drawable` este o clasă **abstractă** — nu poate fi instanțiată direct, pentru că are o metodă virtuală pură (`= 0`).
- O metodă virtuală pură obligă orice clasă care moștenește din `Drawable` să implementeze `DrawBoard`.
- `Painter : public Drawable` înseamnă că `Painter` moștenește din `Drawable`, deci este obligat să aibă `DrawBoard`.
- `override` confirmă că metoda suprascrie pe cea din clasa de bază.
- Avantaj: orice alt desenator viitor (de exemplu grafic) va moșteni din `Drawable` și va respecta același contract.

---

## Prompt: Compilarea și publicarea

> Am făcut modificările, ce fac acum?

### Răspuns

Compilarea (în terminalul MINGW64, în directorul proiectului):

```
make clean
make
```

Compilarea s-a încheiat cu succes și a produs `TicTacToe.exe`, fără erori.

Publicarea:

```
git add .
git commit -m "constructori si operatori"
git push --set-upstream origin lab03
```

Apoi `lab03` a fost unit în `main` printr-un pull request.

---

## Concepte explicate (pentru apărare)

- **Constructor implicit** — creează obiectul fără parametri, cu valori de start.
- **Constructor cu parametri** — creează obiectul cu valori date; `this->row` deosebește câmpul de parametru.
- **Constructor de copiere** — `Tip(const Tip& other)` — creează un obiect nou copiind altul.
- **Operator de atribuire** — `operator=` — copiază într-un obiect deja existent; `return *this;` permite înlănțuirea (`a = b = c`).
- **Operatori de comparare** — `operator==` compară conținutul; `operator!=` este negarea lui `==`.
- **Operatori de intrare/ieșire** — `operator<<` (afișare) și `operator>>` (citire) sunt **funcții libere**, nu metode, pentru că operandul din stânga este `std::cout` / `std::cin`.
- **Clasă abstractă** — o clasă cu cel puțin o metodă virtuală pură (`= 0`); nu poate fi instanțiată direct, servește drept contract pentru clasele derivate.
- **Metodă virtuală pură** (`virtual ... = 0`) — obligă clasele derivate să o implementeze.
- **Moștenire** (`class Painter : public Drawable`) — `Painter` preia contractul din `Drawable`.
- **`override`** — confirmă că o metodă suprascrie una din clasa de bază.
