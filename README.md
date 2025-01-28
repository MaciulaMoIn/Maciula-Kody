# Maciula-Kody
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <locale.h>
#ifdef _WIN32
#include <windows.h>
#endif

// Definicje struktur
struct Kontakt {
    char imie[30];
    char nazwisko[30];
    char numer[20];
    char grupa[20];
};

struct Element {
    struct Kontakt dane;
    struct Element* prev;
    struct Element* next;
};

struct Lista {
    struct Element* poczatek;
    struct Element* koniec;
    int metoda_sortowania; // 0 - imie, 1 - nazwisko, 2 - grupa
};

char translit_pl(char c) {
    switch (c) {
    case 'ą': case 'Ą': return 'a';
    case 'ć': case 'Ć': return 'c';
    case 'ę': case 'Ę': return 'e';
    case 'ł': case 'Ł': return 'l';
    case 'ń': case 'Ń': return 'n';
    case 'ó': case 'Ó': return 'o';
    case 'ś': case 'Ś': return 's';
    case 'ź': case 'Ź': return 'z';
    case 'ż': case 'Ż': return 'z';
    default: return c;
    }
}

char tolower_pl(char c) {
    return tolower((unsigned char)translit_pl(c));
}

int strcasecmp_pl(const char* s1, const char* s2) {
    while (*s1 && *s2) {
        char c1 = tolower_pl(*s1);
        char c2 = tolower_pl(*s2);
        if (c1 != c2) {
            return c1 - c2;
        }
        s1++;
        s2++;
    }
    return tolower_pl(*s1) - tolower_pl(*s2);
}

void TworzenieListy(struct Lista* lista) {
    lista->poczatek = NULL;
    lista->koniec = NULL;
    lista->metoda_sortowania = 0;
}

void FreeAL(struct Lista* lista) {
    struct Element* E = lista->poczatek;
    while (E != NULL) {
        struct Element* do_usuniecia = E;
        E = E->next;
        free(do_usuniecia);
    }
    lista->poczatek = NULL;
    lista->koniec = NULL;
}

// Funkcja do dodawania na koniec listy
void AddEnde(struct Lista* lista, struct Kontakt kontakt) {
    struct Element* nowy = malloc(sizeof(struct Element));
    nowy->dane = kontakt;
    nowy->prev = lista->koniec;
    nowy->next = NULL;

    if (lista->koniec != NULL) {
        lista->koniec->next = nowy;
    }
    else {
        lista->poczatek = nowy;
    }

    lista->koniec = nowy;
}

// Funkcja wyświetlania listy
void PrintLista(struct Lista* lista) {
    struct Element* E = lista->poczatek;
    while (E != NULL) {
        printf("%s %s, Tel: %s, Grupa: %s\n",
            E->dane.imie,
            E->dane.nazwisko,
            E->dane.numer,
            E->dane.grupa);
        E = E->next;
    }
}

// Funkcja porównywania kontaktów
int PorownajK(struct Kontakt* a, struct Kontakt* b, int metoda) {
    switch (metoda) {
    case 0:
        return strcasecmp_pl(a->imie, b->imie);
    case 1:
        return strcasecmp_pl(a->nazwisko, b->nazwisko);
    case 2:
        return strcasecmp_pl(a->grupa, b->grupa);
    }
    return 0;
}

// Sortowanie listy przez przepisywanie wskaźników
void SortLista(struct Lista* lista) {
    if (lista->poczatek == NULL || lista->poczatek->next == NULL) return;

    int zamiana;
    do {
        zamiana = 0;
        struct Element* E = lista->poczatek;

        while (E->next != NULL) {
            if (PorownajK(&E->dane, &E->next->dane, lista->metoda_sortowania) > 0) {

                struct Element* vor = E->prev;
                struct Element* nach = E->next;
                struct Element* nach_nach = nach->next;

                if (vor != NULL) vor->next = nach;
                else lista->poczatek = nach;

                if (nach_nach != NULL) nach_nach->prev = E;
                else lista->koniec = E;

                nach->prev = vor;
                nach->next = E;
                E->prev = nach;
                E->next = nach_nach;

                zamiana = 1;
            }
            else {
                E = E->next;
            }
        }
    } while (zamiana);
}

// Dodawanie elementu w posortowanej liście
void DodajK(struct Lista* lista, struct Kontakt kontakt) {
    struct Element* nowy = malloc(sizeof(struct Element));
    nowy->dane = kontakt;
    nowy->prev = NULL;
    nowy->next = NULL;

    if (lista->poczatek == NULL) {
        lista->poczatek = lista->koniec = nowy;
        return;
    }

    struct Element* E = lista->poczatek;
    while (E != NULL && PorownajK(&E->dane, &kontakt, lista->metoda_sortowania) < 0) {
        E = E->next;
    }

    if (E == NULL) {
        AddEnde(lista, kontakt);
        free(nowy);
        return;
    }

    nowy->next = E;
    nowy->prev = E->prev;

    if (E->prev != NULL) {
        E->prev->next = nowy;
    }
    else {
        lista->poczatek = nowy;
    }

    E->prev = nowy;
}

