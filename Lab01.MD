# Rezolvare lab01

## Prompt

tu esti un profesor de c++ cu abilitati de explicare. 

eu am urmatoarea sarcina

---

### Crearea unui proiect. Lucrul cu GIT

#### Scop

După executarea acestei lucrării de laborator student va face cunoștința cu sistem de control codului sursă GIT și va primi cunoștințe de bază necesare pentru organizarea proiectului.

#### Sarcina

1. Selectați pentru realizarea o joacă din specificate:
   1. joc de cărți "Durak"
   2. tic-tac-toe (X și O)
   3. Snake
   4. Minesweeper
   5. Pacman
   6. Dame
   7. Story Tale (text adventure)
   8. Visual Novel
   9. Rogue-like
   10. etc.
2. Înregistrați-vă pe [GitHub](https://github.com/).
3. Creați un repozitoriu nou.
4. Clonați repozitoriu pe calculator personal
5. Adăugați în repozitoriu fișier README.md cu descrierea proiectului
   1. Denumirea proiectului
   2. Descrierea proiectului / reguli de joc
   3. Structuri de date si descrierea lor
6. Creați fișiere `hpp` cu definirea structurilor de bază necesitate pentru proiect. Fisiere obligatorii:
   1. motor de joaca (de exemplu, `GameEngine.hpp` sau `Engine.hpp`)
   2. desenator (de exemplu `Painter.hpp`, `Presenter.hpp` sau `Renderer.hpp`)
   3. ascultator (de exemplu `Listener.hpp`)
7. Plasați cod pe __GitHub__

> În caz, dacă se utilizează LLM (Large Language Model) pentru obținerea rezultatului, proiectul trebuie să includă și mențiunea corespunzătoare în README.md, și fișierul `prompts.md`, care conține prompturile utilizate și răspunsurile generate.

#### Prezentare

La lucrare de laborator în Moodle adaugați referința repozitoriu.

#### Evaluare

- `1p` - crearea contului pe GitHub
- `1p` - crearea proiectului
- `2p` - crearea fișierelor cu structurile necesare
- `2p` - adăugarea descrierii proiectului
- `1p` - publicarea codului pe GitHub in ramura `lab01`
- `3p` - apararea proiectului
- `-1p` - pentru fiecare zi de întârziere
- `-5p` - pentru copierea codului de la colegi
---
eu am ales tic tac toe, explicami ce trebuie sa fac

## Raspuns

Tic-tac-toe e o alegere foarte bună: regulile sunt simple, deci te poți concentra pe ce se evaluează cu adevărat, adică **Git** și **organizarea proiectului**. Hai să luăm pașii pe rând, ca să știi și *ce* faci, și *de ce* faci. La apărare (3p) te vor întreba tocmai „de ce”.

---

### Ideea de bază: cele 3 fișiere obligatorii

Profesorul îți cere să împarți jocul în trei părți. Este un principiu clasic de proiectare, numit **separarea responsabilităților** (seamănă cu modelul MVC):

| Componentă | Ce face | Ce NU face |
|---|---|---|
| **Engine** (motorul) | Ține tabla, verifică mutările, decide cine a câștigat | Nu afișează nimic, nu citește de la tastatură |
| **Renderer** (desenatorul) | Afișează tabla și mesajele pe ecran | Nu știe regulile jocului |
| **Listener** (ascultătorul) | Citește ce introduce jucătorul | Nu știe cum arată tabla, nici regulile |

**De ce așa?** Dacă mâine vrei să treci de la consolă la o interfață grafică, schimbi doar `Renderer` și `Listener`, iar motorul rămâne neatins. Aceasta e o frază bună de spus la apărare.

---

### Pasul 1–3: Contul GitHub și repozitoriul

1. Te înregistrezi pe [github.com](https://github.com/).
2. Apeși **New repository**, îi dai un nume (de exemplu `tic-tac-toe`), îl faci **Public** și bifezi „Add a README file”.
3. Apeși **Create repository**.

### Pasul 4: Clonarea pe calculator

Instalezi Git (de pe [git-scm.com](https://git-scm.com/)), apoi în terminal scrii:

```bash
git clone https://github.com/NUMELE_TAU/tic-tac-toe.git
cd tic-tac-toe
git checkout -b lab01
```

Ultima comandă e importantă: **creează ramura `lab01`** și te mută pe ea. Ea valorează 1p separat, deci nu lucra pe `main`.

> Prima dată Git îți va cere să te prezinți:
> `git config --global user.name "Numele Tău"` și `git config --global user.email "email@exemplu.com"`

---

### Pasul 5: README.md

Trebuie să conțină cele 3 secțiuni cerute. Un exemplu de structură:

```markdown
# Tic-Tac-Toe (X și O)

## Descriere și reguli
Joc pentru doi jucători pe o tablă 3x3. Jucătorii pun pe rând X și O.
Câștigă cel care formează primul o linie de 3 simboluri (orizontal,
vertical sau pe diagonală). Dacă tabla se umple fără câștigător, e remiză.

## Structuri de date
- `Cell` – enum: starea unei celule (Empty, X, O)
- `GameState` – enum: starea jocului (Playing, XWins, OWins, Draw)
- `Move` – struct: o mutare (rând, coloană)
- `GameEngine` – clasa cu logica jocului
- `Renderer` – clasa care afișează jocul
- `Listener` – clasa care citește input-ul jucătorului

## Utilizarea LLM
La realizarea proiectului a fost folosit Claude (Anthropic).
Prompturile și răspunsurile sunt în fișierul `prompts.md`.
```

**Atenție la ultima secțiune:** pentru că mă folosești pe mine acum, regulamentul îți cere să menționezi asta în README și să creezi `prompts.md`. Acolo copiezi întrebarea ta și răspunsul meu. E mult mai sigur decât să fii prins la apărare.

---

### Pasul 6: Fișierele .hpp

În această etapă scrii doar **definițiile** (ce există), nu și implementarea (cum funcționează). Implementarea vine în laboratoarele următoare. Îți propun 4 fișiere: cele 3 obligatorii plus unul cu tipurile comune.

**`Types.hpp`**: tipurile folosite de toate celelalte fișiere.
```cpp
#pragma once

enum class Cell { Empty, X, O };                       // starea unei celule
enum class GameState { Playing, XWins, OWins, Draw };  // starea jocului

struct Move {
    int row;   // 0..2
    int col;   // 0..2
};
```

**`GameEngine.hpp`**: motorul.
```cpp
#pragma once
#include "Types.hpp"

class GameEngine {
public:
    static const int SIZE = 3;

    GameEngine();
    void reset();                         // golește tabla, începe X
    bool makeMove(const Move& move);      // false dacă mutarea e invalidă
    GameState getState() const;
    Cell getCell(int row, int col) const;
    Cell getCurrentPlayer() const;

private:
    Cell board[SIZE][SIZE];
    Cell currentPlayer;
    GameState state;

    bool isValidMove(const Move& move) const;
    void updateState();                   // verifică câștig / remiză
    void switchPlayer();
};
```

**`Renderer.hpp`**: desenatorul.
```cpp
#pragma once
#include <string>
#include "GameEngine.hpp"

class Renderer {
public:
    void drawBoard(const GameEngine& engine) const;
    void showMessage(const std::string& message) const;
    void showResult(GameState state) const;
};
```

**`Listener.hpp`**: ascultătorul.
```cpp
#pragma once
#include "Types.hpp"

class Listener {
public:
    Move readMove() const;       // citește rândul și coloana
    bool askPlayAgain() const;   // întreabă dacă se mai joacă o dată
};
```

Câteva lucruri pe care să le poți explica la apărare:

- **`#pragma once`** împiedică includerea aceluiași fișier de două ori.
- **`enum class`** în loc de `enum` simplu: valorile nu „se scurg” în spațiul global și nu pot fi confundate cu numere întregi.
- **`private`** pentru tablă: nimeni din afară nu poate modifica tabla direct, ci doar prin `makeMove`, care verifică regulile. Asta se numește **încapsulare**.
- **`const` după metodă** (`getState() const`): metoda doar citește, nu modifică obiectul.
- **`const GameEngine&`** în `Renderer`: desenatorul primește motorul prin referință, fără copiere, și nu îl poate modifica.

---

### Pasul 7: Publicarea pe GitHub

```bash
git add .
git commit -m "Lab01: README, structuri de bază (Engine, Renderer, Listener)"
git push -u origin lab01
```

Pe GitHub verifici că ramura `lab01` apare și conține toate fișierele. Apoi copiezi link-ul repozitoriului în Moodle.

**Sfat:** fă mai multe commit-uri mici (unul pentru README, unul pentru fișierele `.hpp` etc.). Un istoric de commit-uri arată că ai lucrat tu, pas cu pas, ceea ce contează și pentru suspiciunea de copiere (-5p).

---

### Structura finală a proiectului

```
tic-tac-toe/
├── README.md
├── prompts.md
├── Types.hpp
├── GameEngine.hpp
├── Renderer.hpp
└── Listener.hpp
```

Dacă vrei, te pot pregăti pentru apărare cu câteva întrebări pe care le-ar putea pune profesorul, ca să verifici că poți explica totul cu cuvintele tale.
