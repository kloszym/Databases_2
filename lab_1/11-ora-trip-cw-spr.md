

# Oracle PL/Sql

widoki, funkcje, procedury, triggery
ćwiczenie

---


Imiona i nazwiska autorów : Szymon Kłodowski, Adrian Krawczyk

---

# Tabele

![table](ora-trip1-0.png)


- `Trip`  - wycieczki
	- `trip_id` - identyfikator, klucz główny
	- `trip_name` - nazwa wycieczki
	- `country` - nazwa kraju
	- `trip_date` - data
	- `max_no_places` -  maksymalna liczba miejsc na wycieczkę
- `Person` - osoby
	- `person_id` - identyfikator, klucz główny
	- `firstname` - imię
	- `lastname` - nazwisko


- `Reservation`  - rezerwacje/bilety na wycieczkę
	- `reservation_id` - identyfikator, klucz główny
	- `trip_id` - identyfikator wycieczki
	- `person_id` - identyfikator osoby
	- `status` - status rezerwacji
		- `N` – New - Nowa
		- `P` – Confirmed and Paid – Potwierdzona  i zapłacona
		- `C` – Canceled - Anulowana
- `Log` - dziennik zmian statusów rezerwacji 
	- `log_id` - identyfikator, klucz główny
	- `reservation_id` - identyfikator rezerwacji
	- `log_date` - data zmiany
	- `status` - status


```sql
create sequence s_person_seq  
   start with 1  
   increment by 1;

create table person  
(  
  person_id int not null
      constraint pk_person  
         primary key,
  firstname varchar(50),  
  lastname varchar(50)
)  

alter table person  
    modify person_id int default s_person_seq.nextval;
   
```


```sql
create sequence s_trip_seq  
   start with 1  
   increment by 1;

create table trip  
(  
  trip_id int  not null
     constraint pk_trip  
         primary key, 
  trip_name varchar(100),  
  country varchar(50),  
  trip_date date,  
  max_no_places int
);  

alter table trip 
    modify trip_id int default s_trip_seq.nextval;
```


```sql
create sequence s_reservation_seq  
   start with 1  
   increment by 1;

create table reservation  
(  
  reservation_id int not null
      constraint pk_reservation  
         primary key, 
  trip_id int,  
  person_id int,  
  status char(1)
);  

alter table reservation 
    modify reservation_id int default s_reservation_seq.nextval;


alter table reservation  
add constraint reservation_fk1 foreign key  
( person_id ) references person ( person_id ); 
  
alter table reservation  
add constraint reservation_fk2 foreign key  
( trip_id ) references trip ( trip_id );  
  
alter table reservation  
add constraint reservation_chk1 check  
(status in ('N','P','C'));

```


```sql
create sequence s_log_seq  
   start with 1  
   increment by 1;


create table log  
(  
    log_id int not null
         constraint pk_log  
         primary key,
    reservation_id int not null,  
    log_date date not null,  
    status char(1)
);  

alter table log 
    modify log_id int default s_log_seq.nextval;
  
alter table log  
add constraint log_chk1 check  
(status in ('N','P','C')) enable;
  
alter table log  
add constraint log_fk1 foreign key  
( reservation_id ) references reservation ( reservation_id );
```


---
# Dane


Należy wypełnić  tabele przykładowymi danymi 
- 4 wycieczki
- 10 osób
- 10  rezerwacji

Dane testowe powinny być różnorodne (wycieczki w przyszłości, wycieczki w przeszłości, rezerwacje o różnym statusie itp.) tak, żeby umożliwić testowanie napisanych procedur.

W razie potrzeby należy zmodyfikować dane tak żeby przetestować różne przypadki.


```sql
-- trip
insert into trip(trip_name, country, trip_date, max_no_places)
values ('Wycieczka do Paryza', 'Francja', to_date('2023-09-12', 'YYYY-MM-DD'), 5);

insert into trip(trip_name, country, trip_date,  max_no_places)
values ('Piekny Krakow', 'Polska', to_date('2025-05-03','YYYY-MM-DD'), 3);

insert into trip(trip_name, country, trip_date,  max_no_places)
values ('Znow do Francji', 'Francja', to_date('2025-05-01','YYYY-MM-DD'), 2);

insert into trip(trip_name, country, trip_date,  max_no_places)
values ('Hel', 'Polska', to_date('2025-05-01','YYYY-MM-DD'),  2);

-- person
insert into person(firstname, lastname)
values ('Jan', 'Nowak');

insert into person(firstname, lastname)
values ('Jan', 'Kowalski');

insert into person(firstname, lastname)
values ('Jan', 'Nowakowski');

insert into person(firstname, lastname)
values  ('Novak', 'Nowak');

insert into person(firstname, lastname)
values ('Adrian', 'Nowak');

insert into person(firstname, lastname)
values ('Adrian', 'Kowalski');

insert into person(firstname, lastname)
values ('Adrian', 'Nowakowski');

insert into person(firstname, lastname)
values  ('Szymon', 'Nowak');

insert into person(firstname, lastname)
values ('Szymon', 'Nowakowski');

insert into person(firstname, lastname)
values ('Szymon', 'Kowalski');

-- reservation
-- trip1
insert  into reservation(trip_id, person_id, status)
values (1, 1, 'P');

insert into reservation(trip_id, person_id, status)
values (1, 2, 'N');

insert into reservation(trip_id, person_id, status)
values (1, 3, 'C');

-- trip 2
insert into reservation(trip_id, person_id, status)
values (2, 4, 'P');

insert into reservation(trip_id, person_id, status)
values (2, 5, 'C');

insert into reservation(trip_id, person_id, status)
values (2, 6, 'N');

-- trip 3
insert into reservation(trip_id, person_id, status)
values (3, 7, 'P');

insert into reservation(trip_id, person_id, status)
values (3, 8, 'C');


-- trip 4
insert into reservation(trip_id, person_id, status)
values (2, 4, 'P');

insert into reservation(trip_id, person_id, status)
values (2, 4, 'N');
```