// Usuwanie elementu
void DelK(struct Lista* lista, const char* klucz) {
    struct Element* E = lista->poczatek;

    while (E != NULL) {
        if (strcasecmp_pl(E->dane.imie, klucz) == 0 ||
            strcasecmp_pl(E->dane.nazwisko, klucz) == 0) {
            if (E->prev != NULL) {
                E->prev->next = E->next;
            }
            else {
                lista->poczatek = E->next;
            }

            if (E->next != NULL) {
                E->next->prev = E->prev;
            }
            else {
                lista->koniec = E->prev;
            }

            free(E);
            return;
        }
        E = E->next;
    }
}

// Wyszukiwanie kontaktów
void WyszukajK(struct Lista* lista, const char* klucz) {
    struct Element* E = lista->poczatek;
    while (E != NULL) {
        if (strstr(E->dane.imie, klucz) || strstr(E->dane.nazwisko, klucz)) {
            printf("%s %s, Tel: %s, Grupa: %s\n",
                E->dane.imie,
                E->dane.nazwisko,
                E->dane.numer,
                E->dane.grupa);
        }
        E = E->next;
    }
}

void WyszukajGr(struct Lista* lista, const char* grupa) {
    struct Element* E = lista->poczatek;
    int znaleziono = 0;
    while (E != NULL) {
        //printf("Porównuję '%s' z '%s'\n", E->dane.grupa, grupa); 
        if (strcasecmp_pl(E->dane.grupa, grupa) == 0) {
            printf("%s %s, Tel: %s, Grupa: %s\n",
                E->dane.imie,
                E->dane.nazwisko,
                E->dane.numer,
                E->dane.grupa);
            znaleziono = 1;
        }
        E = E->next;
    }
    if (!znaleziono) {
        printf("Brak kontaktów w grupie '%s'.\n", grupa);
    }
}

// Wczytywanie kontaktów z pliku CSV
void DaneExcel(struct Lista* lista, const char* nazwa_pliku) {
    FILE* plik = fopen(nazwa_pliku, "r");
    if (plik == NULL) {
        perror("Nie można otworzyć pliku");
        return;
    }

    char linia[128];
    while (fgets(linia, sizeof(linia), plik)) {
        struct Kontakt kontakt;
        char* x = strtok(linia, ";\n");
        if (x) strcpy(kontakt.imie, x);
        x = strtok(NULL, ";\n");
        if (x) strcpy(kontakt.nazwisko, x);
        x = strtok(NULL, ";\n");
        if (x) strcpy(kontakt.numer, x);
        x = strtok(NULL, ";\n");
        if (x) strcpy(kontakt.grupa, x);

        AddEnde(lista, kontakt);
    }

    fclose(plik);
}

