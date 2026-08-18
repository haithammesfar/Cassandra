# Apache Cassandra – Klausur-Zusammenfassung

> Ziel: Cassandra/CQL-Code in der Klausur **schnell lesen und verstehen**.

---

# 1. Cassandra ↔ SQL – wichtigste Befehle

| Cassandra / CQL | SQL | Bedeutung |
|---|---|---|
| `CREATE KEYSPACE` | `CREATE DATABASE` | Datenbank/Keyspace erstellen |
| `USE` | `USE` | Keyspace auswählen |
| `CREATE TABLE` | `CREATE TABLE` | Tabelle erstellen |
| `INSERT INTO` | `INSERT INTO` | Daten einfügen |
| `SELECT` | `SELECT` | Daten lesen |
| `WHERE` | `WHERE` | Daten filtern |
| `UPDATE` | `UPDATE` | Daten ändern |
| `DELETE` | `DELETE` | Daten löschen |
| `ORDER BY` | `ORDER BY` | nach Clustering Column sortieren |
| `LIMIT` | `LIMIT` | Ergebnis begrenzen |
| `IN` | `IN` | mehrere Werte |
| `ALLOW FILTERING` | – | Cassandra erlaubt sonst problematische Filterung |
| `PRIMARY KEY` | `PRIMARY KEY` | Schlüssel definieren |

---

# 2. Wichtigstes Thema: PRIMARY KEY ⭐

In Cassandra besteht der Primary Key aus:

```text
PRIMARY KEY
    │
    ├── Partition Key
    │
    └── Clustering Column(s)
```

Allgemein:

```cql
PRIMARY KEY ((partition_key), clustering_column)
```

---

# 3. Einfacher Primary Key

```cql
CREATE TABLE students (
    id int,
    name text,
    city text,
    PRIMARY KEY (id)
);
```

Hier ist:

```text
Partition Key: id
Clustering Columns: keine
```

`id` bestimmt, **auf welcher Partition / welchem Node** die Daten gespeichert werden.

Beispiel:

```cql
SELECT *
FROM students
WHERE id = 1;
```

---

# 4. Partition Key

Der **Partition Key** bestimmt:

```text
Wo werden die Daten gespeichert?
```

Beispiel:

```cql
PRIMARY KEY (student_id)
```

Dann ist:

```text
student_id = Partition Key
```

Cassandra berechnet ungefähr:

```text
hash(student_id)
      ↓
richtige Partition / Node
```

Deshalb ist eine Abfrage über den Partition Key sehr schnell:

```cql
SELECT *
FROM students
WHERE student_id = 10;
```

---

# 5. Clustering Column ⭐

Beispiel:

```cql
CREATE TABLE grades (
    student_id int,
    semester int,
    subject text,
    grade int,

    PRIMARY KEY (student_id, semester)
);
```

Hier gilt:

```text
student_id → Partition Key

semester   → Clustering Column
```

Bedeutung:

```text
student_id
→ bestimmt die Partition

semester
→ sortiert Daten INNERHALB der Partition
```

Beispieldaten:

```text
student_id | semester | subject
-----------|----------|---------
1          | 1        | SQL
1          | 2        | MongoDB
1          | 3        | Cassandra
```

Alle Daten mit:

```text
student_id = 1
```

liegen zusammen.

Innerhalb dieser Partition werden sie nach:

```text
semester
```

geordnet.

---

# 6. PRIMARY KEY `(a, b)` richtig lesen ⭐

```cql
PRIMARY KEY (student_id, semester)
```

Bedeutet:

```text
student_id = Partition Key

semester = Clustering Column
```

NICHT:

```text
student_id + semester zusammen Partition Key
```

---

# 7. Composite Partition Key ⭐⭐

Jetzt wichtig:

```cql
PRIMARY KEY ((student_id, course_id), semester)
```

Die **doppelten Klammern** sind entscheidend.

```text
(student_id, course_id)
→ zusammen Partition Key

semester
→ Clustering Column
```

Also:

