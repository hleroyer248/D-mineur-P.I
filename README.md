#include <iostream>
#include <ctime>
#include <queue> // utilisation d'une file pour révélations cases vide

using namespace std;

const int MAX_LINES = 30;
const int MAX_COLL = 16;

struct Cases {
    bool Mine = false;
    bool Revealed = false;
    bool Flag = false;
    int adjtMines = 0;
};

// board du jeu
Cases board[MAX_LINES][MAX_COLL];
int lines, coll, totalMines;
bool Perdu = false;
bool Gagner = false;

// choix des niveaux
void Levels() {

    int choice;

    cout << "choice difficulty level:" << endl;
    cout << "1. easy (9x9, 12 mines)" << endl;
    cout << "2. Normal (16x16, 38 mines)" << endl;
    cout << "3. hard (30x16, 72 mines)" << endl;
    cout << "choice: ";
    cin >> choice;

    switch (choice) {
    case 1:
        coll = 9;
        lines = 9;
        totalMines = 12;
        break;

    case 2:
        coll = 16;
        lines = 16;
        totalMines = 38;
        break;

    case 3:
        coll = 30;
        lines = 16;
        totalMines = 72;
        break;

    default:
        cout << "Choix invalide, niveau facile par defaut." << endl;
        lines = 9;
        coll = 9;
        totalMines = 12;
    }
}

// afficher le tableau
void printBoard(bool revealAll = false) {
    cout << "    ";

    // affichange de la légende 
    for (int c = 0; c < coll; c++) {

        if (c < 10) cout << c << "  ";

        else cout << c << " ";
    }
    cout << endl;

    for (int l = 0; l < lines; l++) {

        if (l < 10) cout << " " << l << " ";

        else cout << l << " ";

        // affichage complet si le jouer selectionne une mine
        for (int c = 0; c < coll; c++) {

            if (revealAll) {

                if (board[l][c].Mine) cout << "[*]";

                else if (board[l][c].adjtMines > 0) cout << " " << board[l][c].adjtMines << " "; // le premier sert a afficher les cases vides et le deuxiéme pour le chiffres

                else cout << "   ";
            }
            else {

                // affichage du tableau pour le joueur 
                if (board[l][c].Revealed) {

                    if (board[l][c].Mine) cout << "[*]";

                    else if (board[l][c].adjtMines > 0) cout << " " << board[l][c].adjtMines << " ";

                    else cout << "   ";

                }

                else if (board[l][c].Flag) {
                    cout << "[F]";
                }

                else {
                    cout << "[ ]";
                }
            }
        }
        cout << endl;
    }
}

// vérifie que la case selectionnée est bien dans le tableau
bool isInside(int l, int c) {

    return l >= 0 && l < lines && c >= 0 && c < coll;
}

// pose les mines , 15%
void placeMines(int safeL, int safeC) {

    int placed = 0;
    while (placed < totalMines) {

        int l = rand() % lines;
        int c = rand() % coll;

        if ((l == safeL && c == safeC) || board[l][c].Mine) continue; // évite de tomber sur une mine au premier coup
        board[l][c].Mine = true;
        placed++;
    }
}

// calcul des mines adj
void calculateAdjacents() {

    // direction des 8 cases

    /*  (-1, -1)   (-1, 0)   (-1, +1)
        (0, -1)      (X)     (0, +1)
        (+1, -1)   (+1, 0)   (+1, +1)*/

    int dx[] = { -1, -1, -1, 0, 1, 1, 1, 0 };
    int dy[] = { -1, 0, 1, 1, 1, 0, -1, -1 };

    for (int l = 0; l < lines; l++) {

        for (int c = 0; c < coll; c++) {

            if (board[l][c].Mine) continue;

            int count = 0;
            for (int i = 0; i < 8; i++) {

                int nr = l + dy[i];
                int nc = c + dx[i];

                if (isInside(nr, nc) && board[nr][nc].Mine) // affiche une erreur mais la fonction fonctionne quand meme 
                {
                    count++;
                }
            }
            board[l][c].adjtMines = count;
        }
    }
}