int main() {
    puts(" /$$   /$$           /$$                     /$$                                                       ");
    puts("| $$  /$$/          |__/                    | $$                                                       ");
    puts("| $$ /$$/   /$$$$$$$ /$$  /$$$$$$  /$$$$$$$$| $$   /$$  /$$$$$$                                        ");
    puts("| $$$$$/   /$$_____/| $$ |____  $$|____ /$$/| $$  /$$/ |____  $$                                       ");
    puts("| $$  $$  |  $$$$$$ | $$  /$$$$$$$   /$$$$/ | $$$$$$/   /$$$$$$$                                       ");
    puts("| $$\\  $$  \\____  $$| $$ /$$__  $$  /$$__/  | $$_  $$  /$$__  $$                                       ");
    puts("| $$ \\  $$ /$$$$$$$/| $$|  $$$$$$$ /$$$$$$$$| $$ \\  $$|  $$$$$$$                                       ");
    puts("|__/  \\__/|_______/ |__/ \\_______/|________/|__/  \\__/ \\_______/                                       ");
    puts("                                                                                                       ");
    puts(" /$$$$$$$$        /$$            /$$$$$$                    /$$                                        ");
    puts("|__  $$__/       | $$           /$$__  $$                  |__/                                        ");
    puts("   | $$  /$$$$$$ | $$  /$$$$$$ | $$  \\__//$$$$$$  /$$$$$$$  /$$  /$$$$$$$ /$$$$$$$$ /$$$$$$$   /$$$$$$ ");
    puts("   | $$ /$$__  $$| $$ /$$__  $$| $$$$   /$$__  $$| $$__  $$| $$ /$$_____/|____ /$$/| $$__  $$ |____  $$");
    puts("   | $$| $$$$$$$$| $$| $$$$$$$$| $$_/  | $$  \\ $$| $$  \\ $$| $$| $$         /$$$$/ | $$  \\ $$  /$$$$$$$");
    puts("   | $$| $$_____/| $$| $$_____/| $$    | $$  | $$| $$  | $$| $$| $$        /$$__/  | $$  | $$ /$$__  $$");
    puts("   | $$|  $$$$$$$| $$|  $$$$$$$| $$    |  $$$$$$/| $$  | $$| $$|  $$$$$$$ /$$$$$$$$| $$  | $$|  $$$$$$$");
    puts("   |__/ \\_______/|__/ \\_______/|__/     \\______/ |__/  |__/|__/ \\_______/|________/|__/  |__/ \\_______/");
    puts("                                                                                                      ");

    struct Lista lista;
    TworzenieListy(&lista);
    setlocale(LC_ALL, "");
#ifdef _WIN32
    SetConsoleOutputCP(CP_UTF8);
#endif
    int opcja;
    do {
        printf("\nMENU:\n");
        printf("1. Wczytaj kontakty z pliku CSV\n");
        printf("2. Wyświetl listę kontaktów\n");
        printf("3. Posortuj listę kontaktów\n");
        printf("4. Wyszukaj kontakt\n");
        printf("5. Usuń kontakt\n");
        printf("6. Dodaj nowy kontakt\n");
        printf("7. Wyczyść całą listę\n");
        printf("8. Wyświetl kontakty z jednej grupy\n");
        printf("9. Wyjście \n");
        printf("Wybierz opcję: ");
        scanf("%d", &opcja);
        getchar(); // Oczyszczanie bufora

        switch (opcja) {
        case 1:
            DaneExcel(&lista, "rozbudowana_lista.csv");
            printf("Kontakty zostały wczytane.\n");
            break;
        case 2:
            printf("Lista kontaktów:\n");
            PrintLista(&lista);
            break;
        case 3:
            printf("Wybierz metodę sortowania (0: imię, 1: nazwisko, 2: grupa): ");
            scanf("%d", &lista.metoda_sortowania);
            getchar();
            SortLista(&lista);
            printf("Lista została posortowana.\n");
            break;
        case 4: {
            char klucz[30];
            printf("Podaj imię lub nazwisko do wyszukania: ");
            fgets(klucz, sizeof(klucz), stdin);
            klucz[strcspn(klucz, "\n")] = 0;
            printf("Wyniki wyszukiwania:\n");
            WyszukajK(&lista, klucz);
            break;
        }
        case 5: {
            char klucz[30];
            printf("Podaj imię lub nazwisko kontaktu do usunięcia: ");
            fgets(klucz, sizeof(klucz), stdin);
            klucz[strcspn(klucz, "\n")] = 0; // Usuwanie znaku nowej linii
            DelK(&lista, klucz);
            printf("Kontakt został usunięty (jeśli istniał).\n");
            break;
        }
        case 6: {
            struct Kontakt nowy;
            printf("Podaj imię: ");
            fgets(nowy.imie, sizeof(nowy.imie), stdin);
            nowy.imie[strcspn(nowy.imie, "\n")] = 0;

            printf("Podaj nazwisko: ");
            fgets(nowy.nazwisko, sizeof(nowy.nazwisko), stdin);
            nowy.nazwisko[strcspn(nowy.nazwisko, "\n")] = 0;

            printf("Podaj numer telefonu: ");
            fgets(nowy.numer, sizeof(nowy.numer), stdin);
            nowy.numer[strcspn(nowy.numer, "\n")] = 0;

            printf("Podaj grupę: ");
            fgets(nowy.grupa, sizeof(nowy.grupa), stdin);
            nowy.grupa[strcspn(nowy.grupa, "\n")] = 0;

            DodajK(&lista, nowy);
            printf("Kontakt został dodany w posortowany sposób.\n");
            break;
        }
        case 7:
            FreeAL(&lista);
            printf("Lista została wyczyszczona.\n");
            break;
        case 8:
			char grupa[30]; //char zwraca jako błąd "deklaracja nie może mieć etykiety" mimo to program nie działa bez niego
            printf("Podaj nazwę grupy do wyświetlenia: ");
            fgets(grupa, sizeof(grupa), stdin);
            grupa[strcspn(grupa, "\n")] = 0;
            printf("Kontakty w grupie '%s':\n", grupa);
            WyszukajGr(&lista, grupa);
            break;
        case 9:
            printf("Zakończenie programu.\n");
            break;
        default:
            printf("Nieprawidłowa opcja.\n");
        }

    } while (opcja != 9);

    FreeAL(&lista);
    return 0;
}
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <math.h>
#include <locale.h>
#include <time.h>
#include <C:\Users\macie\source\repos\maciula\maciula\googleChart.c>
#define RESETSAMPLI 300

