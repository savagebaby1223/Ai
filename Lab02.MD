Această sarcină continuă direct laboratorul 1. În lab01 ai scris *ce* există (declarațiile din `.hpp`). Acum scrii *cum* funcționează (implementările din `.cpp`) și înveți cum se transformă codul într-un program executabil.

---

## Pasul 0: Ramura `lab02`

Lucrul se face pe o ramură nouă, creată **din `lab01`**, ca să păstrezi tot ce ai făcut deja:

```bash
git checkout lab01
git checkout -b lab02
```

---

## Pasul 1: Implementarea metodelor (2p)

Regula de bază este simplă: **fiecare `.hpp` primește un `.cpp` cu același nume.** În `.cpp` scrii corpul metodelor folosind sintaxa `NumeClasa::numeMetoda`. Operatorul `::` îi spune compilatorului că metoda aparține acelei clase.

**`GameEngine.cpp`**
```cpp
#include "GameEngine.hpp"

GameEngine::GameEngine() {
    reset();
}

void GameEngine::reset() {
    for (int r = 0; r < SIZE; ++r)
        for (int c = 0; c < SIZE; ++c)
            board[r][c] = Cell::Empty;
    currentPlayer = Cell::X;
    state = GameState::Playing;
}

bool GameEngine::isValidMove(const Move& m) const {
    return state == GameState::Playing &&
           m.row >= 0 && m.row < SIZE &&
           m.col >= 0 && m.col < SIZE &&
           board[m.row][m.col] == Cell::Empty;
}

bool GameEngine::makeMove(const Move& m) {
    if (!isValidMove(m)) return false;
    board[m.row][m.col] = currentPlayer;
    updateState();
    if (state == GameState::Playing) switchPlayer();
    return true;
}

void GameEngine::switchPlayer() {
    currentPlayer = (currentPlayer == Cell::X) ? Cell::O : Cell::X;
}

void GameEngine::updateState() {
    Cell p = currentPlayer;   // doar cel care a mutat acum poate câștiga
    bool win = false;

    for (int i = 0; i < SIZE; ++i) {
        if (board[i][0] == p && board[i][1] == p && board[i][2] == p) win = true; // rând
        if (board[0][i] == p && board[1][i] == p && board[2][i] == p) win = true; // coloană
    }
    if (board[0][0] == p && board[1][1] == p && board[2][2] == p) win = true;     // diagonala
    if (board[0][2] == p && board[1][1] == p && board[2][0] == p) win = true;     // antidiagonala

    if (win) {
        state = (p == Cell::X) ? GameState::XWins : GameState::OWins;
        return;
    }

    for (int r = 0; r < SIZE; ++r)
        for (int c = 0; c < SIZE; ++c)
            if (board[r][c] == Cell::Empty) return;   // mai sunt locuri libere

    state = GameState::Draw;
}

GameState GameEngine::getState() const        { return state; }
Cell GameEngine::getCell(int r, int c) const  { return board[r][c]; }
Cell GameEngine::getCurrentPlayer() const     { return currentPlayer; }
```

Observă că `makeMove` apelează mai întâi `isValidMove`. Aceasta este încapsularea din lab01 în acțiune: tabla nu poate ajunge niciodată într-o stare greșită, pentru că singura cale de a o modifica trece prin verificare.

**`Renderer.cpp`**

Mai întâi adaugă în `Renderer.hpp` o metodă nouă, `void showTurn(Cell player) const;`. Este normal ca declarațiile să evolueze pe parcurs.

```cpp
#include "Renderer.hpp"
#include <iostream>

namespace {
    char toChar(Cell c) {
        switch (c) {
            case Cell::X: return 'X';
            case Cell::O: return 'O';
            default:      return ' ';
        }
    }
}

void Renderer::drawBoard(const GameEngine& engine) const {
    std::cout << "\n    1   2   3\n";
    for (int r = 0; r < GameEngine::SIZE; ++r) {
        std::cout << ' ' << r + 1 << ' ';
        for (int c = 0; c < GameEngine::SIZE; ++c) {
            std::cout << ' ' << toChar(engine.getCell(r, c)) << ' ';
            if (c < GameEngine::SIZE - 1) std::cout << '|';
        }
        std::cout << '\n';
        if (r < GameEngine::SIZE - 1) std::cout << "   ---+---+---\n";
    }
    std::cout << '\n';
}

void Renderer::showTurn(Cell player) const {
    std::cout << "Rândul lui " << toChar(player) << ". Introdu rândul și coloana (1-3): ";
}

void Renderer::showMessage(const std::string& message) const {
    std::cout << message << '\n';
}

void Renderer::showResult(GameState state) const {
    switch (state) {
        case GameState::XWins: std::cout << "A câștigat X!\n"; break;
        case GameState::OWins: std::cout << "A câștigat O!\n"; break;
        case GameState::Draw:  std::cout << "Remiză!\n";        break;
        default: break;
    }
}
```