```text
Partition Key:
student_id + course_id

Clustering:
semester
```

---

# 8. Unterschied `(a,b)` vs `((a,b))` ⭐

## Variante 1

```cql
PRIMARY KEY (a, b)
```

bedeutet:

```text
Partition Key:
a

Clustering Column:
b
```

---

## Variante 2

```cql
PRIMARY KEY ((a, b))
```

bedeutet:

```text
Composite Partition Key:
a + b

Keine Clustering Column
```

---

## Variante 3

```cql
PRIMARY KEY ((a, b), c)
```

bedeutet:

```text
Partition Key:
a + b

Clustering Column:
c
```

---

# 9. Mehrere Clustering Columns

```cql
PRIMARY KEY (
    student_id,
    year,
    semester,
    subject
)
```

Bedeutung:

```text
Partition Key:
student_id

Clustering Columns:
1. year
2. semester
3. subject
```

Die Reihenfolge ist wichtig:

```text
student_id
    ↓
year
    ↓
semester
    ↓
subject
```

Die Daten werden innerhalb der Partition lexikographisch nach diesen Spalten organisiert.

---

# 10. Composite Primary Key

Ein Primary Key mit Partition Key + Clustering Columns heißt häufig:

```text
Composite Primary Key
```

Beispiel:

```cql
PRIMARY KEY (student_id, semester)
```

besteht aus:

```text
Partition Key:
student_id

+

Clustering Column:
semester
```

---

# 11. Komplettes Schlüssel-Beispiel ⭐

```cql
CREATE TABLE exams (
    course_id int,
    year int,
    semester int,
    student_id int,
    grade double,

    PRIMARY KEY (
        (course_id, year),
        semester,
        student_id
    )
);
```

Lesen:

```text
Partition Key:
course_id + year

Clustering Columns:
1. semester
2. student_id
```

Also:

```text
(course_id + year)
       ↓
bestimmt Partition

semester
       ↓
erste Sortierung

student_id
       ↓
zweite Sortierung
```

---

# 12. PRIMARY KEY muss eindeutig sein

Beispiel:

```cql
PRIMARY KEY (student_id, semester)
```

Dann muss die Kombination eindeutig sein:

```text
student_id + semester
```

Erlaubt:

```text
student_id | semester
1          | 1
1          | 2
1          | 3
```

Nicht zwei verschiedene Zeilen mit exakt:

```text
student_id = 1
semester   = 1
```

Ein erneutes `INSERT` mit demselben Primary Key überschreibt/aktualisiert die vorhandenen Werte.

---

# 13. CREATE KEYSPACE

```cql
CREATE KEYSPACE university
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};
```

Bedeutung:

```text
university
→ Name des Keyspace

replication_factor = 3
→ Daten werden repliziert
```

Für Produktions-Cluster sieht man oft Strategien wie `NetworkTopologyStrategy`.

---

# 14. USE

```cql
USE university;
```

Bedeutung:

```text
Ab jetzt im Keyspace university arbeiten.
```

---

# 15. CREATE TABLE

```cql
CREATE TABLE students (
    id int,
    name text,
    age int,
    city text,

    PRIMARY KEY (id)
);
```

---

# 16. Wichtige Datentypen

| Cassandra | Bedeutung |
|---|---|
| `int` | Ganzzahl |
| `bigint` | große Ganzzahl |
| `float` | Kommazahl |
| `double` | Kommazahl |
| `decimal` | genaue Dezimalzahl |
| `text` | Text |
| `varchar` | Text |
| `boolean` | `true/false` |
| `timestamp` | Datum + Zeit |
| `date` | Datum |
| `time` | Uhrzeit |
| `uuid` | UUID |
| `timeuuid` | zeitbasierte UUID |
| `blob` | Binärdaten |

---

# 17. INSERT

```cql
INSERT INTO students (
    id,
    name,
    age,
    city
)
VALUES (
    1,
    'Ali',
    22,
    'Berlin'
);
```

Ergebnis:

```text
id | name | age | city
1  | Ali  | 22  | Berlin
```

---

