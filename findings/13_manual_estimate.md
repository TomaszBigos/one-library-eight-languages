# Ile zająłby port bez AI (szacunek)

> To szacunek z jawnych założeń, nie pomiar. Założenia są przedziałami, żeby było widać, jak bardzo wynik od nich
> zależy (`../data/manual_estimate.csv`).

## Założenia

Doświadczony programista znający oba języki, z działającą referencją w Javie, testami referencyjnymi i gotową bramką
równoważności, 8 godzin dziennie:
- 100–250 linii kodu biblioteki dziennie, razem z czytaniem oryginału, kompilacją i usuwaniem błędów;
- 150–300 linii testów dziennie;
- 1–3 dni na każdą własną rundę optymalizacji (profil, zmiana, pomiar A/B na zajętej maszynie, bramka), a dla Kotlina
  i Scali 0,5–1 dnia na przeniesienie każdej z 13 wspólnych rund z Javy;
- ręczne uruchamianie przy tej samej liczbie uruchomień co AI (`../data/ai_work.csv`): 3–10 minut uwagi człowieka na
  każde budowanie albo przebieg testów (start, czekanie, czytanie wyniku) i 10–20 minut na każdy przebieg bramki.

Linie kodu i testów pochodzą z `../data/port_cost.csv`, rundy i liczba uruchomień z `../data/ai_work.csv`.

## Wynik

| port | dni robocze człowieka | aktywne godziny AI | ile razy więcej godzin |
|---|---|---|---|
| Kotlin | 40–93 | 4,1 | 77–182 |
| Scala | 34–81 | 5,4 | 51–120 |
| Rust | 47–118 | 9,0 | 41–105 |
| C# | 41–108 | 7,1 | 46–121 |
| Go | 42–107 | 2,7 | 125–316 |
| C++ | 29–73 | 4,8 | 49–121 |
| TypeScript | 19–48 | 2,7 | 57–141 |

(`../data/manual_estimate.csv`)

Siedem portów razem to 252–627 dni roboczych, czyli ok. 1–2,5 roku pracy jednej osoby, wobec ok. 36 godzin aktywnej
pracy AI nad tymi portami (`../data/port_cost.csv`). Największą część szacunku stanowi samo pisanie kodu. Ręczne
uruchamianie budowań, testów i bramki to razem 9–23 dni, a pisanie testów 33–65 dni (`../data/manual_estimate.csv`).
Go wychodzi najdrożej dla człowieka, bo ma najwięcej linii (8205), a AI przepisało go najszybciej.

## Czego szacunek nie obejmuje

- czasu człowieka nadzorującego AI: decyzji, przeglądu wyników, zgód; nie był mierzony;
- nauki języka i projektu odtworzenia Javy (silnik wyrażeń regularnych, normalizacja Unicode), które w Ruście trzeba
  było wymyślić, a nie tylko przepisać;
- czasu maszyny na pomiary, bo jest taki sam przy pracy ręcznej i z AI;
- tego, że człowiek uruchamiałby budowanie rzadziej, ale każda iteracja trwałaby dłużej;
- oryginału w Javie: to była praca badawcza, a nie port.

**Wniosek:** nawet przy optymistycznych założeniach różnica to dwa rzędy wielkości godzin. Warunkiem była gotowa
referencja i automatyczna bramka. Bez nich AI nie miałoby jak sprawdzić portu, a człowiek musiałby te same
sprawdzenia zrobić ręcznie.