`toChar` este pusă într-un `namespace` fără nume. Asta înseamnă că funcția e vizibilă doar în acest fișier, fiind un detaliu intern al desenatorului.

**`Listener.cpp`**
```cpp
#include "Listener.hpp"
#include <iostream>
#include <limits>
#include <cstdlib>

Move Listener::readMove() const {
    int r, c;
    if (!(std::cin >> r >> c)) {
        if (std::cin.eof()) std::exit(0);   // utilizatorul a închis input-ul
        std::cin.clear();                   // resetează starea de eroare
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
        return {-1, -1};                    // mutare invalidă, motorul o va respinge
    }
    return {r - 1, c - 1};                  // jucătorul scrie 1-3, tabla folosește 0-2
}

bool Listener::askPlayAgain() const {
    char answer;
    std::cin >> answer;
    return answer == 'd' || answer == 'D';
}
```

Ascultătorul doar citește. Nu verifică dacă celula e liberă, pentru că asta e treaba motorului.

**`main.cpp`**: aici cele trei componente se întâlnesc.
```cpp
#include "GameEngine.hpp"
#include "Renderer.hpp"
#include "Listener.hpp"

int main() {
    GameEngine engine;
    Renderer renderer;
    Listener listener;

    do {
        engine.reset();
        while (engine.getState() == GameState::Playing) {
            renderer.drawBoard(engine);
            renderer.showTurn(engine.getCurrentPlayer());
            if (!engine.makeMove(listener.readMove()))
                renderer.showMessage("Mutare invalidă, încearcă din nou.");
        }
        renderer.drawBoard(engine);
        renderer.showResult(engine.getState());
        renderer.showMessage("Mai jucați? (d/n): ");
    } while (listener.askPlayAgain());

    return 0;
}
```

---

## Pasul 2: Compilarea unui fișier în fișier obiect

Construirea unui program C++ are **două etape**, iar profesorul vrea să le vezi separat:

1. **Compilarea**: fiecare `.cpp` este transformat separat într-un **fișier obiect** (`.o`), adică cod mașină care încă nu știe nimic de celelalte fișiere.
2. **Linkarea**: toate fișierele `.o` sunt unite într-un singur executabil.

Compilezi un singur fișier, cu opțiunea `-c` („doar compilează, nu lega”):

```bash
g++ -std=c++17 -Wall -c GameEngine.cpp -o GameEngine.o
```

Va apărea `GameEngine.o`. Dacă încerci să-l rulezi, nu merge: nu are `main` și nu e legat de nimic. Acesta e un lucru bun de arătat la apărare.

Ce înseamnă opțiunile: `-std=c++17` alege standardul limbajului, `-Wall` activează avertismentele, `-c` oprește procesul după compilare, iar `-o` dă numele fișierului rezultat.

> Pe Windows ai nevoie de compilatorul g++ instalat (de exemplu prin MSYS2/MinGW). Verifici cu `g++ --version`.

---

## Pasul 3: Fișierul de construcție (4p, cea mai valoroasă parte)

Fără un fișier de construcție ar trebui să scrii de fiecare dată 5 comenzi lungi. Le scrii o singură dată într-un fișier. Îți recomand **Makefile**, iar mai jos ai și varianta `cmd`.

**`Makefile`**
```makefile
CXX      = g++
CXXFLAGS = -std=c++17 -Wall -Wextra
TARGET   = tictactoe
OBJS     = main.o GameEngine.o Renderer.o Listener.o

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CXX) $(OBJS) -o $(TARGET)

main.o: main.cpp GameEngine.hpp Renderer.hpp Listener.hpp Types.hpp
	$(CXX) $(CXXFLAGS) -c main.cpp -o main.o

GameEngine.o: GameEngine.cpp GameEngine.hpp Types.hpp
	$(CXX) $(CXXFLAGS) -c GameEngine.cpp -o GameEngine.o

Renderer.o: Renderer.cpp Renderer.hpp GameEngine.hpp Types.hpp
	$(CXX) $(CXXFLAGS) -c Renderer.cpp -o Renderer.o

Listener.o: Listener.cpp Listener.hpp Types.hpp
	$(CXX) $(CXXFLAGS) -c Listener.cpp -o Listener.o

clean:
	rm -f $(OBJS) $(TARGET) $(TARGET).exe

.PHONY: all clean
```

⚠️ **Foarte important:** liniile cu comenzi (cele de sub reguli) trebuie să înceapă cu **TAB**, nu cu spații. Altfel primești eroarea `missing separator`.

