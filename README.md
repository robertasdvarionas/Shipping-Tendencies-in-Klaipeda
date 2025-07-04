# EXPLORATORY DATA ANALYSIS OF THE SHIPPING TENDENCIES IN PORT OF KLAIPĖDA, LITHUANIA

![alt text](https://github.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/blob/main/Related%20Images/Klaipedos%20Uosto%20Tendencijos_page-0001.jpg)

## GOAL

The goal of this project is to apply data analysis tools - **SQL and Power BI** - to the dataset and then to understand and visualize what shipping tendencies or insights can be drawn in the port of Klaipėda.

## DATA OVERVIEW

The dataset was taken from Lietuvos Atvirų Duomenų Portalas and is publicly available on https://data.gov.lt/.
Data was originally provided by AB Klaipėdos valstybinio jūrų uosto direkcija.

The dataset contains information on international ships arriving at or departing from the port of Klaipėda registered by AB Klaipėdos valstybinio jūrų uosto direkcija – ship registration number, arrival or departure time, ship technical data and agent company.
Warships are not included in the dataset.
Data has been collected since 2013.

The dataset contains 47,802 rows.

## DATA CLEANING AND PREPARATION

Firstly, I created a new database titled *'Laivai'* and imported the raw csv file into Microsoft SQL Server Management Studio as a flat file.

The raw dataset looks like this.

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Related%20Images/raw_dataset.png)


As a safety measure, I duplicated the table *'Laivas'* and called it *'Laivas_edited'* to keep the original raw dataset in case I need to access it later on.

```sql
SELECT * INTO Laivas_edited
FROM Laivas;
```

Then, I started to look over the data to see what needs clean-up, standartization and etc.

After a general overview, I immediately saw that the first four columns will not be relevant in our project, hence I decided to remove them from our dataset.

```sql
ALTER TABLE Laivas_edited
DROP COLUMN type,id,revision,page_next;
```

I checked if there are any duplicate ship ID - **laivo_id** - column entries.

```sql
SELECT laivo_id, COUNT(*)
FROM Laivas_edited
GROUP BY laivo_id
HAVING COUNT(*) > 1;
```

Since there are no duplicates, I moved forward.

One of the main columns in this dataset is the ship type - **laivo_tipas** - column and I saw that there were too many distinct ship types. A lot of them were overlapping in function or type, therefore, I could standardize them into concise categories for a nicer and cleaner visualization later on.

```sql
UPDATE Laivas_edited
SET laivo_tipas = CASE
    WHEN lower(laivo_tipas) LIKE '%tanklaivis%' THEN 'Tanklaivis'
    WHEN LOWER(laivo_tipas) LIKE '%bendr%' THEN N'Bendrųjų krovinių'
    WHEN LOWER(laivo_tipas) LIKE '%ro-ro%' THEN 'Ro-ro'
    WHEN LOWER(laivo_tipas) LIKE N'%barž%' THEN N'Barža'
    WHEN LOWER(laivo_tipas) LIKE '%biriems%' THEN 'Biriems kroviniams'
    
    WHEN LOWER(laivo_tipas) LIKE '%avarinis%'
       OR LOWER(laivo_tipas) LIKE '%stumikas%'
       OR LOWER(laivo_tipas) LIKE '%vilkikas%'
       OR LOWER(laivo_tipas) LIKE N'%priemonėms'
       OR LOWER(laivo_tipas) LIKE '%pagalbinis%'
       OR LOWER(laivo_tipas) LIKE '%kranas%'
       OR LOWER(laivo_tipas) LIKE '%pontonas%'
       OR LOWER(laivo_tipas) LIKE '%mokymo%'
       OR LOWER(laivo_tipas) LIKE '%nesavaeigis%'
       OR LOWER(laivo_tipas) LIKE '%trauleris%'
       OR LOWER(laivo_tipas) LIKE '%hidrografinis%'
       OR LOWER(laivo_tipas) LIKE N'žem%' THEN N'Uosto operacijų'

    WHEN LOWER(laivo_tipas) LIKE '%kitas%'
       OR LOWER(laivo_tipas) LIKE '%transporto%'
       OR LOWER(laivo_tipas) LIKE '%sportinis%'
       OR LOWER(laivo_tipas) LIKE '%vidaus%' THEN 'Kiti'

    ELSE laivo_tipas
END
WHERE
    LOWER(laivo_tipas) LIKE '%tanklaivis%'
 OR LOWER(laivo_tipas) LIKE '%bendr%'
 OR LOWER(laivo_tipas) LIKE '%ro-ro%'
 OR LOWER(laivo_tipas) LIKE N'%barž%'
 OR LOWER(laivo_tipas) LIKE '%biriems%'
 OR LOWER(laivo_tipas) LIKE '%avarinis%'
 OR LOWER(laivo_tipas) LIKE '%stumikas%'
 OR LOWER(laivo_tipas) LIKE '%vilkikas%'
 OR LOWER(laivo_tipas) LIKE N'%priemonėms'
 OR LOWER(laivo_tipas) LIKE '%pagalbinis%'
 OR LOWER(laivo_tipas) LIKE '%kranas%'
 OR LOWER(laivo_tipas) LIKE '%pontonas%'
 OR LOWER(laivo_tipas) LIKE '%mokymo%'
 OR LOWER(laivo_tipas) LIKE '%nesavaeigis%'
 OR LOWER(laivo_tipas) LIKE '%trauleris%'
 OR LOWER(laivo_tipas) LIKE '%hidrografinis%'
 OR LOWER(laivo_tipas) LIKE N'žem%'
 OR LOWER(laivo_tipas) LIKE '%kitas%'
 OR LOWER(laivo_tipas) LIKE '%transporto%'
 OR LOWER(laivo_tipas) LIKE '%sportinis%'
 OR LOWER(laivo_tipas) LIKE '%vidaus%';
```