typedef struct {
    double A, B, C, D;
    double xmin, xmax;
} ZmSygnal;

/*
class paramety {
public:
    double A, B, C, D;
    double xmin, xmax;

    parametry() {
        this->A = 10;
    }

};
*/
/* Funkcja pobierająca parametry sygnału od użytkownika
i sprawdzająca na bieżąco poprawność wpisywanych danych*/
void Generacja(ZmSygnal* params) {
    int Warun = 0;

    while (!Warun) {
        printf("Podaj wspolczynniki A, B, C, D (liczby): ");
        if (scanf("%lf %lf %lf %lf", &params->A, &params->B, &params->C, &params->D) == 4) {
            Warun = 1; // Podano cyfry warunek = 1 koniec petli
        }
        else {
            printf("Blad: Prosze wpisac poprawne liczby.\n\n");
            while (getchar() != '\n'); // Reset wartości trzeba wpisać od nowa pętla od nowa
        }
    }

    Warun = 0;
    while (!Warun) {
        printf("Podaj granice dziedziny xmin, xmax: ");
        if (scanf("%lf %lf", &params->xmin, &params->xmax) == 2 && &params->xmin < &params->xmax) {
            Warun = 1;
        }
        else {
            printf("Blad: Prosze wpisac poprawne liczby.\n\n");
            while (getchar() != '\n'); // Reset trzeba wpisać od nowa
        }
    }
}
void Tester(ZmSygnal* params) {
    /*
        &params->A = (double)12;
    &params->B = (double)8;
    &params->C = (double)8;
    &params->D = (double)9;
    &params->xmin = (double)4;
    &params->xmax = (double)15;
    */
    //params->A = 12;
}

/* Funkcja pobierająca liczbę próbek
i sprawdzająca czy podana wartość to int>0 */
int zbierzprobki() {
    int n;
    int Warun = 0;

    while (!Warun) {
        printf("Podaj liczbe probek (liczba calkowita): ");
        if (scanf("%d", &n) == 1 && n > 0) {
            Warun = 1; //poprawnie wykonana petla
        }
        else {
            printf("Błąd: Proszę wpisać liczbę całkowitą większą od zera.\n");
            while (getchar() != '\n'); // reset
        }
    }

    return n;
}

// Funkcja generująca sygnal
void Funkcja(double* sygnal, ZmSygnal* params, int ileprobek) {
    double dX = (params->xmax - params->xmin) / (ileprobek - 1);
    for (int i = 0; i < ileprobek; i++) {
        double x = params->xmin + i * dX;
        sygnal[i] = params->A * cos((x + params->B) / sqrt(params->C)) + params->D;
    }
}

// Funkcja zapisująca sygnał do pliku excel
void saveToFile(const double* sygnal, int ileprobek, const char* filename) {
    FILE* file = fopen(filename, "w");
    if (file) {
        for (int i = 0; i < ileprobek; i++) {
            fprintf(file, "%.6f\n", sygnal[i]);
        }
        fclose(file);
        printf("Zapisano sygnal do pliku  C:/Users/macie/source/repos/maciula/maciula/%s.\n \n", filename);
    }
    else {
        printf("blad zapisu");
    }
}

// Funkcja generująca losowy szum

void zaszumianie(double* sygnal, int ileprobek, double PoziomSzumu) {
    for (int i = 0; i < ileprobek; i++) {
        double szum = ((rand() / (double)RAND_MAX) * 2.0 - 1.0) * PoziomSzumu / 100; //dzielenie przez 100 dla wygody uzytkownika
        sygnal[i] += szum / (rand() % 100 + 1); //znowu randomizuje zeby szum nie pojawiał się w kazdym (plus jeden bo inf)
    }
}
filtrowanie(const double* input, double* output, int ileprobek) {
    if (ileprobek < 2) {
        printf("Za malo probek do zastosowania filtru.\n");
        return;
    }

    output[0] = input[0];
    output[ileprobek - 1] = input[ileprobek - 1];

    for (int i = 1; i < ileprobek - 1; i++) {
        output[i] = (input[i - 1] + input[i + 1]) / 2.0;
    }
}

