# Laborator 1 — Crearea unui proiect. Lucrul cu GIT

Acest fișier conține interacțiunea cu asistentul AI (LLM) folosit la realizarea laboratorului: prompturile utilizate și răspunsurile generate, în ordine.

Joc ales: **Tic-tac-toe (X și 0)**.

---

## Prompt: Ce trebuie să fac?

> Am ales tic-tac-toe pentru laboratorul de GIT. Dă-mi pas cu pas ce trebuie să fac și explică-mi.

### Răspuns

Planul complet al laboratorului:

1. Creează cont pe GitHub.
2. Creează un repozitoriu nou (`TicTacToe`).
3. Clonează repozitoriul local.
4. Creează ramura de lucru `lab01`.
5. Scrie fișierele `.hpp` cu structurile de bază ale jocului, inclusiv cele trei obligatorii: motor (`game_engine.hpp`), desenator (`painter.hpp`), ascultător (`listener.hpp`).
6. Scrie `README.md` cu denumirea, regulile și structurile de date.
7. Publică codul pe GitHub în ramura `lab01` și unește-l în `main` printr-un pull request.

Ideea de bază — separarea responsabilităților:

- **Motorul (GameEngine)** — ține logica jocului (tabla, regulile, câștigătorul). Nu afișează, nu citește.
- **Desenatorul (Painter)** — afișează pe ecran. Nu știe regulile.
- **Ascultătorul (Listener)** — citește intrarea jucătorului. Nu știe regulile, nu afișează.

---

## Prompt: Crearea repozitoriului

> Am făcut contul. Cum creez repozitoriul?

### Răspuns

1. Pe GitHub, apasă `+` (dreapta sus) → **New repository**.
2. Nume: `TicTacToe`, public.
3. **Create repository**.
4. Copiază URL-ul (butonul verde **Code** → HTTPS).

---

## Prompt: Clonarea și ramura lab01

> Am creat repozitoriul. Mai departe?

### Răspuns

Clonează repozitoriul în Visual Studio (Clone a repository → lipește URL-ul → Clone).

Creează ramura `lab01`:
- meniul **Git → New Branch** → nume `lab01` → based on `main` → Create.

Atenție: branch-ul nu se poate crea dacă repozitoriul este complet gol (fără niciun commit). În acest caz, se face întâi un prim commit pe `main`, apoi se creează `lab01`.

---

## Prompt: Scrierea fișierelor .hpp

> Vreau să scriu eu fișierele, tu dă-mi codul și explică.

### Răspuns

Fișierele se scriu de la cel mai simplu la cel mai complex. În această etapă se scriu doar **declarațiile** (ce există), nu implementarea.

### `point.hpp`

```cpp
#pragma once

struct Point {
   int row;
   int col;
};
```

`Point` reține o poziție pe tablă prin rând (`row`) și coloană (`col`). `#pragma once` asigură includerea o singură dată a fișierului.

### `cell.hpp`

```cpp
#pragma once

enum class Cell {
   Empty, X, O
};
```

`Cell` este o enumerare ce descrie starea unei căsuțe: goală, cu X sau cu O. `enum class` obligă folosirea formei `Cell::X`, deci previne confuziile.

### `player.hpp`

```cpp
#pragma once

#include "cell.hpp"

class Player {
   Cell _symbol;
public:
   Player();
   Player(Cell symbol);
   Cell GetSymbol() const;
};
```

`Player` reprezintă un jucător, caracterizat prin simbolul cu care joacă. Câmpul `_symbol` este privat (încapsulare); accesul se face prin `GetSymbol()`.

### `board.hpp`

```cpp
#pragma once

#include "cell.hpp"
#include "point.hpp"

class Board {
   Cell _cells[3][3];
public:
   Board();
   void Reset();
   bool PlaceSymbol(const Point& position, Cell symbol);
   Cell GetCell(const Point& position) const;
   bool IsFull() const;
};
```

`Board` reprezintă tabla, stocată ca matrice 3×3. Oferă golire, plasare de simbol, citire a unei căsuțe și verificare dacă e plină.

### `game_engine.hpp` (motorul — obligatoriu)

```cpp
#pragma once

#include "board.hpp"
#include "player.hpp"
#include "point.hpp"
#include "cell.hpp"

class GameEngine {
   Board _board;
   Player _players[2];
   int _currentPlayer;
public:
   GameEngine();
   void Init();
   void Run();
   bool MakeMove(const Point& position);
   Cell CheckWinner() const;
   void SwitchPlayer();
};
```