proszę pamiętać o zatwierdzeniu transakcji

---
# Zadanie 0 - modyfikacja danych, transakcje

Należy zmodyfikować model danych tak żeby rezerwacja mogła dotyczyć kilku miejsc/biletów na wycieczkę
- do tabeli reservation należy dodać pole
	- no_tickets
- do tabeli log należy dodac pole
	- no_tickets
	
Należy zmodyfikować zestaw danych testowych

Należy przeprowadzić kilka eksperymentów związanych ze wstawianiem, modyfikacją i usuwaniem danych
oraz wykorzystaniem transakcji

Skomentuj dzialanie transakcji. Jak działa polecenie `commit`, `rollback`?.
Co się dzieje w przypadku wystąpienia błędów podczas wykonywania transakcji? Porównaj sposób programowania operacji wykorzystujących transakcje w Oracle PL/SQL ze znanym ci systemem/językiem MS Sqlserver T-SQL

pomocne mogą być materiały dostępne tu:
https://upel.agh.edu.pl/mod/folder/view.php?id=311899
w szczególności dokument: `1_ora_modyf.pdf`

### Dodanie kolumn + aktualizacja danych

```sql

-- przyklady, kod, zrzuty ekranów, komentarz ...

--Zmiany w tabelach Reservation i Log

alter table RESERVATION
    add no_tickets INTEGER;

alter table LOG
    add no_tickets INTEGER;


--Modyfikacja zestawu danych

SET TRANSACTION READ WRITE;

UPDATE RESERVATION
    SET NO_TICKETS = 2
    WHERE RESERVATION_ID = 1;

UPDATE RESERVATION
    SET NO_TICKETS = 1
    WHERE RESERVATION_ID = 2;

UPDATE RESERVATION
    SET NO_TICKETS = 3
    WHERE RESERVATION_ID = 3;

UPDATE RESERVATION
    SET NO_TICKETS = 1
    WHERE RESERVATION_ID = 4;

UPDATE RESERVATION
    SET NO_TICKETS = 2
    WHERE RESERVATION_ID = 5;

UPDATE RESERVATION
    SET NO_TICKETS = 5
    WHERE RESERVATION_ID = 6;

UPDATE RESERVATION
    SET NO_TICKETS = 1
    WHERE RESERVATION_ID = 7;

UPDATE RESERVATION
    SET NO_TICKETS = 3
    WHERE RESERVATION_ID = 8;

UPDATE RESERVATION
    SET NO_TICKETS = 1
    WHERE RESERVATION_ID = 9;

UPDATE RESERVATION
    SET NO_TICKETS = 4
    WHERE RESERVATION_ID = 10;

COMMIT

```
### Kilka eksperymentów związanych ze wstawianiem, modyfikacją i usuwaniem danych oraz wykorzystaniem transakcji

**1. Dodanie rezerwacji i jej anulowanie w ramach jednej transakcji**

```sql
SET TRANSACTION READ WRITE;

INSERT INTO reservation (trip_id, person_id, status, no_tickets) VALUES (1, 3, 'N', 1);

ROLLBACK;
```

Zdjęcie pokazujące zmiany w bazie danych:

![0_ex_1](zad_0_przyklad_1.png)


**2. Zmiana statusu rezerwacji i zapis do loga**

```sql
SET TRANSACTION READ WRITE;

UPDATE reservation SET status = 'P' WHERE reservation_id = 2;

INSERT INTO log (reservation_id, log_date, status, no_tickets)
    SELECT RESERVATION_ID, SYSDATE, STATUS, NO_TICKETS FROM RESERVATION WHERE RESERVATION_ID = 2;

COMMIT;
```
Zdjęcie pokazujące zmiany w bazie danych:

![0_ex_2](zad_0_przyklad_2.png)