void logo() {
    puts(
        "###########################################################################################################\n"
        "###########################################################################################################\n\n"
        "     GGGG     EEEEEEEE  NNNN      NNN  EEEEEEEE  RRRRRRRR       AAA      TTTTTTTTT   OOOOOOOO   RRRRRRRR   \n"
        "   GGGGGGGG   EEEEEEEE  NNNNN     NNN  EEEEEEEE  RRR    RR     AAAAA     TTTTTTTTT  OOO    OOO  RRR    RR  \n"
        "   GGG        EEE       NNN NN    NNN  EEE       RRR    RR    AA   AA       TTT     OOO    OOO  RRR    RR  \n"
        "   GGG  GGG   EEEEEEE   NNN  NN   NNN  EEEEEEEE  RRRRRRRR    AAA   AAA      TTT     OOO    OOO  RRRRRRRR   \n"
        "   GGG  GGG   EEEEEEE   NNN   NN  NNN  EEEEEEEE  RRRRRR     AAA     AAA     TTT     OOO    OOO  RRRRRR     \n"
        "   GGG   GG   EEE       NNN    NN NNN  EEE       RRR  RR    AAAAAAAAAAA     TTT     OOO    OOO  RRR  RR    \n"
        "   GGGGGGGG   EEEEEEEE  NNN     NNNNN  EEEEEEEE  RRR   RR   AAA     AAA     TTT     OOO    OOO  RRR   RR   \n"
        "     GGGG     EEEEEEEE  NNN      NNNN  EEEEEEEE  RRR    RR  AAA     AAA     TTT      OOOOOOOO   RRR    RR  \n\n"
        //
        "              SSSSSS   YY     YY    GGGG    NNNN      NNN      AAA         LLL  LLL  UUU   UUU   \n"
        "             SSSSSSSS   YY   YY   GGGGGGGG  NNNNN     NNN     AAAAA        LLL LLL   UUU   UUU   \n"
        "             SSS         YY YY    GGG       NNN NN    NNN    AA   AA       LLLLLL    UUU   UUU   \n"
        "             SSSSSSSS     YYY     GGG  GGG  NNN  NN   NNN   AAA   AAA    LLLLLLL     UUU   UUU   \n"
        "                  SSS     YYY     GGG  GGG  NNN   NN  NNN  AAA     AAA  LLLLLL       UUU   UUU   \n"
        "                  SSS     YYY     GGG   GG  NNN    NN NNN  AAAAAAAAAAA  LL LLL       UUU   UUU   \n"
        "             SSSSSSSS     YYY     GGGGGGGG  NNN     NNNNN  AAA     AAA     LLLLLLLL  UUU   UUU   \n"
        "              SSSSSS      YYY       GGGG    NNN      NNNN  AAA     AAA     LLLLLLLL   UUUUUUU       M.S. 279729\n\n"
        "###########################################################################################################\n"
        "###########################################################################################################\n"

    );
}
void displayMenu() {
    printf(
        "1. Generowanie sygnalu podstawowego\n"
        "2. Wprowadzenie szumu\n"
        "3. Filtracja sygnalu\n"
        "4. Zapis sygnalu do pliku csv\n"
        "5. Zakonczenie programu\n"
        "6. Wyswietlanie wartosci\n"
        "7. Wygeneruj wykres (Googlechart)\n"
        "Wybierz opcje: ");
}

void FunkcjaPila(double* sygnal, ZmSygnal* params, int ileprobek) {
    double dX = (params->xmax - params->xmin) / (ileprobek - 1);
    for (int i = 0; i < ileprobek; i++) {
        double x = params->xmin + i * dX;
        double okres = fmod((x + params->B), params->C) / params->C;
        sygnal[i] = params->A * (2 * okres) + params->D;             // Sygnał piłokształtny niestety 
        //działa tylko dla niektorych parametrow
    }
}