Cum citești o regulă:
```
GameEngine.o: GameEngine.cpp GameEngine.hpp Types.hpp
	comanda
```
Ea spune: „ca să obții `GameEngine.o`, ai nevoie de aceste fișiere. Dacă vreunul s-a schimbat de la ultima construire, rulează comanda.” Aici e avantajul major al Makefile-ului: dacă modifici doar `Renderer.cpp`, la următorul `make` se recompilează doar `Renderer.o`, apoi se refac legăturile. Într-un proiect mare asta economisește minute întregi.

Utilizare:
```bash
make          # construiește proiectul
./tictactoe   # rulează (pe Windows: tictactoe.exe)
make clean    # șterge fișierele generate
```

> Pe Windows, cu MinGW, comanda se poate numi `mingw32-make`, iar `rm` poate lipsi. În cazul acesta e mai simplă varianta `cmd` de mai jos.

**`build.cmd`** (alternativa pentru Windows)
```bat
@echo off
g++ -std=c++17 -Wall -c main.cpp       -o main.o       || goto error
g++ -std=c++17 -Wall -c GameEngine.cpp -o GameEngine.o || goto error
g++ -std=c++17 -Wall -c Renderer.cpp   -o Renderer.o   || goto error
g++ -std=c++17 -Wall -c Listener.cpp   -o Listener.o   || goto error
g++ main.o GameEngine.o Renderer.o Listener.o -o tictactoe.exe || goto error
echo Construire reusita!
exit /b 0

:error
echo Eroare la construire!
exit /b 1
```

`|| goto error` înseamnă: „dacă această comandă eșuează, sari la eticheta `error`”. Așa scriptul nu continuă după o eroare de compilare. Spre deosebire de Makefile, scriptul `cmd` recompilează totul de fiecare dată. Dacă le pui pe amândouă în proiect, ai și ce compara la apărare.

---

## Pasul 4: `.gitignore` (1p)

Fișierele `.o` și `.exe` se **generează** din codul sursă, deci nu au ce căuta în repozitoriu: ocupă loc, diferă de la calculator la calculator și pot fi refăcute oricând cu `make`. Creează fișierul `.gitignore` (cu punct la început):

```gitignore
# Fișiere obiect
*.o
*.obj

# Executabile
*.exe
tictactoe

# Fișiere ale editoarelor
.vscode/
.idea/
```

Verifici că funcționează: după `make`, comanda `git status` **nu** trebuie să afișeze fișierele `.o` și executabilul.

---

## Pasul 5: Actualizarea README.md (1p)

Adaugă o secțiune nouă:

````markdown
## Construirea proiectului

Necesită compilatorul g++ (C++17).

### Cu Makefile
```
make          # construiește
./tictactoe   # rulează
make clean    # curăță fișierele generate
```

### Cu script cmd (Windows)
```
build.cmd
tictactoe.exe
```

### Manual
```
g++ -std=c++17 -c GameEngine.cpp -o GameEngine.o
g++ -std=c++17 -c Renderer.cpp -o Renderer.o
g++ -std=c++17 -c Listener.cpp -o Listener.o
g++ -std=c++17 -c main.cpp -o main.o
g++ main.o GameEngine.o Renderer.o Listener.o -o tictactoe
```
````

Nu uita să **adaugi și această conversație în `prompts.md`**, ca la laboratorul anterior.

---

## Pasul 6: Publicarea (2p)

Fă commit-uri separate, pe etape. Istoricul arată astfel că ai lucrat progresiv:

```bash
git add GameEngine.cpp Renderer.cpp Listener.cpp main.cpp Renderer.hpp
git commit -m "Implementarea metodelor claselor"

git add Makefile build.cmd
git commit -m "Adăugat fișiere de construcție"

git add .gitignore README.md prompts.md
git commit -m "Adăugat .gitignore și instrucțiuni de construire"

git push -u origin lab02
```

Pe GitHub verifici că ramura `lab02` conține toate fișierele și **niciun** `.o` sau `.exe`. Apoi pui link-ul în Moodle.

---

## Structura finală

```
tic-tac-toe/
├── .gitignore
├── README.md
├── prompts.md
├── Makefile
├── build.cmd
├── Types.hpp
├── GameEngine.hpp   GameEngine.cpp
├── Renderer.hpp     Renderer.cpp
├── Listener.hpp     Listener.cpp
└── main.cpp
```

Un sfat important: înainte de a preda, **rulează și joacă efectiv** câteva partide (câștig pe rând, pe coloană, pe diagonală, remiză, mutare invalidă). Dacă ceva nu compilează sau nu se comportă cum te aștepți, scrie-mi eroarea exactă și o rezolvăm împreună.