**3. Przykład wycofania transakcji w przypadku błędu (próba dodania więcej biletów niż dostępnych miejsc)**

```sql

SET TRANSACTION READ WRITE;

DECLARE
    v_max_places INT;
    v_reserved INT;
    v_number_of_tickets INT;

BEGIN
    v_number_of_tickets := 2;
    SELECT max_no_places INTO v_max_places FROM trip WHERE trip_id = 1;
    SELECT COALESCE(SUM(no_tickets), 0) INTO v_reserved FROM reservation WHERE trip_id = 1 AND status != 'C';

    IF v_reserved + v_number_of_tickets > v_max_places THEN
        ROLLBACK;
    ELSE
        INSERT INTO reservation (trip_id, person_id, status, no_tickets) VALUES (1, 5, 'N', v_number_of_tickets);
        COMMIT;
    END IF;
END;

```
Zdjęcie pokazujące zmiany w bazie danych:

![0_ex_3](zad_0_przyklad_3.png)
Wysoki indeks nowej rezerwacji spowodowany jest wcześniejszą próbą dodania przykładowych danych do tabeli, które się zdublowały niestety

### Transkacje - co i jak

&nbsp;&nbsp;&nbsp;&nbsp;Transakcja w bazie danych to zbiór operacji, które są wykonywane jako jedna, niepodzielna jednostka. Oznacza to, że wszystkie zmiany dokonane w ramach transakcji muszą zostać zatwierdzone (commit) lub wycofane (rollback) w przypadku błędu.

&nbsp;&nbsp;&nbsp;&nbsp;```COMMIT``` – zatwierdza wszystkie operacje wykonane w ramach transakcji, czyniąc zmiany trwałymi. Po wykonaniu COMMIT nie można już cofnąć zmian.
&nbsp;&nbsp;&nbsp;&nbsp;```ROLLBACK``` – cofa wszystkie operacje wykonane w bieżącej transakcji, przywracając stan bazy sprzed jej rozpoczęcia.

&nbsp;&nbsp;&nbsp;&nbsp;Jeśli w trakcie wykonywania transakcji wystąpi błąd wykonuje się automatyczne wycofanie (rollback). Możemy oczywiście założyć możliwość takiego błędu, złapać go i zrobić co chcemy wtedy.

&nbsp;&nbsp;&nbsp;&nbsp;*Oracle PL/SQL* oraz *MS Sqlserver T-SQL* mają podobne mechanizmy transakcyjne, ale różnią się składnią i sposobem zarządzania sesjami oraz błędami. Oracle domyślnie zaczyna transakcję automatycznie, podczas gdy w SQL Server trzeba ją jawnie rozpocząć.

---
# Zadanie 1 - widoki


Tworzenie widoków. Należy przygotować kilka widoków ułatwiających dostęp do danych. Należy zwrócić uwagę na strukturę kodu (należy unikać powielania kodu)

Widoki:
-   `vw_reservation`
	- widok łączy dane z tabel: `trip`,  `person`,  `reservation`
	- zwracane dane:  `reservation_id`,  `country`, `trip_date`, `trip_name`, `firstname`, `lastname`, `status`, `trip_id`, `person_id`, `no_tickets`
- `vw_trip` 
	- widok pokazuje liczbę wolnych miejsc na każdą wycieczkę
	- zwracane dane: `trip_id`, `country`, `trip_date`, `trip_name`, `max_no_places`, `no_available_places` (liczba wolnych miejsc)
-  `vw_available_trip`
	- podobnie jak w poprzednim punkcie, z tym że widok pokazuje jedynie dostępne wycieczki (takie które są w przyszłości i są na nie wolne miejsca)


Proponowany zestaw widoków można rozbudować wedle uznania/potrzeb
- np. można dodać nowe/pomocnicze widoki, funkcje
- np. można zmienić def. widoków, dodając nowe/potrzebne pola

# Zadanie 1  - rozwiązanie

**Dla ```vw_reservation```**:
```sql

create view vw_reservation
as
       SELECT reservation_id, country, trip_date, trip_name, firstname, lastname, status, t.trip_id, p.person_id, no_tickets
       FROM TRIP t
       JOIN RESERVATION r ON t.TRIP_ID = r.TRIP_ID
       JOIN PERSON p ON r.PERSON_ID = p.PERSON_ID
       ORDER BY reservation_id, t.trip_id;

commit;

```
A rezultatem uruchomienia tego widoku jest:

![1_ex_1](zad_1_przyklad_1.png)

**Dla ```vw_trip```:**
```sql
create view vw_trip
as
    SELECT UNIQUE t.trip_id, country, trip_date, trip_name, max_no_places, (max_no_places - NVL(SUM(r.no_tickets), 0)) as no_available_places
    FROM trip t
    LEFT JOIN reservation r ON t.trip_id = r.trip_id AND status != 'C'
    GROUP BY t.trip_id, country, trip_date, trip_name, max_no_places, max_no_places;
    
commit;
```
A rezultatem uruchomienia tego widoku jest:

