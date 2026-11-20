# Koszt wytworzenia portów

Biblioteka w ośmiu językach ma od ok. 4 tys. (TypeScript) do ok. 8 tys. linii kodu (Go), licząc rdzeń i oba pipeline'y
bez pustych linii i komentarzy (`../data/port_cost.csv`). Java, oryginał, pochłonęła najwięcej pracy AI, ale ta liczba
obejmuje cały rozwój algorytmu, nie samo przepisywanie; każdy kolejny port korzystał z gotowej bramki równoważności,
testów referencyjnych i narzędzi, więc późniejsze porty są tańsze z powodu kolejności, nie języka. Port na C++, pisany
z gotowego portu Rusta, zajął 4,8 godziny aktywnej pracy AI (`../data/port_cost.csv`), a tempo wyznaczała bramka,
nie kompilator. Kolumny pracy AI to przybliżenie
z przypisania wywołań narzędzi do plików; dla Javy to dolna granica, bo najstarsze zapisy sesji nie są już dostępne.
Wniosek: przy referencji i bramce bit w bit przepisanie biblioteki na nowy język jest pracą na godziny, a nie tygodnie.