int main() {
    logo();
    setlocale(LC_NUMERIC, "pl_PL.UTF-8"); // Ustawienie przecinka jako separator (psuje googlechart)

    srand(time(NULL));

    ZmSygnal params;
    double* sygnal = NULL;
    int ileprobek = RESETSAMPLI;
    int running = 1;

    while (running) {
        displayMenu();
        int choice;
        scanf("%d", &choice);

        switch (choice) {
        case 1: // Generowanie sygnału 
            Generacja(&params);
            ileprobek = zbierzprobki();
            sygnal = (double*)malloc(ileprobek * sizeof(double));
            if (sygnal) {
                int typSygnalu;
                printf("Wybierz typ sygnalu: 1-sinus, 2-Pilosztaltny: ");
                scanf("%d", &typSygnalu);
                if (typSygnalu == 1) {
                    Funkcja(sygnal, &params, ileprobek);
                }
                else if (typSygnalu == 2) {
                    FunkcjaPila(sygnal, &params, ileprobek);
                }
                else {
                    printf("Nieznany typ sygnalu.\n");
                    free(sygnal);
                    sygnal = NULL;
                }
                printf("Sygnal podstawowy zostal wygenerowany.\n\n");
            }
            else {
                printf("zrob od nowa\n\n");
            }
            break;
        case 2: // Wprowadzenie szumu
            if (sygnal) {
                double PoziomSzumu;
                do {
                    printf("Podaj poziom szumu (od 0 do 100): ");
                    scanf("%lf", &PoziomSzumu);
                    if (PoziomSzumu < 0 || PoziomSzumu > 100) {
                        printf("Poziom szumu musi być pomiędzy 0 a 100.\n");
                    }
                } while (PoziomSzumu < 0 || PoziomSzumu > 100);

                zaszumianie(sygnal, ileprobek, PoziomSzumu);
                printf("Szum zostal dodany\n\n");
            }
            else {
                printf("Najpierw wygeneruj sygnal podstawowy.\n\n");
            }
            break;
        case 3: // Filtracja 
            if (sygnal) {
                double* FSygnal = (double*)malloc(ileprobek * sizeof(double));
                if (FSygnal) {
                    filtrowanie(sygnal, FSygnal, ileprobek);
                    printf("Sygnal został przefiltrowany metodą sąsiedniego uśredniania.\n\n");

                    free(sygnal);
                    sygnal = FSygnal;
                }
                else {
                    printf("Blad alokacji pamieci dla filtru.\n\n");
                }
            }
            else {
                printf("Najpierw wygeneruj sygnal podstawowy.\n\n");
            }
            break;
        case 4: // Zapis do pliku
            if (sygnal) {
                char filename[50];
                printf("Podaj nazwe pod ktora zostanie zapisany plik: ");
                scanf("%s", filename);
                saveToFile(sygnal, ileprobek, filename);
            }
            else {
                printf("Najpierw wygeneruj sygnal podstawowy.\n\n");
            }
            break;
        case 5:
            running = 0;
            break;
        case 6:
            if (sygnal) {
                double dX = (params.xmax - params.xmin) / (ileprobek - 1);
                printf("Współrzędne dla x i y:\n");
                for (int i = 0; i < ileprobek; i++) {
                    double x = params.xmin + i * dX;
                    printf("DLA X[%f],Y=%f\n", x, sygnal[i]);
                }
            }
            else {
                printf("Najpierw wygeneruj sygnal podstawowy.\n");
            }
            break;
        case 7: //Googlechart
            setlocale(LC_NUMERIC, "C"); //naprawa formatu wyswietlania liczb dla googlechart
            googleChart(sygnal, NULL, ileprobek); // sprintf(temp, "[%d, %f]", i, sygnal[i]); //gdy nie podano dziedziny, numeruj próbki od 0
            setlocale(LC_NUMERIC, "pl_PL.UTF-8"); // Ustawienie przecinka jako separatora (mozliwy ponowny zapis w excelu)
            printf("\n");
            break;
            //case 8: //Googlechart
               //tester(&params); 
            break;
        default:
            printf("Program zakonczony\n");
        }
    }

    free(sygnal);
    printf("Program zakonczony\n");
    return 0;
}
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char* nazwa;
    int szerokosc;
    int wysokosc;
    int szarosc;
    unsigned char** pixels;
} Obraz;

//baza 
Obraz* obrazy123 = NULL;
int IloscPlikow = 0;

// Funkcja alokacji pamięci dla pikseli obrazu
int Alokacja(Obraz* war) { 
    war->pixels = (unsigned char**)malloc(war->wysokosc * sizeof(unsigned char*));
    if (!war->pixels) return 0;
    for (int i = 0; i < war->wysokosc; i++) {
        war->pixels[i] = (unsigned char*)malloc(war->szerokosc * sizeof(unsigned char));
        if (!war->pixels[i]) return 0;
    }
    return 1;
}

// Funkcja zwalniania pamięci dla obrazu
void CzyszczeniePamieci(Obraz* war) {
    if (war->pixels) {
        for (int i = 0; i < war->wysokosc; i++) {
            free(war->pixels[i]);
        }
        free(war->pixels);
    }
    free(war->nazwa);
}

// Funkcja odczytu obrazu PGM
int Odczyt(const char* filenazwa, Obraz* war) {
    FILE* file = fopen(filenazwa, "r");
    if (!file) {
        printf("Nie mozna otworzyc pliku: %s\n", filenazwa);
        return 0;
    }

    char buffer[128];
    if (!fgets(buffer, sizeof(buffer), file) || strncmp(buffer, "P2", 2) != 0) {
        printf("Nieprawidlowy format pliku.\n");
        fclose(file);
        return 0;
    }

    do { fgets(buffer, sizeof(buffer), file); } while (buffer[0] == '#');
    sscanf(buffer, "%d %d", &war->szerokosc, &war->wysokosc);
    fscanf(file, "%d", &war->szarosc);

    if (!Alokacja(war)) {
        printf("Blad alokacji pamieci.\n");
        fclose(file);
        return 0;
    }

    for (int i = 0; i < war->wysokosc; i++) {
        for (int j = 0; j < war->szerokosc; j++) {
            int pixel;
            fscanf(file, "%d", &pixel);
            war->pixels[i][j] = (unsigned char)pixel;
        }
    }
    war->nazwa = _strdup(filenazwa);
    fclose(file);
    return 1;
}

