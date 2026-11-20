# Ręczny asembler i SIMD wobec kompilatora

W C++ jądro odległości edycyjnej z progiem 2 (200 000 par polskich słów, 15 rund w przeplocie) napisane ręcznie
w asemblerze miało medianę 0,93 i 0,70 czasu pierwszej wersji C++ w dwóch seriach, a ta sama pętla przepisana w C++
„jak jądro” 0,89 i 0,84; najszybsze pojedyncze rundy obu wersji ręcznych są prawie równe (101 ns asemblera wobec 102
i 117 ns C++) (`../data/kernels.csv`). Listing kompilatora pokazał przyczynę różnicy: pierwsza wersja C++ po każdym
zapisie od nowa czytała wskaźnik danych, bo kompilator nie umiał dowieść, że zapis go nie zmienia; osobnej funkcji
asemblerowej nie da się przy tym wkleić w pętlę wywołującą. W Ruście kod z kompilatora potrzebował 100–102 ns na tych
samych parach (ta sama suma kontrolna), mniej niż każda wersja C++, a wstawka `asm!` wewnątrz funkcji dała 0,86–0,87
tego czasu, tylko za cenę kodu na jedną architekturę, więc zostaje wyłączona domyślnie (`../data/kernels.csv`).
Instrukcje wektorowe w porównaniu Hamminga (osiem pozycji jednym wektorem) dały 0,23 czasu pętli skalarnej w C++,
0,21 w Ruście i 0,19–0,31 w C#, gdzie rozrzut pochodzi z samej pętli skalarnej JIT (1273–1877 ns); w Ruście dopiero
po dopisaniu `#[inline]` funkcji z innej paczki, bez którego zostawało prawdziwe dzielenie na każdą parę. Wniosek:
ręczny kod był przede wszystkim narzędziem diagnostycznym, bo pokazywał, czego kompilator nie zrobił, a poprawka
w zwykłym kodzie dawała większość zysku; trwały zysk dała dopiero wektoryzacja.
