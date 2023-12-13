| imie  | nazwisko    | nr indeksu |
|-------|-------------|------------|
| Dawid | Mielewczyk | 189637     |
$189637$ $mod$ $8 = 5$
#### 1. Wyświetl zestawienie, które przedstawi ile portów występuje w danym rejonie (obszar, liczba_portów). Wynik posortuj względem rosnącej liczby portów.

- Zapytanie:
```sql
SELECT obszar, COUNT(port_id) AS liczba_portow
FROM port
GROUP BY obszar
ORDER BY liczba_portow ASC;
```
- Surowa odpowiedź:
```
    obszar     | liczba_portow
---------------+---------------
 Wegorzewo     |             1
 Zalew Wislany |             1
 Mikolajki     |             2
 Jeziorak      |             2
 Ruciane-Nida  |             2
 Gizycko       |             6
               |             6
(7 rows)
```
- Tabela markdown:

| obszar       | liczba_portow |
|--------------|---------------|
| Wegorzewo    | 1             |
| Zalew Wislany| 1             |
| Mikolajki    | 2             |
| Jeziorak     | 2             |
| Ruciane-Nida | 2             |
| Gizycko      | 6             |
|              | 6             |

#### 2. Wyświetl w formie jednej tabeli zestawienie wypożyczeń wszystkich jachtów, dla których data wynajmu mieści się między 10 września a 10 października 2022 (nazwisko, imię, model, nazwa, miasto, obszar, data_wynajmu).

- Zapytanie:
```sql
SELECT 
    kl.nazwisko, 
    kl.imie, 
    mo.nazwa AS model, 
    ja.nazwa, 
    po.miasto, 
    po.obszar, 
    wy.data_wynajmu
FROM 
    wynajem wy
JOIN 
    klient kl ON wy.kto_wynajal = kl.klient_id
JOIN 
    jacht ja ON wy.co_wynajal = ja.jacht_id
JOIN 
    model mo ON ja.model = mo.model_id
JOIN 
    port po ON wy.port_wyplyniecia = po.port_id
WHERE 
    wy.data_wynajmu BETWEEN '2022-09-10' AND '2022-10-10'
ORDER BY 
    wy.data_wynajmu;
```
- Surowa odpowiedź:
```
  nazwisko  |   imie    |  model  |   nazwa    |    miasto    |  obszar   |    data_wynajmu
------------+-----------+---------+------------+--------------+-----------+---------------------
 Cikowski   | Stanislaw | Focus   | Slonecznik | Mikolajki    |           | 2022-09-12 00:00:00
 Mroz       | Agata     | Focus   | Slonecznik | Ruciane-Nida |           | 2022-09-15 00:00:00
 Golimowska | Maria     | Aquatic | Drapacz    | Bogaczewo    | Gizycko   | 2022-09-21 00:00:00
 Gorgon     | Jerzy     | Cobra   | Lopian     | Mikolajki    |           | 2022-09-25 00:00:00
 Deyna      | Kazimierz | Antila  | Anemon     | Stare Sady   | Mikolajki | 2022-10-02 00:00:00
(5 rows)
```
- Tabela markdown:

| nazwisko   | imie      | model   | nazwa      | miasto       | obszar | data_wynajmu       |
|------------|-----------|---------|------------|--------------|--------|--------------------|
| Cikowski   | Stanislaw | Focus   | Slonecznik | Mikolajki    |        | 2022-09-12 00:00:00|
| Mroz       | Agata     | Focus   | Slonecznik | Ruciane-Nida |        | 2022-09-15 00:00:00|
| Golimowska | Maria     | Aquatic | Drapacz    | Bogaczewo    | Gizycko| 2022-09-21 00:00:00|
| Gorgon     | Jerzy     | Cobra   | Lopian     | Mikolajki    |        | 2022-09-25 00:00:00|
| Deyna      | Kazimierz | Antila  | Anemon     | Stare Sady   | Mikolajki | 2022-10-02 00:00:00|