// Funkcja zapisu obrazu PGM
int Zapis(const char* filenazwa, const Obraz* war) {
    FILE* file = fopen(filenazwa, "w");
    if (!file) {
        printf("Nie mozna zapisac pliku: %s\n", filenazwa);
        return 0;
    }

    fprintf(file, "P2\n");
    fprintf(file, "# Wygenerowany przez program\n");
    fprintf(file, "%d %d\n", war->szerokosc, war->wysokosc);
    fprintf(file, "%d\n", war->szarosc);

    for (int i = 0; i < war->wysokosc; i++) {
        for (int j = 0; j < war->szerokosc; j++) {
            fprintf(file, "%d ", war->pixels[i][j]);
        }
        fprintf(file, "\n");
    }
    fclose(file);
    return 1;
}

void GenAscii(const Obraz* war) { //działa dużych obrazów nie są czytelne ze względu na ograniczenia wielkości okna cmd ale plik test.pgm pokazuje że działa
    const char* ascii_chars = " .:-=+*#%@";
    int num_chars = strlen(ascii_chars);

    printf("\nPodglad obrazu w formie ASCII Art:\n\n");
    for (int i = 0; i < war->wysokosc; i++) {
        for (int j = 0; j < war->szerokosc; j++) {
            int intensity = war->pixels[i][j] * (num_chars - 1) / war->szarosc;
            printf("%c", ascii_chars[intensity]);
        }
        printf("\n");
    }
}

// Funkcja do obrotu obrazu o 90 stopni
void rotateImage(Obraz* war) {
    unsigned char** rotated = (unsigned char**)malloc(war->szerokosc * sizeof(unsigned char*));
    for (int i = 0; i < war->szerokosc; i++) {
        rotated[i] = (unsigned char*)malloc(war->wysokosc * sizeof(unsigned char));
    }

    for (int i = 0; i < war->wysokosc; i++) {
        for (int j = 0; j < war->szerokosc; j++) {
            rotated[j][war->wysokosc - 1 - i] = war->pixels[i][j];
        }
    }
    for (int i = 0; i < war->wysokosc; i++) free(war->pixels[i]);
    free(war->pixels);

    war->pixels = rotated;
    int temp = war->szerokosc;
    war->szerokosc = war->wysokosc;
    war->wysokosc = temp;
}

// Funkcja przetwarzajaaca wybrany obraz na negatyw
void generateNegative(Obraz* war) {
    for (int i = 0; i < war->wysokosc; i++) {
        for (int j = 0; j < war->szerokosc; j++) {
            war->pixels[i][j] = war->szarosc - war->pixels[i][j];
        }
    }
}

// Funkcja stosująca filtr medianowy
void SortOkno(unsigned char* array, int size) {
    for (int i = 0; i < size - 1; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (array[j] > array[j + 1]) {
                unsigned char temp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = temp;
            }
        }
    }
}

void Fmedianowy(Obraz* war) {
    unsigned char** filtered = (unsigned char**)malloc(war->wysokosc * sizeof(unsigned char*));
    for (int i = 0; i < war->wysokosc; i++) {
        filtered[i] = (unsigned char*)malloc(war->szerokosc * sizeof(unsigned char));
    }

    for (int i = 1; i < war->wysokosc - 1; i++) {
        for (int j = 1; j < war->szerokosc - 1; j++) {
            unsigned char window[9];
            int idx = 0;

            for (int di = -1; di <= 1; di++) {
                for (int dj = -1; dj <= 1; dj++) {
                    window[idx++] = war->pixels[i + di][j + dj];
                }
            }

            SortOkno(window, 9);
            filtered[i][j] = window[4];
        }
    }

    for (int i = 0; i < war->wysokosc; i++) free(war->pixels[i]);
    free(war->pixels);

    war->pixels = filtered;
}

// Funkcje zarządzania bazą obrazów
void InsertBaza(Obraz* war) {
    Obraz* temp = realloc(obrazy123, (IloscPlikow + 1) * sizeof(Obraz));
    if (!temp) {
        printf("Brak takiego pliku (upewnij sie ze dodales .pgm na koncu pliku)\n");
        return;
    }
    obrazy123 = temp;
    obrazy123[IloscPlikow] = *war;
    IloscPlikow++;
}

void UsunZbazy(int index) {
    if (index < 0 || index >= IloscPlikow) {
        printf("Brak takiego pliku\n");
        return;
    }
    CzyszczeniePamieci(&obrazy123[index]);
    for (int i = index; i < IloscPlikow - 1; i++) {
        obrazy123[i] = obrazy123[i + 1];
    }
    Obraz* temp = realloc(obrazy123, (IloscPlikow - 1) * sizeof(Obraz));
    if (temp || IloscPlikow - 1 == 0) {
        obrazy123 = temp;
        IloscPlikow--;
    }
}