![1_ex_2](zad_1_przyklad_2.png)

**Dla ```vw_available_trip```:**
```sql
create view vw_available_trip
as
    SELECT *
    FROM vw_trip
    WHERE no_available_places > 0;
    
commit;
```
A rezultatem uruchomienia tego widoku jest:

![1_ex_3](zad_1_przyklad_3.png)

---
# Zadanie 2  - funkcje


Tworzenie funkcji pobierających dane/tabele. Podobnie jak w poprzednim przykładzie należy przygotować kilka funkcji ułatwiających dostęp do danych

Procedury:
- `f_trip_participants`
	- zadaniem funkcji jest zwrócenie listy uczestników wskazanej wycieczki
	- parametry funkcji: `trip_id`
	- funkcja zwraca podobny zestaw danych jak widok  `vw_eservation`
-  `f_person_reservations`
	- zadaniem funkcji jest zwrócenie listy rezerwacji danej osoby 
	- parametry funkcji: `person_id`
	- funkcja zwraca podobny zestaw danych jak widok `vw_reservation`
-  `f_available_trips_to`
	- zadaniem funkcji jest zwrócenie listy wycieczek do wskazanego kraju, dostępnych w zadanym okresie czasu (od `date_from` do `date_to`)
	- parametry funkcji: `country`, `date_from`, `date_to`


Funkcje powinny zwracać tabelę/zbiór wynikowy. Należy rozważyć dodanie kontroli parametrów, (np. jeśli parametrem jest `trip_id` to można sprawdzić czy taka wycieczka istnieje). Podobnie jak w przypadku widoków należy zwrócić uwagę na strukturę kodu

Czy kontrola parametrów w przypadku funkcji ma sens?
- jakie są zalety/wady takiego rozwiązania?

Proponowany zestaw funkcji można rozbudować wedle uznania/potrzeb
- np. można dodać nowe/pomocnicze funkcje/procedury

# Zadanie 2  - rozwiązanie

**f_trip_participants**

```sql

create or replace function f_trip_participants(trip_id int)
    return persons_on_trip_table
as
    result persons_on_trip_table;
begin

    SELECT person_on_trip(res.firstname, res.lastname)
    bulk collect
    into result
    from vw_reservation res
    where res.trip_id = f_trip_participants.trip_id and res.status != 'C';

    return result;
end;
/

```
A rezultatem uruchomienia tej funkcji dla trip_id = 1 jest:

![2_ex_1](zad_2_przyklad_1.png)

**f_person_reservations**

```sql

create or replace function f_person_reservations(person_id int)
    return trips_table
as
    result trips_table;
begin

    SELECT trip_object(res.TRIP_ID, res.TRIP_NAME)
    bulk collect
    into result
    from vw_reservation res
    where res.PERSON_ID = f_person_reservations.person_id and res.status != 'C';

    return result;
end;
/

```

A rezultatem uruchomienia tej funkcji dla person_id = 1 jest:

![2_ex_2](zad_2_przyklad_2.png)

**f_available_trips_to**

```sql

create or replace function f_available_trips_to(country varchar2, date_from date, date_to date)
    return trips_table
as
    result trips_table;
begin

    SELECT trip_object(avt.TRIP_ID, avt.TRIP_NAME)
    bulk collect
    into result
    from vw_available_trip avt
    where (avt.COUNTRY = f_available_trips_to.country) and (avt.TRIP_DATE BETWEEN f_available_trips_to.date_from and f_available_trips_to.date_to);

    return result;
end;
/

```

A rezultatem uruchomienia tej funkcji dla country = 'Polska', date_from = 01-01-2025, date_to = 01-01-2026 jest:

![2_ex_3](zad_2_przyklad_3.png)
---
# Zadanie 3  - procedury


Tworzenie procedur modyfikujących dane. Należy przygotować zestaw procedur pozwalających na modyfikację danych oraz kontrolę poprawności ich wprowadzania

Procedury
- `p_add_reservation`
	- zadaniem procedury jest dopisanie nowej rezerwacji
	- parametry: `trip_id`, `person_id`,  `no_tickets`
	- procedura powinna kontrolować czy wycieczka jeszcze się nie odbyła, i czy sa wolne miejsca
	- procedura powinna również dopisywać inf. do tabeli `log`
- `p_modify_reservation_status
	- zadaniem procedury jest zmiana statusu rezerwacji 
	- parametry: `reservation_id`, `status`
	- procedura powinna kontrolować czy możliwa jest zmiana statusu, np. zmiana statusu już anulowanej wycieczki (przywrócenie do stanu aktywnego nie zawsze jest możliwa – może już nie być miejsc)
	- procedura powinna również dopisywać inf. do tabeli `log`
- `p_modify_reservation
	- zadaniem procedury jest zmiana statusu rezerwacji 
	- parametry: `reservation_id`, `no_iickets`
	- procedura powinna kontrolować czy możliwa jest zmiana liczby sprzedanych/zarezerwowanych biletów – może już nie być miejsc
	- procedura powinna również dopisywać inf. do tabeli `log`
