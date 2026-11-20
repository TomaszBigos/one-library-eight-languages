# Czas i pamięć na 341 tekstach konstytucji

Na czasie samej biblioteki (341 tekstów, jeden klient, mediana trzech rund) czołówkę tworzą Rust, C++, C# z JIT i Go:
0,64–0,80 czasu Javy JIT w polskim i 0,69–0,80 w malgaskim, w obu seriach (`../data/matrix.csv`). Różnice w tej
czwórce są w granicach szumu maszyny i kolejność zmienia się między seriami; trwała jest tylko przewaga Go w malgaskim
(0,69–0,70 wobec 0,76–0,80 pozostałych). Obraz natywny Javy (GraalVM z profilem kompilacji) ma 0,91–0,98 czasu Javy
JIT, Kotlin i Scala w obrazie natywnym zrównują się z Javą (0,99–1,12), na JVM są wolniejsze (Kotlin 1,09–1,19,
Scala 1,16–1,32), a TypeScript jest 5,7–9,1 razy wolniejszy (`../data/matrix.csv`). Profil kompilacji (PGO) liczy się
w obrazach GraalVM (bez niego 1,16–1,39 czasu Javy), w Ruście nie zmienia nic (0,64–0,80 z profilem i bez), a w Go
daje 0,69–0,70 wobec 0,83 w malgaskim, w polskim zaś wersje z profilem i bez zamieniły się miejscami między seriami. Start biblioteki z wytrenowanymi detektorami wczytanymi
z dysku trwa w Ruście 0,6–1,0 s, w Javie JIT 3,4–4,1 s, w Scali JIT 4,9–5,9 s; szczyt pamięci procesu to 180–221 MB
dla obrazów natywnych JVM, 212–263 MB dla Rusta i Go, 281–310 MB dla C++, 318–362 MB dla Javy JIT i do 601 MB dla Buna
(`../data/matrix.csv`). Wniosek: cztery języki dzielą czołówkę około 0,7 czasu Javy, a obraz natywny daje czas Javy
przy ok. 0,6 jej pamięci i 0,5–0,6 jej czasu startu. Przepustowość w dok/s obrazów natywnych JVM zaniża dodatkowo serializacja
raportu w programie pomiarowym, nie biblioteka (`../data/README.md`).