//révélation des cases choisis
void CaseReveal(int startL, int startC) {
    queue<pair<int, int>> toReveal; // case de départ entrez dans la file, tant que la cage adjacente = 0 , le file continue a révélé les cases
    toReveal.push({ startL, startC }); 
    board[startL][startC].Revealed = true;

    while (!toReveal.empty()) {
        int l = toReveal.front().first;
        int c = toReveal.front().second;
        toReveal.pop();


        for (int dl = -1; dl <= 1; ++dl) {
            for (int dc = -1; dc <= 1; ++dc) {
                int nl = l + dl;
                int nc = c + dc;

                if (!isInside(nl, nc)) continue;
                if (board[nl][nc].Revealed || board[nl][nc].Flag) continue;

                board[nl][nc].Revealed = true;


                if (board[nl][nc].adjtMines == 0) {
                    toReveal.push({ nl, nc });
                }
            }
        }
    }
}

// révélation 
void reveal(int l, int c) {

    if (!isInside(l, c) || board[l][c].Revealed || board[l][c].Flag) return;

    board[l][c].Revealed = true;

    if (board[l][c].Mine) { // si mine = perdu
        Perdu = true;
        return;
    }


    if (board[l][c].adjtMines == 0) {
        CaseReveal(l, c);
    }
}

// vérification de victoire
bool isWin() {
    for (int l = 0; l < lines; l++) {
        for (int c = 0; c < coll; c++) {
            if (!board[l][c].Mine && !board[l][c].Revealed) return false;
        }
    } // si case sans mine non révélé = pas encore gagné
    return true;
}

// fonction principale du jeu
void Demineur() {
    bool firstMove = true; // pour évité que la première cases révélé soit une mine

    while (!Perdu && !Gagner) {
        printBoard();
        int l, c;          
        char action; // pour demander l'action faite par le joueur
        cout << endl << "Entrez votre action (r = reveler, f = drapeau): ";
        cin >> action;
        cout << endl << "Entrez les coordonnees: ";
        cin >> l >> c;

        if (!isInside(l, c)) {
            cout << "Coordonnees invalides." << endl;
            continue;
        }

        if (action == 'f') { // placer un drapeau
            board[l][c].Flag = !board[l][c].Flag;
        }
        else if (action == 'r') { // révélé une cases
            if (firstMove) {
                placeMines(l, c);
                calculateAdjacents();
                firstMove = false;
            }
            reveal(l, c);
        }
        else { // en cas d'erreur
            cout << "Action invalide. Utilisez 'r' pour révéler ou 'f' pour drapeau." << endl;
        }

        if (isWin()) { // si toute cases révélé
            Gagner = true;
        }
    }

    printBoard(true); // message de victoire
    if (Gagner) {
        cout << endl << " Vous avez gagne, GG!" << endl;
    }
    else {
        cout << endl << " Vous avez perdu, FF!" << endl;
    }
}
// pour pouvoir reset et rejouer
void replayBoard() {  
    for (int l = 0; l < MAX_LINES; l++) {
        for (int c = 0; c < MAX_COLL; c++) {
            board[l][c] = Cases();
        }
    }
    Perdu = false;
    Gagner = false;
}


int main() {
    srand(static_cast<unsigned int>(time(0)));

    char rejouer;// int ne fonctionne pas

    do {
        replayBoard();
        cout << endl << " [] = case non revelee " << endl;
        Levels();
        Demineur();

        cout << "rejouer ? (o/n) : ";
        cin >> rejouer;

        while (rejouer != 'o' && rejouer != 'n') {
            cout << "'o' pour oui ou 'n' pour non : ";
            cin >> rejouer;
        }

    } while (rejouer == 'o');
    };
