// Nowy plik: bot.h
#pragma once
#include "gracz.h"
#include <string>

class Bot : public Gracz {
private:
    int limitPunktow;

public:
    enum TypBota { RYZYKUJACY, NORMALNY, OSTROZNY };

    Bot(std::string imie, TypBota typ);

    bool chceDobierac() const override;
};

// Nowy plik: bot.cpp
#include "bot.h"

Bot::Bot(std::string imie, TypBota typ) : Gracz(imie) {
    switch(typ) {
        case RYZYKUJACY: limitPunktow = 19; break;
        case NORMALNY: limitPunktow = 17; break;
        case OSTROZNY: limitPunktow = 15; break;
    }
}

bool Bot::chceDobierac() const {
    return ilePunktow() < limitPunktow;
}

// --- Modyfikacja kasyno.h ---
// Dodaj #include "bot.h"
#include "bot.h"

// --- Modyfikacja kasyno.cpp ---
// W funkcji inicjalizującej graczy dodaj tworzenie Botów, np:
// zamiast tylko ludzi, twórz też Boty:

// przykładowo (pseudo):
if (czyBot) { // zakładamy, że masz flagę lub możesz losować
    gracze.push_back(new Bot("Bot1", Bot::NORMALNY));
} else {
    gracze.push_back(new Gracz("ImieGracza"));
}

// --- Modyfikacja main.cpp ---
// Upewnij się, że podczas tworzenia gry możesz dodawać Boty do gry,
// np. z losowym typem bota (ryzykujący, normalny, ostrożny).

// Przyklady tworzenia botów w main.cpp:
Kasyno kasyno;
kasyno.dodajGracza(new Bot("Bot1", Bot::RYZYKUJACY));
kasyno.dodajGracza(new Bot("Bot2", Bot::NORMALNY));
kasyno.dodajGracza(new Bot("Bot3", Bot::OSTROZNY));

// a potem normalnie startujesz gre.