# 18. SELECT

Alle Spalten:

```cql
SELECT *
FROM students;
```

Bestimmte Spalten:

```cql
SELECT name, age
FROM students;
```

---

# 19. WHERE ⭐

```cql
SELECT *
FROM students
WHERE id = 1;
```

Wichtig:

Cassandra ist **nicht wie normale SQL-Datenbanken**.

Man kann nicht beliebig nach jeder Spalte filtern.

Sehr gut:

```cql
WHERE partition_key = ...
```

Problematisch:

```cql
WHERE irgendeine_normale_spalte = ...
```

---

# 20. Partition Key bei WHERE ⭐

Tabelle:

```cql
PRIMARY KEY (student_id, semester)
```

Sehr gute Abfrage:

```cql
SELECT *
FROM grades
WHERE student_id = 1;
```

Denn:

```text
student_id = Partition Key
```

---

# 21. Clustering Column filtern

Bei:

```cql
PRIMARY KEY (student_id, semester)
```

geht:

```cql
SELECT *
FROM grades
WHERE student_id = 1
AND semester = 2;
```

Weil zuerst der Partition Key angegeben wurde.

---

# 22. Falsche typische Abfrage

```cql
SELECT *
FROM grades
WHERE semester = 2;
```

Problem:

```text
semester ist nur Clustering Column.

Partition Key student_id fehlt.
```

Cassandra weiß dadurch nicht direkt, in welcher Partition gesucht werden soll.

---

# 23. Reihenfolge von Clustering Columns ⭐

Angenommen:

```cql
PRIMARY KEY (
    student_id,
    year,
    semester
)
```

Dann:

```text
Partition Key:
student_id

Clustering:
1. year
2. semester
```

Gut:

```cql
WHERE student_id = 1
AND year = 2026
AND semester = 2;
```

Auch:

```cql
WHERE student_id = 1
AND year = 2026;
```

Aber problematisch:

```cql
WHERE student_id = 1
AND semester = 2;
```

Warum?

```text
year wurde übersprungen.
```

**Merksatz:**

```text
Clustering Columns von LINKS NACH RECHTS verwenden.
```

---

# 24. Range Queries

Bei:

```cql
PRIMARY KEY (student_id, semester)
```

kann man z.B.:

```cql
SELECT *
FROM grades
WHERE student_id = 1
AND semester >= 2;
```

Bedeutung:

```text
Student 1
und Semester >= 2
```

Range-Operatoren:

```text
>
>=
<
<=
```

sind besonders sinnvoll auf Clustering Columns innerhalb einer bekannten Partition.

---

# 25. IN

```cql
SELECT *
FROM students
WHERE id IN (1, 2, 3);
```

Bedeutung:

```text
id = 1
ODER id = 2
ODER id = 3
```

---

# 26. AND

```cql
SELECT *
FROM grades
WHERE student_id = 1
AND semester = 2;
```

Cassandra verwendet bei Abfragen typischerweise:

```text
AND
```

---

# 27. ORDER BY ⭐

Beispiel:

```cql
PRIMARY KEY (student_id, semester)
```

Dann:

```cql
SELECT *
FROM grades
WHERE student_id = 1
ORDER BY semester DESC;
```

Bedeutung:

```text
Nur Student 1

und

semester absteigend sortieren
```

Wichtig:

```text
ORDER BY funktioniert auf Clustering Columns
innerhalb einer Partition.
```

Nicht beliebig auf jeder normalen Spalte.

---

# 28. ASC / DESC

```cql
ORDER BY semester ASC;
```

```text
1
2
3
4
```

```cql
ORDER BY semester DESC;
```

```text
4
3
2
1
```

---

# 29. CLUSTERING ORDER BY

Sortierreihenfolge kann beim Erstellen definiert werden:

```cql
CREATE TABLE grades (
    student_id int,
    semester int,
    grade int,

    PRIMARY KEY (student_id, semester)
)
WITH CLUSTERING ORDER BY (
    semester DESC
);
```

Bedeutung:

```text
Innerhalb jeder student_id-Partition
werden Semester standardmäßig absteigend gespeichert/gelesen.
```

---

# 30. LIMIT

```cql
SELECT *
FROM students
LIMIT 5;
```

Bedeutung:

```text
Maximal 5 Ergebnisse.
```

---

# 31. UPDATE

```cql
UPDATE students
SET age = 23
WHERE id = 1;
```

Vorher:

```text
Ali → 22
```

Nachher:

```text
Ali → 23
```

Wichtig:

```text
WHERE enthält normalerweise den Primary Key.
```

---

# 32. UPDATE mehrere Spalten

```cql
UPDATE students
SET age = 23,
    city = 'Hamburg'
WHERE id = 1;
```

---

# 33. DELETE komplette Zeile

```cql
DELETE FROM students
WHERE id = 1;
```

Bedeutung:

```text
Zeile mit id = 1 löschen.
```

---

# 34. DELETE einzelne Spalte

```cql
DELETE city
FROM students
WHERE id = 1;
```

Bedeutung:

```text
Nur city entfernen.

Die komplette Zeile bleibt bestehen.
```

---

# 35. COUNT

```cql
SELECT COUNT(*)
FROM students;
```

Bedeutung:

```text
Anzahl der Zeilen.
```

---

# 36. DISTINCT

```cql
SELECT DISTINCT student_id
FROM grades;
```

Wichtig:

`DISTINCT` ist in Cassandra stärker eingeschränkt als in klassischen SQL-Datenbanken und wird typischerweise mit Partition-Key-Spalten verwendet.

---

# 37. ALLOW FILTERING ⭐

Beispiel:

```cql
SELECT *
FROM students
WHERE age = 22
ALLOW FILTERING;
```

Wenn `age` kein geeigneter Key/Index ist, müsste Cassandra möglicherweise viele Daten durchsuchen.

`ALLOW FILTERING` bedeutet ungefähr:

```text
"Ich weiß, dass diese Abfrage ineffizient sein kann.
Führe sie trotzdem aus."
```

Für Klausur merken:

```text
ALLOW FILTERING
→ erlaubt Filterung außerhalb des normalen Key-Zugriffspfades
→ kann teuer/langsam sein
```

---

# 38. IF NOT EXISTS

Beim Erstellen:

```cql
CREATE TABLE IF NOT EXISTS students (
    id int PRIMARY KEY,
    name text
);
```

Bedeutung:

```text
Erstelle Tabelle nur,
wenn sie noch nicht existiert.
```

---

Bei INSERT:

```cql
INSERT INTO students (
    id,
    name
)
VALUES (
    1,
    'Ali'
)
IF NOT EXISTS;
```

Bedeutung:

```text
Nur einfügen,
wenn der Primary Key noch nicht existiert.
```

---

# 39. IF bei UPDATE

```cql
UPDATE students
SET age = 23
WHERE id = 1
IF age = 22;
```

Bedeutung:

```text
Ändere age auf 23

ABER nur wenn aktuell:
age = 22
```

Das ist eine **Lightweight Transaction (LWT)**.

---

# 40. USING TTL ⭐

TTL = **Time To Live**

```cql
INSERT INTO sessions (
    id,
    user
)
VALUES (
    1,
    'Ali'
)
USING TTL 3600;
```

Bedeutung:

```text
Datensatz läuft nach 3600 Sekunden ab.

3600 Sekunden = 1 Stunde
```

Danach wird der Wert automatisch ungültig/entfernt.

---

# 41. TTL beim UPDATE

```cql
UPDATE students
USING TTL 600
SET city = 'Berlin'
WHERE id = 1;
```

Bedeutung:

```text
city = Berlin

für 600 Sekunden.
```

---

# 42. USING TIMESTAMP

```cql
UPDATE students
USING TIMESTAMP 1000000
SET age = 23
WHERE id = 1;
```

Cassandra verwendet Timestamps, um bei konkurrierenden Writes zu entscheiden, welcher Wert neuer ist.

Merken:

```text
höherer/neuerer Timestamp
→ gewinnt typischerweise
```

---

# 43. Collections

Cassandra besitzt Collection-Typen:

```text
list
set
map
```

---

# 44. LIST

```cql
CREATE TABLE students (
    id int PRIMARY KEY,
    name text,
    skills list<text>
);
```

Beispielwert:

```text
skills = ['SQL', 'MongoDB', 'Cassandra']
```

---

# 45. SET

```cql
skills set<text>
```

Beispiel:

```text
{'SQL', 'MongoDB', 'Cassandra'}
```

Eigenschaft:

```text
keine Duplikate
```

---

# 46. MAP

```cql
grades map<text, int>
```

Beispiel:

```text
{
    'SQL': 1,
    'MongoDB': 2,
    'Cassandra': 1
}
```

Also:

```text
Key → Value
```

---

# 47. Collection aktualisieren

## List

```cql
UPDATE students
SET skills = skills + ['Cassandra']
WHERE id = 1;
```

---

## Set

```cql
UPDATE students
SET skills = skills + {'Cassandra'}
WHERE id = 1;
```

---

## Map

```cql
UPDATE students
SET grades['Cassandra'] = 1
WHERE id = 1;
```

---

# 48. CREATE INDEX

Beispiel:

```cql
CREATE INDEX ON students (city);
```

Danach können bestimmte Abfragen nach `city` unterstützt werden:

```cql
SELECT *
FROM students
WHERE city = 'Berlin';
```

Aber:

```text
Primary-Key-basiertes Datenmodell
ist in Cassandra wichtiger als klassische SQL-Indizes.
```

---

# 49. DROP

Tabelle löschen:

```cql
DROP TABLE students;
```

Keyspace löschen:

```cql
DROP KEYSPACE university;
```

---

# 50. TRUNCATE

```cql
TRUNCATE students;
```

Bedeutung:

```text
Alle Daten löschen,
aber Tabelle behalten.
```

Vergleich:

```text
DROP TABLE
→ Tabelle + Daten weg

TRUNCATE
→ nur Daten weg
```

---

# 51. Cassandra denkt anders als SQL ⭐

SQL denkt oft:

```text
1 Tabelle erstellen
2 Daten speichern
3 später beliebige Abfragen machen
```

Cassandra denkt eher:

```text
1 Welche Query brauche ich?
2 Danach Tabelle und Primary Key gestalten
3 Daten passend zu dieser Query speichern
```

Das nennt man:

```text
Query-driven Data Modeling
```

---

# 52. Partition Key – wichtigste Bedeutung ⭐⭐⭐

Der Partition Key entscheidet:

```text
WO liegen die Daten?
```

Beispiel:

```cql
PRIMARY KEY (student_id, semester)
```

```text
student_id
    ↓
Hash
    ↓
Partition / Node
```

Alle Zeilen mit:

```text
student_id = 1
```

liegen logisch in derselben Partition.

---

# 53. Clustering Column – wichtigste Bedeutung ⭐⭐⭐

Die Clustering Column entscheidet:

```text
WIE sind Daten innerhalb einer Partition organisiert/sortiert?
```

Beispiel:

```cql
PRIMARY KEY (student_id, semester)
```

```text
Partition:

student_id = 1

    semester 1
    semester 2
    semester 3
    semester 4
```

---

# 54. Composite Partition Key ⭐⭐⭐

```cql
PRIMARY KEY (
    (course_id, year),
    semester
)
```

Bedeutung:

```text
course_id + year
→ gemeinsam Partition Key

semester
→ Clustering Column
```

Beispiel:

```text
course_id = 10
year      = 2026

        ↓

eine bestimmte Partition
```

Innerhalb:

```text
semester 1
semester 2
semester 3
```

---

# 55. Warum Composite Partition Key?

Ohne Composite Key:

```cql
PRIMARY KEY (country, user_id)
```

```text
Partition Key:
country
```

Wenn Millionen Benutzer:

```text
country = Germany
```

könnte eine riesige Partition entstehen.

Mit:

```cql
PRIMARY KEY ((country, region), user_id)
```

werden Daten stärker verteilt:

```text
Germany + Berlin
Germany + Hamburg
Germany + München
```

---

# 56. PRIMARY KEY schnell erkennen ⭐⭐⭐

## Beispiel A

```cql
PRIMARY KEY (id)
```

```text
Partition Key:
id

Clustering:
keine
```

---

## Beispiel B

```cql
PRIMARY KEY (user_id, timestamp)
```

```text
Partition Key:
user_id

Clustering:
timestamp
```

---

## Beispiel C

```cql
PRIMARY KEY (user_id, year, timestamp)
```

```text
Partition Key:
user_id

Clustering:
year
timestamp
```

---

## Beispiel D

```cql
PRIMARY KEY ((user_id, year), timestamp)
```

```text
Partition Key:
user_id + year

Clustering:
timestamp
```

---

## Beispiel E

```cql
PRIMARY KEY (
    (country, city),
    year,
    month,
    timestamp
)
```

```text
Partition Key:
country + city

Clustering:
1. year
2. month
3. timestamp
```

---

# 57. Klausur-Falle: Klammern ⭐⭐⭐

Sehr wichtig:

```cql
PRIMARY KEY (a, b, c)
```

ist:

```text
Partition Key:
a

Clustering:
b, c
```

Aber:

```cql
PRIMARY KEY ((a, b), c)
```

ist:

```text
Partition Key:
a + b

Clustering:
c
```

Die **inneren Klammern**:

```text
(a, b)
```

zeigen einen **Composite Partition Key**.

---

# 58. Klausur-Falle: WHERE ⭐

Bei:

```cql
PRIMARY KEY (
    (course_id, year),
    semester,
    student_id
)
```

Gute Query:

```cql
SELECT *
FROM exams
WHERE course_id = 10
AND year = 2026;
```

Warum?

```text
Kompletter Partition Key vorhanden.
```

---

Auch:

```cql
SELECT *
FROM exams
WHERE course_id = 10
AND year = 2026
AND semester = 2;
```

Sehr gut:

```text
Partition vollständig
+
erste Clustering Column
```

---

Problematisch:

```cql
WHERE course_id = 10;
```

Warum?

```text
Composite Partition Key ist:

course_id + year

year fehlt.
```

---

Problematisch:

```cql
WHERE course_id = 10
AND year = 2026
AND student_id = 5;
```

Warum?

```text
Clustering-Reihenfolge:

semester
→ student_id

semester wurde übersprungen.
```

---

# 59. Typisches Klausur-Beispiel ⭐⭐⭐

```cql
CREATE TABLE measurements (
    sensor_id int,
    day date,
    timestamp timestamp,
    temperature double,

    PRIMARY KEY (
        (sensor_id, day),
        timestamp
    )
)
WITH CLUSTERING ORDER BY (
    timestamp DESC
);
```

Schnell lesen:

```text
Partition Key:
sensor_id + day

Clustering Column:
timestamp

Sortierung:
timestamp DESC

Messwerte eines Sensors
an einem bestimmten Tag
liegen zusammen.

Neueste Messung zuerst.
```

Query:

```cql
SELECT *
FROM measurements
WHERE sensor_id = 5
AND day = '2026-08-18'
LIMIT 10;
```

Bedeutung:

```text
Hole für Sensor 5
am 18.08.2026
die ersten 10 Messungen.

Da timestamp DESC:
→ die neuesten zuerst.
```

---

# 60. Cassandra Klausur-Spickzettel

