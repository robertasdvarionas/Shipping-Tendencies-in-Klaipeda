# EXPLORATORY DATA ANALYSIS OF THE SHIPPING TENDENCIES IN PORT OF KLAIPĖDA, LITHUANIA

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Klaipedos%20Uosto%20Tendencijos_page-0001.jpg)

## GOAL

The goal of this project is to apply data analysis tools - SQL and Power BI - to the dataset and then to understand and visualise what shipping tendencies or insights can be drawn in the port of Klaipėda.

## DATA OVERVIEW

The dataset was taken from Lietuvos Atvirų Duomenų Portalas and is publicly available on https://data.gov.lt/.
Data was originally provided by AB Klaipėdos valstybinio jūrų uosto direkcija.

The dataset contains information on international ships arriving at or departing from the port of Klaipėda registered by AB Klaipėdos valstybinio jūrų uosto direkcija – ship registration number, arrival or departure time, ship technical data and agent company.
Warships are not included in the dataset.
Data has been collected since 2013.

The dataset contains 47,802 rows.

## DATA CLEANING AND PREPARATION

Firstly I check if there are any duplicate laivo_id entries.

```sql
SELECT laivo_id, COUNT(*)
FROM Laivas_edited
GROUP BY laivo_id
HAVING COUNT(*) > 1;
```

Since there are no duplicates I move forward.

One of the main columns in this dataset is the ship type and I see that there are way too many distinct ship types. It seems like a lot of them are overlapping in function or type, therefore, we can standardize them into concise categories for a nicer and cleaner visualization later on.

```sql
UPDATE Laivas_edited
SET laivo_tipas = 'Tanklaivis'
WHERE laivo_tipas LIKE '%tanklaivis%';

UPDATE Laivas_edited
SET laivo_tipas = N'Bendrųjų krovinių'
WHERE LOWER([laivo_tipas]) LIKE '%bendr%';

UPDATE Laivas_edited
SET laivo_tipas = 'Ro-ro'
WHERE LOWER([laivo_tipas]) LIKE '%ro-ro%';

UPDATE Laivas_edited
SET laivo_tipas = N'Barža'
WHERE LOWER([laivo_tipas]) LIKE N'%barž%';

UPDATE Laivas_edited
SET laivo_tipas = 'Biriems kroviniams'
WHERE LOWER([laivo_tipas]) LIKE '%biriems%';

UPDATE Laivas_edited
SET laivo_tipas = N'Uosto operacijų'
WHERE 
LOWER([laivo_tipas]) LIKE '%avarinis%' OR
LOWER([laivo_tipas]) LIKE '%stumikas%' OR
LOWER([laivo_tipas]) LIKE '%vilkikas%' OR
LOWER([laivo_tipas]) LIKE N'%priemonėms' OR
LOWER([laivo_tipas]) LIKE '%pagalbinis%' OR
LOWER([laivo_tipas]) LIKE '%kranas%' OR
LOWER([laivo_tipas]) LIKE 'pontonas' OR
LOWER([laivo_tipas]) LIKE 'mokymo' OR
LOWER([laivo_tipas]) LIKE 'nesavaeigis%' OR
LOWER([laivo_tipas]) LIKE 'trauleris' OR
LOWER([laivo_tipas]) LIKE '%hidrografinis%' OR
LOWER([laivo_tipas]) LIKE N'žem%';

UPDATE Laivas_edited
SET laivo_tipas = 'Kiti'
WHERE
LOWER([laivo_tipas]) LIKE '%kitas%' OR
LOWER([laivo_tipas]) LIKE '%transporto%' OR
LOWER([laivo_tipas]) LIKE 'sportinis' OR
LOWER([laivo_tipas]) LIKE 'vidaus%';
```

Then I check the ship registration country column.
Power BI can take in Lithuanian country names and perfectly plot them on the map. However, I see that there are quite a few entries where the country or territory name will not be translated correcly by Power BI, therefore, some standartization is needed. To be safe, I changed the incorrect translations to English to be 100% sure that Power BI plots them correctly.

```sql
UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'Virgin Islands'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE '%mergel%';

UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'United Kingdom'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE '%britanija%';

UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'Faroe Islands'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE 'farer%';

UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'Cayman Islands'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE 'kaiman%';

UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'Marshall Islands'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE N'maršalo%';

UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'Aruba'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE 'Olandijos%';

UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'Saudi Arabia'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE '%arabija%';

UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'Saint Kitts'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE '%kristoferis%';

UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = 'Saint Vincent and the Grenadines'
WHERE
LOWER([registracijos_valstybes_pavadinimas]) LIKE '%vinsentas%';
```

## DATA VISUALIZATION AND INSIGHTS

## CONCLUSION