void listImages() {
    if (IloscPlikow == 0) {
        printf("Brak obrazow w bazie\n");
        return;
    }
    for (int i = 0; i < IloscPlikow; i++) {
        printf("%d: %s (%dx%d)\n", i + 1, obrazy123[i].nazwa, obrazy123[i].szerokosc, obrazy123[i].wysokosc);
    }
}
void TitleCard() {
    puts(" ____               _                                     _      ");
    puts("|  _ | _ __ _______| |___      ____ _ _ __ ______ _ _ __ (_) ___ ");
    puts("| |_) | '__|_  / _ | __| | /| / / _` | '__|_  / _` | '_ || |/ _ |");
    puts("|  __|| |   / /  __/ |_ | V  V / (_| | |   / / (_| | | | | |  __/");
    puts("|_|   |_|  /___|___||__| |_/|_/ |__,_|_|  /___|__,_|_| |_|_||___|");
    puts("   ___  _                    __            ____   ____ __  __     ");
    puts("  / _ || |__  _ __ __ _ ____/_/__      __ |  _ | / ___|  |/  |    ");
    puts(" | | | | '_ || '__/ _` |_  / _ | | /| / / | |_) | |  _| ||/| |    ");
    puts(" | |_| | |_) | | | (_| |/ / (_) | V  V /  |  __/| |_| | |  | |    ");
    puts("  |___/|_.__/|_|  |__,_/___|___/ |_/|_/   |_|    |____|_|  |_|   M.S ");
}
    // Główne menu
void showMenu() {
    printf("\nMenu glowne:\n");
    printf("1. Dodaj obraz\n");
    printf("2. Usun obraz\n");
    printf("3. Lista obrazow\n");
    printf("4. Przetwarzaj obraz\n");
    printf("5. Wyjscie\n");
}

void PrzetwarzanieObrazu(Obraz* war) {
    int choice;
    while (1) {
        printf("\nPrzetwarzanie obrazu: %s\n", war->nazwa);
        printf("1. Obrot obrazu\n");
        printf("2. Negatyw obrazu\n");
        printf("3. Filtr medianowy\n");
        printf("4. Zapisz obraz\n");
        printf("5. Wyswietl jaco obraz ASCII\n");
        printf("6. Powrot do glownego menu\n");
        printf("Wybierz opcje: ");
        scanf("%d", &choice);

        if (choice == 1) {
            rotateImage(war);
            printf("Obraz zostal obrocony 90 stopni w prawo\n");
        }
        else if (choice == 2) {
            generateNegative(war);
            printf("Negatyw obrazu został wygenerowany.\n");
        }
        else if (choice == 3) {
            Fmedianowy(war);
            printf("Nalozono filtr medianowy\n");
        }
        else if (choice == 4) {
            char filenazwa[128];
            printf("Podaj nazwe pliku do zapisu: ");
            scanf("%s", filenazwa);
            if (Zapis(filenazwa, war)) {
                printf("Obraz zapisany jako %s.\n", filenazwa);
            }
            else {
                printf("Blad zapisu obrazu.\n");
            }
        }
        else if (choice == 5) {
            GenAscii(war);
        }
        else if (choice == 6) {
            break;
        }
        else {
            printf("Nieprawidlowy wybor.\n");
        }
    }
}


int main() {
    TitleCard();
    int choice;
    while (1) {
        showMenu();
        scanf("%d", &choice);

        if (choice == 1) {
            char filenazwa[128];
            Obraz war;
            printf("Podaj nazwe pliku: ");
            scanf("%s", filenazwa);
            if (Odczyt(filenazwa, &war)) {
                InsertBaza(&war);
            }
        }
        else if (choice == 2) {
            listImages();
            int index;
            printf("Podaj numer obrazu ktory chcesz usunac: ");
            scanf("%d", &index);
            UsunZbazy(index - 1);
        }
        else if (choice == 3) {
            listImages();
        }
        else if (choice == 4) {
            listImages();
            int index;
            printf("Wybierz obraz: ");
            scanf("%d", &index);
            if (index > 0 && index <= IloscPlikow) {
                PrzetwarzanieObrazu(&obrazy123[index - 1]);
            }
            else {
                printf("Nie ma takiej pozycji\n");
            }
        }
        else if (choice == 5) {
            for (int i = 0; i < IloscPlikow; i++) {
                CzyszczeniePamieci(&obrazy123[i]);
            }
            free(obrazy123);
            break;
        }
        else {
            printf("Nie ma takiej opcji.\n");
        }
    }
    return 0;
}