```text
====================================
PRIMARY KEY
====================================

PRIMARY KEY (a)

Partition Key:
a


PRIMARY KEY (a, b)

Partition Key:
a

Clustering:
b


PRIMARY KEY (a, b, c)

Partition Key:
a

Clustering:
b
c


PRIMARY KEY ((a, b))

Composite Partition Key:
a + b


PRIMARY KEY ((a, b), c)

Partition Key:
a + b

Clustering:
c


PRIMARY KEY ((a, b), c, d)

Partition Key:
a + b

Clustering:
c
d


====================================
BEDEUTUNG
====================================

Partition Key
→ WO liegen Daten?
→ bestimmt Partition / Node


Clustering Column
→ WIE liegen Daten innerhalb
  einer Partition?
→ Sortierung


Composite Partition Key
→ mehrere Spalten bilden
  gemeinsam den Partition Key


====================================
KLAMMER-REGEL ⭐
====================================

(a, b)

wenn direkt nach PRIMARY KEY:

PRIMARY KEY (a, b)

→ a Partition Key
→ b Clustering


((a, b))

→ a + b zusammen Partition Key


====================================
QUERY-REGEL ⭐
====================================

Am besten immer:

WHERE kompletter_partition_key = ...


Clustering Columns:

von LINKS NACH RECHTS verwenden.


Beispiel:

PRIMARY KEY (id, year, month)

id
→ Partition

year
→ Clustering 1

month
→ Clustering 2


GUT:

WHERE id = 1


GUT:

WHERE id = 1
AND year = 2026


GUT:

WHERE id = 1
AND year = 2026
AND month = 8


PROBLEMATISCH:

WHERE year = 2026


PROBLEMATISCH:

WHERE id = 1
AND month = 8

weil year übersprungen wurde.


====================================
CQL BEFEHLE
====================================

CREATE KEYSPACE
→ Datenbank erstellen

USE
→ Keyspace auswählen

CREATE TABLE
→ Tabelle erstellen

INSERT
→ Daten einfügen

SELECT
→ Daten lesen

WHERE
→ filtern

UPDATE
→ Daten ändern

DELETE
→ Daten löschen

ORDER BY
→ Clustering Column sortieren

LIMIT
→ Anzahl begrenzen

IN
→ mehrere Werte

ALLOW FILTERING
→ teurere Filterung erlauben

TRUNCATE
→ alle Daten löschen,
  Tabelle behalten

DROP
→ Tabelle/Keyspace löschen


====================================
SONSTIGES
====================================

TTL
→ Ablaufzeit der Daten

USING TTL 3600
→ Daten 3600 Sekunden gültig


IF NOT EXISTS
→ nur erstellen/einfügen,
  wenn noch nicht vorhanden


IF ...
→ bedingtes Update
→ Lightweight Transaction


CLUSTERING ORDER BY
→ Standard-Sortierung innerhalb
  einer Partition


====================================
COLLECTIONS
====================================

list<text>
→ geordnete Liste
→ Duplikate möglich


set<text>
→ Menge
→ keine Duplikate


map<text,int>
→ Key-Value


====================================
EXTREM WICHTIG ⭐⭐⭐
====================================

Partition Key
→ verteilt Daten im Cluster


Clustering Column
→ sortiert Daten INNERHALB
  einer Partition


PRIMARY KEY (a,b)

a = Partition Key
b = Clustering


PRIMARY KEY ((a,b),c)

a+b = Composite Partition Key
c   = Clustering


Kompletter Partition Key
ist für effiziente Queries
besonders wichtig.


Clustering Columns
immer in ihrer Reihenfolge lesen.


Cassandra:
QUERY FIRST

→ Tabelle wird passend
  zur benötigten Query gebaut.
```

---

# 61. Eine Regel für die Klausur

Wenn du siehst:

```cql
PRIMARY KEY (
    (A, B),
    C,
    D
)
```

denke sofort:

```text
(A + B)
    ↓
PARTITION KEY
    ↓
Welche Partition?


C
↓
Clustering Column 1


D
↓
Clustering Column 2
```

Und bei einer Query:

```cql
WHERE A = ...
AND B = ...
AND C = ...
```

liest du:

```text
A + B
→ Partition gefunden

C
→ innerhalb dieser Partition filtern
```

> **Merksatz:**  
> **Partition Key = WO liegen die Daten?**  
> **Clustering Columns = WIE sind die Daten dort sortiert?**  
> **Doppelte Klammern `((...))` = Composite Partition Key.**