Then I checked the ship registration country - **registracijos_valstybes_pavadinimas** - column.

Power BI can accept Lithuanian country names and perfectly plot them on the map. However, I saw that there are quite a few entries where the country or territory name will not be translated correcly by **Power BI**, therefore, some standartization is needed. To be safe, I changed the incorrect translations to English to be 100% sure that **Power BI** plots them correctly.

```sql
UPDATE Laivas_edited
SET registracijos_valstybes_pavadinimas = CASE
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE '%mergel%' THEN 'Virgin Islands'
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE '%britanija%' THEN 'United Kingdom'
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE 'farer%' THEN 'Faroe Islands'
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE 'kaiman%' THEN 'Cayman Islands'
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE N'maršalo%' THEN 'Marshall Islands'
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE 'olandijos%' THEN 'Aruba'
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE '%arabija%' THEN 'Saudi Arabia'
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE '%kristoferis%' THEN 'Saint Kitts'
    WHEN LOWER(registracijos_valstybes_pavadinimas) LIKE '%vinsentas%' THEN 'Saint Vincent and the Grenadines'
    ELSE registracijos_valstybes_pavadinimas
END
WHERE
    LOWER(registracijos_valstybes_pavadinimas) LIKE '%mergel%'
 OR LOWER(registracijos_valstybes_pavadinimas) LIKE '%britanija%'
 OR LOWER(registracijos_valstybes_pavadinimas) LIKE 'farer%'
 OR LOWER(registracijos_valstybes_pavadinimas) LIKE 'kaiman%'
 OR LOWER(registracijos_valstybes_pavadinimas) LIKE N'maršalo%'
 OR LOWER(registracijos_valstybes_pavadinimas) LIKE 'olandijos%'
 OR LOWER(registracijos_valstybes_pavadinimas) LIKE '%arabija%'
 OR LOWER(registracijos_valstybes_pavadinimas) LIKE '%kristoferis%'
 OR LOWER(registracijos_valstybes_pavadinimas) LIKE '%vinsentas%';
```

The same problem appears in the column of the country the ship is arriving from/departing to - **atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas**. I standardized the country names in the same manner as above.

```sql
UPDATE Laivas_edited
SET atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas = CASE
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'd.kongas%' THEN 'Democratic Republic of the Congo'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'dominikos%' THEN 'Dominican Republic'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'dramblio%' THEN 'Ivory Coast'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE '%britanija%' THEN 'United Kingdom'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'farer%' THEN 'Faroe Islands'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE '%zelandija%' THEN 'New Zealand'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'par' THEN 'South Africa'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE '%arabija%' THEN 'Saudi Arabia'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'trinidadas' THEN 'Trinidad and Tobago'
    WHEN LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE N'žaliojo%' THEN 'Cape Verde'
    ELSE atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas
END
WHERE
    LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'd.kongas%'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'dominikos%'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'dramblio%'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE '%britanija%'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'farer%'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE '%zelandija%'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'par'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE '%arabija%'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE 'trinidadas'
 OR LOWER(atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas) LIKE N'žaliojo%';
```