`GameEngine` este motorul jocului. Reunește tabla și cei doi jucători, ține evidența jucătorului la rând și aplică regulile.

### `painter.hpp` (desenatorul — obligatoriu)

```cpp
#pragma once

#include "board.hpp"
#include "cell.hpp"
#include "point.hpp"

class Painter {
public:
   void DrawBoard(const Board& board);
   void WriteMessage(const Point& position, const char* text);
   void ClearScreen();
};
```

`Painter` se ocupă de afișarea pe ecran: desenează tabla, scrie mesaje, curăță ecranul.

### `listener.hpp` (ascultătorul — obligatoriu)

```cpp
#pragma once

#include "point.hpp"

class Listener {
public:
   Point GetMove();
   bool WantsToQuit();
};
```

`Listener` preia intrarea de la jucător: căsuța aleasă și eventuala intenție de a ieși.

---

## Prompt: README.md

> Dă-mi textul pentru README.

### Răspuns

```markdown
# TicTacToe

## Despre proiect

Acest proiect este o implementare a jocului clasic X și 0 (Tic-tac-toe), scrisă în
limbajul C++. Doi jucători se înfruntă pe o tablă de 3×3 căsuțe, punând pe rând
simbolul lor (X sau O). Câștigă cel care reușește primul să alinieze trei simboluri
identice pe un rând, o coloană sau o diagonală.

## Cum se joacă

1. Tabla are 9 căsuțe, dispuse pe 3 rânduri și 3 coloane, toate goale la început.
2. Cei doi jucători mută pe rând: unul joacă cu X, celălalt cu O.
3. La rândul său, un jucător alege o căsuță liberă și își pune simbolul acolo.
4. O căsuță deja ocupată nu poate fi aleasă din nou.
5. Partida este câștigată de jucătorul care aliniază trei simboluri identice pe
   orizontală, verticală sau diagonală.
6. Dacă toate căsuțele sunt ocupate și nimeni nu a aliniat trei simboluri, partida
   se termină la egalitate (remiză).

## Tipurile de date folosite

- Point (point.hpp) — o structură care reține o poziție pe tablă (row, col).
- Cell (cell.hpp) — o enumerare: starea unei căsuțe (Empty, X, O).
- Player (player.hpp) — o clasă: un jucător și simbolul său.
- Board (board.hpp) — clasa tablei de joc 3×3.
- GameEngine (game_engine.hpp) — motorul jocului.
- Painter (painter.hpp) — componenta de afișare.
- Listener (listener.hpp) — componenta de intrare.
```

---

## Prompt: Problema cu fișierele temporare la commit

> La commit îmi dă eroarea „Permission denied" pe folderul .vs.

### Răspuns

Fișierele temporare din folderul `.vs` (generate de Visual Studio) blochează commitul. Soluția este un fișier `.gitignore`:

```
# Fisiere temporare Visual Studio
.vs/
*.slnx
ipch/
*.db
*.opendb
Debug/
Release/
x64/
x86/
*.user
```

Apoi, în terminal, scoaterea folderului `.vs` din urmărire și commitul:

```
git rm -r --cached .vs
git add .
git commit -m "initial commit"
```

---

## Prompt: Publicarea și pull request

> Cum public pe GitHub și fac pull request?

### Răspuns

Publicarea ramurii:

```
git push --set-upstream origin lab01
```

(sau butonul Push din Visual Studio)

Pull request:
1. Pe GitHub: tab **Pull requests** → **New pull request**.
2. Setează **base: main ← compare: lab01**.
3. **Create pull request** → **Merge pull request** → **Confirm merge**.

Ramura `lab01` nu se șterge (este cerută de laborator).

---

## Concepte explicate (pentru apărare)

- **`#pragma once`** — include fișierul o singură dată.
- **`struct` vs `class`** — la `struct` membrii sunt publici implicit; la `class` sunt privați (încapsulare).
- **`enum class`** — tip cu set fix de valori cu nume.
- **`#include`** — aduce conținutul altui fișier; `" "` pentru fișierele proprii, `< >` pentru bibliotecile standard.
- **`const` la finalul metodei** — metoda nu modifică obiectul, doar îl citește.
- **`const Tip&`** — primește obiectul original (eficient, prin `&`) fără a-l modifica (sigur, prin `const`).
- **Branch / commit / push / pull request** — ramură de lucru / salvare locală / publicare pe server / propunerea și integrarea modificărilor în `main`.