- `p_modify_max_no_places`
	- zadaniem procedury jest zmiana maksymalnej liczby miejsc na daną wycieczkę 
	- parametry: `trip_id`, `max_no_places`
	- nie wszystkie zmiany liczby miejsc są dozwolone, nie można zmniejszyć liczby miejsc na wartość poniżej liczby zarezerwowanych miejsc

Należy rozważyć użycie transakcji

Należy zwrócić uwagę na kontrolę parametrów (np. jeśli parametrem jest trip_id to należy sprawdzić czy taka wycieczka istnieje, jeśli robimy rezerwację to należy sprawdzać czy są wolne miejsca itp..)


Proponowany zestaw procedur można rozbudować wedle uznania/potrzeb
- np. można dodać nowe/pomocnicze funkcje/procedury

# Zadanie 3  - rozwiązanie

**p_add_reservation**

```sql

create PROCEDURE p_add_reservation(
    p_trip_id IN trip.trip_id%TYPE,
    p_person_id IN person.person_id%TYPE,
    p_no_tickets IN reservation.no_tickets%TYPE
) AS
    v_trip_date trip.trip_date%TYPE;
    v_max_places trip.max_no_places%TYPE;
    v_reserved NUMBER;
    v_available NUMBER;
    v_trip_exists NUMBER;
    v_person_exists NUMBER;
    e_past_trip EXCEPTION;
    e_no_available_places EXCEPTION;
    e_trip_not_exists EXCEPTION;
    e_person_not_exists EXCEPTION;
BEGIN
    -- Sprawdzenie czy wycieczka istnieje
    SELECT COUNT(*) INTO v_trip_exists FROM trip WHERE trip_id = p_trip_id;
    IF v_trip_exists = 0 THEN
        RAISE e_trip_not_exists;
    END IF;

    -- Sprawdzenie czy osoba istnieje
    SELECT COUNT(*) INTO v_person_exists FROM person WHERE person_id = p_person_id;
    IF v_person_exists = 0 THEN
        RAISE e_person_not_exists;
    END IF;

    -- Pobranie daty wycieczki i maksymalnej liczby miejsc
    SELECT trip_date, max_no_places INTO v_trip_date, v_max_places
    FROM trip
    WHERE trip_id = p_trip_id;

    -- Sprawdzenie czy wycieczka się już nie odbyła
    IF v_trip_date < SYSDATE THEN
        RAISE e_past_trip;
    END IF;

    -- Obliczenie zajętych miejsc (tylko rezerwacje ze statusem N lub P)
    SELECT NVL(SUM(no_tickets), 0) INTO v_reserved
    FROM reservation
    WHERE trip_id = p_trip_id AND status IN ('N', 'P');

    -- Obliczenie dostępnych miejsc
    v_available := v_max_places - v_reserved;

    -- Sprawdzenie czy są dostępne miejsca
    IF p_no_tickets > v_available THEN
        RAISE e_no_available_places;
    END IF;

    -- Dodanie rezerwacji
    INSERT INTO reservation(trip_id, person_id, status, no_tickets)
    VALUES(p_trip_id, p_person_id, 'N', p_no_tickets);

    -- Dodanie wpisu do logu
    INSERT INTO log(reservation_id, log_date, status, no_tickets)
    VALUES(s_reservation_seq.currval, SYSDATE, 'N', p_no_tickets);

    -- Zatwierdzenie transakcji
    COMMIT;

EXCEPTION
    WHEN e_past_trip THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20001, 'Nie można zarezerwować wycieczki, która już się odbyła.');
    WHEN e_no_available_places THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20002, 'Nie ma wystarczającej liczby wolnych miejsc na wycieczce.');
    WHEN e_trip_not_exists THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20003, 'Wycieczka o podanym ID nie istnieje.');
    WHEN e_person_not_exists THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20004, 'Osoba o podanym ID nie istnieje.');
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20000, 'Wystąpił błąd: ' || SQLERRM);
END p_add_reservation;
/

```

Dodanie rezerwacji na wycieczkę o id = 3 przez osobę o id = 1 na 1 bilet skutkuje:

![3_ex_1](zad_3_przyklad_1.png)


**p_modify_reservation_status**