#### 3. Napisz zapytanie, które wygeneruje zestawienie wszystkich wynajętych i niezwróconych do portu jachtów w postaci (imie, nazwisko, nazwa, model, liczba_dni, cena_doba razy liczba_dni). Za liczbę dni przyjmij czas od wynajęcia do dnia dzisiejszego. Zapytanie powinno wyświetlić WSZYSTKICH klientów (dla tych, którzy wynajęli i nie zwrócili uzupełnione będą wszystkie kolumny, dla pozostałych jedynie imie i nazwisko - skorzystaj z RIGHT i LEFT JOIN). Wynik posortuj alfabetycznie a-z wg nazwiska klienta
- Zapytanie:
```sql
SELECT 
    kl.imie, 
    kl.nazwisko, 
    ja.nazwa, 
    mo.nazwa AS model, 
    CASE 
        WHEN wy.data_powrotu_do_portu IS NULL THEN EXTRACT(DAY FROM current_date - wy.data_wynajmu) 
        ELSE NULL 
    END AS liczba_dni,
    CASE 
        WHEN wy.data_powrotu_do_portu IS NULL THEN EXTRACT(DAY FROM current_date - wy.data_wynajmu) * ja.cena_doba 
        ELSE NULL 
    END AS koszt_calkowity
FROM 
    wynajem wy
RIGHT JOIN 
    klient kl ON kl.klient_id = wy.kto_wynajal
LEFT JOIN 
    jacht ja ON wy.co_wynajal = ja.jacht_id
LEFT JOIN 
    model mo ON ja.model = mo.model_id
WHERE 
    wy.data_powrotu_do_portu IS NULL OR wy.kto_wynajal IS NULL
ORDER BY 
    kl.nazwisko ASC;
```
- Surowa odpowiedź:
```
    imie     |     nazwisko     |   nazwa    |  model   | liczba_dni | koszt_calkowity
-------------+------------------+------------+----------+------------+-----------------
 Zygmunt     | Anczok           |            |          |            |
 Jan         | Banas            |            |          |            |
 Mieczyslaw  | Batsch           |            |          |            |
 Izabela     | Belcik           |            |          |            |
 Stanislaw   | Cikowski         | Zlociszek  | Mors     |        273 |        42315.00
 Leslaw      | Cmikiewicz       |            |          |            |
 Krystyna    | Czajkowska       |            |          |            |
 Marian      | Einbacher        | Zlociszek  | Mors     |         59 |         9145.00
 Robert      | Gadocha          |            |          |            |
 Malgorzata  | Glinka           |            |          |            |
 Krystyna    | Jakubowska       |            |          |            |
 Danuta      | Kordaczuk-Wagner |            |          |            |
 Hubert      | Kostka           |            |          |            |
 Jerzy       | Kraska           | Lilia      | Joker    |          9 |         1845.00
 Krystyna    | Krupa            | Rumianek   | Cobra    |        424 |        84800.00
 Waclaw      | Kuchar           |            |          |            |
 Dominika    | Lesniewicz       |            |          |            |
 Maria       | Liktoras         | Rumianek   | Cobra    |         55 |        11000.00
 Stefan      | Loth             |            |          |            |
 Wlodzimierz | Lubanski         |            |          |            |
 Artur       | Marczewski       |            |          |            |
 Jadwiga     | Marko            |            |          |            |
 Zygmunt     | Maszczyk         |            |          |            |
 Stanislaw   | Mielech          |            |          |            |
 Agata       | Mroz             | Slonecznik | Focus    |        454 |        40406.00
 Malgorzata  | Niemczyk-Wolska  |            |          |            |
 Imre        | Pozsonyi         | Tulipan    | Janmor   |         55 |        10945.00
 Aleksandra  | Przybysz         |            |          |            |
 Jadwiga     | Rutkowska        |            |          |            |
 Katarzyna   | Skowronska       |            |          |            |
 Magdalena   | Sliwa            | Chaber     | Janmor   |        129 |        22962.00
 Maria       | sliwka           |            |          |            |
 Leon        | Sperling         |            |          |            |
 Zdzislaw    | Styczen          | Jastrun    | Mellody  |        310 |        44950.00
 Dorota      | Swieniewicz      |            |          |            |
 Tadeusz     | Synowiec         | Szarota    | Nautiner |         37 |         5180.00
 Zofia       | Szczesniewska    |            |          |            |
 Antoni      | Szymanowski      | Nasturcja  | Joker    |          9 |         1044.00
(38 rows)
```
- Tabela markdown:

| imie       | nazwisko           | nazwa      | model    | liczba_dni | koszt_calkowity |
|------------|--------------------|------------|----------|------------|-----------------|
| Zygmunt    | Anczok             |            |          |            |                 |
| Jan        | Banas              |            |          |            |                 |
| Mieczyslaw | Batsch             |            |          |            |                 |
| Izabela    | Belcik             |            |          |            |                 |
| Stanislaw  | Cikowski           | Zlociszek  | Mors     | 273        | 42315.00        |
| Leslaw     | Cmikiewicz         |            |          |            |                 |
| Krystyna   | Czajkowska         |            |          |            |                 |
| Marian     | Einbacher          | Zlociszek  | Mors     | 59         | 9145.00         |
| Robert     | Gadocha            |            |          |            |                 |
| Malgorzata | Glinka             |            |          |            |                 |
| Krystyna   | Jakubowska         |            |          |            |                 |
| Danuta     | Kordaczuk-Wagner   |            |          |            |                 |
| Hubert     | Kostka             |            |          |            |                 |
| Jerzy      | Kraska             | Lilia      | Joker    | 9          | 1845.00         |
| Krystyna   | Krupa              | Rumianek   | Cobra    | 424        | 84800.00        |
| Waclaw     | Kuchar             |            |          |            |                 |
| Dominika   | Lesniewicz         |            |          |            |                 |
| Maria      | Liktoras           | Rumianek   | Cobra    | 55         | 11000.00        |
| Stefan     | Loth               |            |          |            |                 |
| Wlodzimierz| Lubanski           |            |          |            |                 |
| Artur      | Marczewski         |            |          |            |                 |
| Jadwiga    | Marko              |            |          |            |                 |
| Zygmunt    | Maszczyk           |            |          |            |                 |
| Stanislaw  | Mielech            |            |          |            |                 |
| Agata      | Mroz               | Slonecznik | Focus    | 454        | 40406.00        |
| Malgorzata | Niemczyk-Wolska    |            |          |            |                 |
| Imre       | Pozsonyi           | Tulipan    | Janmor   | 55         | 10945.00        |
| Aleksandra | Przybysz           |            |          |            |                 |
| Jadwiga    | Rutkowska          |            |          |            |                 |
| Katarzyna  | Skowronska         |            |          |            |                 |
| Magdalena  | Sliwa              | Chaber     | Janmor   | 129        | 22962.00        |
| Maria      | Sliwka             |            |          |            |                 |
| Leon       | Sperling           |            |          |            |                 |
| Zdzislaw   | Styczen            | Jastrun    | Mellody  | 310        | 44950.00        |
| Dorota     | Swieniewicz        |            |          |            |                 |
| Tadeusz    | Synowiec           | Szarota    | Nautiner | 37         | 5180.00         |
| Zofia      | Szczesniewska      |            |          |            |                 |
| Antoni     | Szymanowski        | Nasturcja  | Joker    | 9          | 1044.00         |