Lastly I saw that the column of the country the ship is arriving from/departing to - **atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas** - has entries where the country is unknown and is marked as *NENURODYTA*. All the other columns with unknown or missing entries had them marked as NULL except for this column. Therefore, I changed the unknown entries to NULL to keep everything standartized.

```sql
UPDATE Laivas_edited
SET atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas = NULL
WHERE
LOWER([atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas]) = 'nenurodyta';
```

## EXPLORATORY DATA ANALYSIS AND VISUALIZATION

Since the data is now cleaned up and ready for visualization, I will do the following:

1. Do some simple calculations to get some general insights on ship length, ship tonage and to find the most popular shipping agent company;
2. Get the Top 10 most popular ship types, ship registration countries and countries of arrival;
3. Import the data into Power BI via SQL Server method and build a dashboard visualizing the points above.

From the ship type - **laivo_ilgis** - column I calculated the average ship length.

```sql
SELECT ROUND(AVG(laivo_ilgis),2) as average_ship_length FROM Laivas_edited;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Related%20Images/avg_ship_length.png)

From the ship tonage - **laivo_tonazas** - column I calculated the average tonage of ships in the Port of Klaipeda.

```sql
SELECT ROUND(AVG(laivo_tonazas),0) as average_ship_tonage FROM Laivas_edited;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Related%20Images/avg_ship_tonage.png)

From the agent company - **agento_imone** - column I found the most popular shipping agent company in the Port of Klaipeda.

```sql
SELECT TOP (1) agento_imone as most_popular_agent_company FROM Laivas_edited
WHERE agento_imone IS NOT NULL
GROUP BY agento_imone
ORDER BY COUNT(laivo_id) DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Related%20Images/most_popular_agent_company.png)

Since these three queries are returning a single value, I can place these values as cards in **Power BI**.

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Related%20Images/cards.png)

Now I moved on to the queries which will give me multiple values and will be used in bar charts in **Power BI** to demonstrate the particular tendencies.

I have standarized the ship type - **laivo_tipas** - column and now I could use it to get Top 10 most popular ship types in the Port of Klaipeda.

```sql
SELECT TOP (10) laivo_tipas as most_popular_ship_types, COUNT(laivo_id) as number_of_ships FROM Laivas_edited
GROUP BY laivo_tipas
ORDER BY COUNT(laivo_id) DESC;
```
![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Related%20Images/most_popular_ship_types.png)

I did the same with with **registracijos_valstybes_pavadinimas** and **atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas** columns.

```sql
SELECT TOP (10) registracijos_valstybes_pavadinimas as most_popular_ship_registration_countries, COUNT(laivo_id) as number_of_ships FROM Laivas_edited
GROUP BY registracijos_valstybes_pavadinimas
ORDER BY COUNT(laivo_id) DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Related%20Images/most_popular_ship_registration_countries.png)

```sql
SELECT TOP (10) atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas as from_where_do_ships_arrive_to_Klaipeda, COUNT(laivo_id) as number_of_ships FROM Laivas_edited
WHERE laivo_plaukimo_kryptis = 'ATV' AND atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas IS NOT NULL
GROUP BY atvykimo_ar_isvykimo_uosto_valstybes_pavadinimas
ORDER BY COUNT(laivo_id) DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/refs/heads/main/Related%20Images/from_where_do_ships_arrive_to_Klaipeda.png)

After getting the wanted queries and values, I used **Power BI** to create a nice and clean dashboard showcasing our results and trends that are happening in the Port of Klaipeda.

![alt text](https://github.com/robertasdvarionas/Shipping-Tendencies-in-Klaipeda/blob/main/Related%20Images/Klaipedos%20Uosto%20Tendencijos_page-0001.jpg)

I added the slicer to side of the dashboard to be able to view the trends yearly and added a map visual to display from where do the ships tend to come to Klaipeda.

## CONCLUSION

In this project I have applied **SQL** and **Power BI** to understand and visualise what shipping tendencies or insights can be drawn in the port of Klaipėda.

1. We can see that the dominant ship types in the Port of Klaipeda are General Cargo, Ro-Ro and Container vessels.

2. The most popular ship registration countries are Lithuania, Cyprus and Denmark. Lithuania being number one is indeed expected as the majority of the fishing, port operation and admin vessels are registered in Lithuania and these vessel types make up a large part of the total number of vessels.

3. The ships that are arriving to Klaipeda are mostly from Sweden, Germany, Poland and Denmark which is again an expected finding knowing that Lithuania together with all of these countries are located by and along the Baltic Sea which makes for an easy partnership and shipping line creation.

4. The average ship length and average tonage can be an important information to the port engineers and planners when further developing the port.