```sql

create PROCEDURE p_modify_reservation_status(
    p_reservation_id IN reservation.reservation_id%TYPE,
    p_status IN reservation.status%TYPE
) AS
    v_old_status reservation.status%TYPE;
    v_trip_id reservation.trip_id%TYPE;
    v_trip_date trip.trip_date%TYPE;
    v_max_places trip.max_no_places%TYPE;
    v_no_tickets reservation.no_tickets%TYPE;
    v_reserved NUMBER;
    v_available NUMBER;
    v_reservation_exists NUMBER;
    e_reservation_not_exists EXCEPTION;
    e_invalid_status EXCEPTION;
    e_no_available_places EXCEPTION;
    e_past_trip EXCEPTION;
BEGIN
    -- Sprawdzenie czy rezerwacja istnieje
    SELECT COUNT(*) INTO v_reservation_exists FROM reservation WHERE reservation_id = p_reservation_id;
    IF v_reservation_exists = 0 THEN
        RAISE e_reservation_not_exists;
    END IF;
    
    -- Pobranie aktualnego statusu i liczby biletów w rezerwacji
    SELECT status, trip_id, no_tickets 
    INTO v_old_status, v_trip_id, v_no_tickets
    FROM reservation
    WHERE reservation_id = p_reservation_id;
    
    -- Nie można zmienić statusu na ten sam
    IF v_old_status = p_status THEN
        RETURN;
    END IF;
    
    -- Pobranie daty wycieczki i maksymalnej liczby miejsc
    SELECT trip_date, max_no_places INTO v_trip_date, v_max_places
    FROM trip
    WHERE trip_id = v_trip_id;
    
    -- Jeśli zmieniamy status z anulowanego (C) na nowy (N) lub potwierdzony (P)
    -- lub z potwierdzonego (P) na nowy (N)
    IF (v_old_status = 'C' AND p_status IN ('N', 'P')) OR
       (v_old_status = 'P' AND p_status = 'N') THEN
        
        -- Sprawdzenie czy wycieczka się już nie odbyła
        IF v_trip_date < SYSDATE THEN
            RAISE e_past_trip;
        END IF;
        
        -- Obliczenie zajętych miejsc (tylko rezerwacje ze statusem N lub P)
        SELECT NVL(SUM(no_tickets), 0) INTO v_reserved
        FROM reservation
        WHERE trip_id = v_trip_id AND status IN ('N', 'P') AND reservation_id != p_reservation_id;
        
        -- Obliczenie dostępnych miejsc
        v_available := v_max_places - v_reserved;
        
        -- Sprawdzenie czy są dostępne miejsca
        IF v_no_tickets > v_available THEN
            RAISE e_no_available_places;
        END IF;
    END IF;
    
    -- Aktualizacja statusu rezerwacji
    UPDATE reservation
    SET status = p_status
    WHERE reservation_id = p_reservation_id;
    
    -- Dodanie wpisu do logu
    INSERT INTO log(reservation_id, log_date, status, no_tickets)
    VALUES(p_reservation_id, SYSDATE, p_status, v_no_tickets);
    
    -- Zatwierdzenie transakcji
    COMMIT;
    
EXCEPTION
    WHEN e_reservation_not_exists THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20005, 'Rezerwacja o podanym ID nie istnieje.');
    WHEN e_invalid_status THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20006, 'Nieprawidłowy status rezerwacji.');
    WHEN e_no_available_places THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20002, 'Nie ma wystarczającej liczby wolnych miejsc na wycieczce.');
    WHEN e_past_trip THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20001, 'Nie można zmienić statusu dla wycieczki, która już się odbyła.');
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20000, 'Wystąpił błąd: ' || SQLERRM);
END p_modify_reservation_status;
/

```

Procedura wykonana z id rezerwacji = 31 i statusem = 'P':

![3_ex_2](zad_3_przyklad_2.png)


**p_modify_reservation**

```sql

create PROCEDURE p_modify_reservation(
    p_reservation_id IN reservation.reservation_id%TYPE,
    p_no_tickets IN reservation.no_tickets%TYPE
) AS
    v_old_tickets reservation.no_tickets%TYPE;
    v_status reservation.status%TYPE;
    v_trip_id reservation.trip_id%TYPE;
    v_trip_date trip.trip_date%TYPE;
    v_max_places trip.max_no_places%TYPE;
    v_reserved NUMBER;
    v_available NUMBER;
    v_reservation_exists NUMBER;
    e_reservation_not_exists EXCEPTION;
    e_no_available_places EXCEPTION;
    e_past_trip EXCEPTION;
    e_canceled_reservation EXCEPTION;
BEGIN
    -- Sprawdzenie czy rezerwacja istnieje
    SELECT COUNT(*) INTO v_reservation_exists FROM reservation WHERE reservation_id = p_reservation_id;
    IF v_reservation_exists = 0 THEN
        RAISE e_reservation_not_exists;
    END IF;

    -- Pobranie aktualnego statusu, liczby biletów i ID wycieczki
    SELECT status, no_tickets, trip_id
    INTO v_status, v_old_tickets, v_trip_id
    FROM reservation
    WHERE reservation_id = p_reservation_id;

    -- Nie można modyfikować anulowanej rezerwacji
    IF v_status = 'C' THEN
        RAISE e_canceled_reservation;
    END IF;

    -- Jeśli liczba biletów się nie zmienia
    IF v_old_tickets = p_no_tickets THEN
        RETURN;
    END IF;

    -- Pobranie daty wycieczki i maksymalnej liczby miejsc
    SELECT trip_date, max_no_places INTO v_trip_date, v_max_places
    FROM trip
    WHERE trip_id = v_trip_id;

    -- Sprawdzenie czy wycieczka się już nie odbyła
    IF v_trip_date < SYSDATE THEN
        RAISE e_past_trip;
    END IF;

    -- Jeśli zwiększamy liczbę biletów
    IF p_no_tickets > v_old_tickets THEN
        -- Obliczenie zajętych miejsc (tylko rezerwacje ze statusem N lub P)
        SELECT NVL(SUM(no_tickets), 0) INTO v_reserved
        FROM reservation
        WHERE trip_id = v_trip_id AND status IN ('N', 'P') AND reservation_id != p_reservation_id;

        -- Obliczenie dostępnych miejsc
        v_available := v_max_places - v_reserved;

        -- Sprawdzenie czy są dostępne miejsca
        IF (p_no_tickets - v_old_tickets) > v_available THEN
            RAISE e_no_available_places;
        END IF;
    END IF;

    -- Aktualizacja liczby biletów
    UPDATE reservation
    SET no_tickets = p_no_tickets
    WHERE reservation_id = p_reservation_id;

    -- Dodanie wpisu do logu
    INSERT INTO log(reservation_id, log_date, status, no_tickets)
    VALUES(p_reservation_id, SYSDATE, v_status, p_no_tickets);

    -- Zatwierdzenie transakcji
    COMMIT;

EXCEPTION
    WHEN e_reservation_not_exists THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20005, 'Rezerwacja o podanym ID nie istnieje.');
    WHEN e_no_available_places THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20002, 'Nie ma wystarczającej liczby wolnych miejsc na wycieczce.');
    WHEN e_past_trip THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20001, 'Nie można modyfikować rezerwacji dla wycieczki, która już się odbyła.');
    WHEN e_canceled_reservation THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20007, 'Nie można modyfikować anulowanej rezerwacji.');
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20000, 'Wystąpił błąd: ' || SQLERRM);
END p_modify_reservation;
/

```

Procedura wykonana z id rezerwacji = 31 i numerem biletów = 2:

![3_ex_3](zad_3_przyklad_3.png)


**p_modify_max_no_places**

```sql

create PROCEDURE p_modify_max_no_places(
    p_trip_id IN trip.trip_id%TYPE,
    p_max_no_places IN trip.max_no_places%TYPE
) AS
    v_reserved NUMBER;
    v_trip_exists NUMBER;
    e_trip_not_exists EXCEPTION;
    e_invalid_places EXCEPTION;
BEGIN
    -- Sprawdzenie czy wycieczka istnieje
    SELECT COUNT(*) INTO v_trip_exists FROM trip WHERE trip_id = p_trip_id;
    IF v_trip_exists = 0 THEN
        RAISE e_trip_not_exists;
    END IF;

    -- Obliczenie zajętych miejsc (tylko rezerwacje ze statusem N lub P)
    SELECT NVL(SUM(no_tickets), 0) INTO v_reserved
    FROM reservation
    WHERE trip_id = p_trip_id AND status IN ('N', 'P');

    -- Sprawdzenie czy nowa liczba miejsc nie jest mniejsza od liczby zarezerwowanych miejsc
    IF p_max_no_places < v_reserved THEN
        RAISE e_invalid_places;
    END IF;

    -- Aktualizacja maksymalnej liczby miejsc
    UPDATE trip
    SET max_no_places = p_max_no_places
    WHERE trip_id = p_trip_id;

    -- Zatwierdzenie transakcji
    COMMIT;

EXCEPTION
    WHEN e_trip_not_exists THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20003, 'Wycieczka o podanym ID nie istnieje.');
    WHEN e_invalid_places THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20008, 'Nie można zmniejszyć liczby miejsc poniżej liczby już zarezerwowanych miejsc.');
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20000, 'Wystąpił błąd: ' || SQLERRM);
END p_modify_max_no_places;
/

```

Procedura wykonana z id wycieczki = 3 i liczbą miejsc = 15:

![3_ex_4](zad_3_przyklad_4.png)
---
# Zadanie 4  - triggery


Zmiana strategii zapisywania do dziennika rezerwacji. Realizacja przy pomocy triggerów

Należy wprowadzić zmianę, która spowoduje, że zapis do dziennika będzie realizowany przy pomocy trigerów

Triggery:
- trigger/triggery obsługujące 
	- dodanie rezerwacji
	- zmianę statusu
	- zmianę liczby zarezerwowanych/kupionych biletów
- trigger zabraniający usunięcia rezerwacji

Oczywiście po wprowadzeniu tej zmiany należy "uaktualnić" procedury modyfikujące dane. 

>UWAGA
Należy stworzyć nowe wersje tych procedur (dodając do nazwy dopisek 4 - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu  umożliwienia weryfikacji ich poprawności

Należy przygotować procedury: `p_add_reservation_4`, `p_modify_reservation_status_4` , `p_modify_reservation_4`


# Zadanie 4  - rozwiązanie

```sql

-- wyniki, kod, zrzuty ekranów, komentarz ...

```



---
# Zadanie 5  - triggery


Zmiana strategii kontroli dostępności miejsc. Realizacja przy pomocy triggerów

Należy wprowadzić zmianę, która spowoduje, że kontrola dostępności miejsc na wycieczki (przy dodawaniu nowej rezerwacji, zmianie statusu) będzie realizowana przy pomocy trigerów

Triggery:
- Trigger/triggery obsługujące: 
	- dodanie rezerwacji
	- zmianę statusu
	- zmianę liczby zakupionych/zarezerwowanych miejsc/biletów

Oczywiście po wprowadzeniu tej zmiany należy "uaktualnić" procedury modyfikujące dane. 

>UWAGA
Należy stworzyć nowe wersje tych procedur (np. dodając do nazwy dopisek 5 - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu  umożliwienia weryfikacji ich poprawności. 

Należy przygotować procedury: `p_add_reservation_5`, `p_modify_reservation_status_5`, `p_modify_reservation_status_5`


# Zadanie 5  - rozwiązanie

```sql

-- wyniki, kod, zrzuty ekranów, komentarz ...

```

---
# Zadanie 6


Zmiana struktury bazy danych. W tabeli `trip`  należy dodać  redundantne pole `no_available_places`.  Dodanie redundantnego pola uprości kontrolę dostępnych miejsc, ale nieco skomplikuje procedury dodawania rezerwacji, zmiany statusu czy też zmiany maksymalnej liczby miejsc na wycieczki.

Należy przygotować polecenie/procedurę przeliczającą wartość pola `no_available_places` dla wszystkich wycieczek (do jednorazowego wykonania)

Obsługę pola `no_available_places` można zrealizować przy pomocy procedur lub triggerów

Należy zwrócić uwagę na spójność rozwiązania.

>UWAGA
Należy stworzyć nowe wersje tych widoków/procedur/triggerów (np. dodając do nazwy dopisek 6 - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu  umożliwienia weryfikacji ich poprawności. 


- zmiana struktury tabeli

```sql
alter table trip add  
    no_available_places int null
```

- polecenie przeliczające wartość `no_available_places`
	- należy wykonać operację "przeliczenia"  liczby wolnych miejsc i aktualizacji pola  `no_available_places`

# Zadanie 6  - rozwiązanie

```sql

-- wyniki, kod, zrzuty ekranów, komentarz ...

```



---
# Zadanie 6a  - procedury



Obsługę pola `no_available_places` należy zrealizować przy pomocy procedur
- procedura dodająca rezerwację powinna aktualizować pole `no_available_places` w tabeli trip
- podobnie procedury odpowiedzialne za zmianę statusu oraz zmianę maksymalnej liczby miejsc na wycieczkę
- należy przygotować procedury oraz jeśli jest to potrzebne, zaktualizować triggery oraz widoki



>UWAGA
Należy stworzyć nowe wersje tych widoków/procedur/triggerów (np. dodając do nazwy dopisek 6a - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu  umożliwienia weryfikacji ich poprawności. 
- może  być potrzebne wyłączenie 'poprzednich wersji' triggerów 


# Zadanie 6a  - rozwiązanie

```sql

-- wyniki, kod, zrzuty ekranów, komentarz ...

```



---
# Zadanie 6b -  triggery


Obsługę pola `no_available_places` należy zrealizować przy pomocy triggerów
- podczas dodawania rezerwacji trigger powinien aktualizować pole `no_available_places` w tabeli trip
- podobnie, podczas zmiany statusu rezerwacji
- należy przygotować trigger/triggery oraz jeśli jest to potrzebne, zaktualizować procedury modyfikujące dane oraz widoki


>UWAGA
Należy stworzyć nowe wersje tych widoków/procedur/triggerów (np. dodając do nazwy dopisek 6b - od numeru zadania). Poprzednie wersje procedur należy pozostawić w celu  umożliwienia weryfikacji ich poprawności. 
- może  być potrzebne wyłączenie 'poprzednich wersji' triggerów 



# Zadanie 6b  - rozwiązanie


```sql

-- wyniki, kod, zrzuty ekranów, komentarz ...

```


# Zadanie 7 - podsumowanie

Porównaj sposób programowania w systemie Oracle PL/SQL ze znanym ci systemem/językiem MS Sqlserver T-SQL

```sql

-- komentarz ...

```