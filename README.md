# Murder Mystery
Interesting case of a murder that happened in SQL city. Can murder mysteries be solved using SQL? Find out now
![murder image](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/2edf4211-b1ec-414a-aa65-fc37e83fc951)

## Case Introduction 
In this murder mystery challenge, we will investigate a crime that occurred on January 15, 2018, in SQL City. The goal is to retrieve the crime scene report and identify the person responsible for the murder. This document outlines the thought process, narrative, and SQL codes used to solve the mystery. 

## Database Information 
To begin the investigation, I accessed the provided SQLite database using the online tool available at https://www.sqliteonline.com. After loading the database file, I examined the tables and their contents to gather relevant information.
![schema](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/87480b85-c4c0-4290-800d-d3ed9d77aa0a)<br>
The tables contained in this database will aid my understanding of the crime situation and eventually pinpoint the person behind this murder. 

## Approach and Analysis 
My approach towards this is to initially retrieve the crime scene report to identify potential suspects, witnesses or any other clues that might lead me to the perpetrator of the crime. 
``` sql 
SELECT *  --use the police report to check for possible suspects
FROM crime_scene_report
WHERE date = '20180115' and city = 'SQL City' and type = 'murder'
``` 
![pol repo](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/224c09fd-c451-4808-9e07-4558ecf62760)
<br>
The description from the police report says; 
> Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave"
>
The next task is to find the 2 witnesses. I’ll start with the witness I have more details of, which is Annabel. 

``` sql 
SELECT * 
FROM person 
WHERE name LIKE '%Annabel%' AND address_street_name = 'Franklin Ave'
``` 
![ANN](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/d9fbed23-a06e-428d-9832-4f0fc18038fd)
<br>
<br>
The name of the first witness is Annabel Miller. Her whereabouts need to be investigated in order to find out the possible location where the crime was committed. I searched for her details and checked if she visited the gym. <br>
I’ve made use of 2 subqueries, one to store the result of the last query (stored as Annabel) and then used it to join the get_fit_now_member table to check if she’s a member of the gym and finally if she’s a member, was she there on the day of the crime. The second subquery is stored as "ann_gym". <br>

``` sql 
  -- let's find annabel's details to fully track her movement on the day of the crime
WITH annabel as(  
  SELECT * 
  from person 
  where name LIKE '%Annabel%' and address_street_name = 'Franklin Ave'
    ),
  -- was she at the gym?
ann_gym as (
  SELECT m.id, a.name, license_id, person_id, membership_start_date, membership_status
  from annabel as a 
  join get_fit_now_member m
  on a.id = m.person_id
    )
SELECT 
	id, name, check_in_date, check_in_time, check_out_time, license_id
from get_fit_now_check_in c
  LEFT join ann_gym g 
    on g.id = c.membership_id
where membership_id in 
	(SELECT id from ann_gym)
      )
  ```
![Picture3](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/a186d123-e731-4049-81e8-6218c825bfe9)
<br>
The output of this query shows:
-	She’s a member of the gym. 
-	The last time she was at the gym was the 9th of January. 
If she was not at the gym on the day of the crime, where was she then? Mhm, only one place to check, the Facebook event! 
``` sql 
WITH annabel as(  
  SELECT * 
  from person 
  where name LIKE '%Annabel%' and address_street_name = 'Franklin Ave'
    )
SELECT * 
from facebook_event_checkin f 
join annabel a 
on a.id = f.person_id
``` 
![Picture4](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/bea6324d-2c6c-4ec8-8b81-06608e898a0e) <br>
Voila! She was indeed at the Facebook event. Since Annabel is a witness, we can conclude that the crime happened at the Facebook event.<rbr>
Before moving on to the next witness, what was said in Annabel’s witness report? <br>  
```
WITH annabel as(  
  SELECT * 
  from person 
  where name LIKE '%Annabel%' and address_street_name = 'Franklin Ave'
    )
SELECT *
from interview i
join annabel a 
on i.person_id = a.id
```
> "I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th” she says
>
Interesting. Now we know that the prime suspect was indeed present at the gym on the 9th of January. That’s all the information we need from Annabel for now. <br>

Remember, according to the police report, there were two witnesses. The other witness lives at the last house on “Northwestern Dr.” Let’s write some sql query to find the last house on this street. 
``` sql 
SELECT * 
FROM person 
WHERE address_street_name LIKE '%Northwestern%'
ORDER BY address_number desc 
LIMIT 1
```
![Picture6](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/99133ccc-679f-44ff-b4ac-76129cbb070f)


The name of our 2nd witness is Morty Shapiro! This begs the question, which places did Morty visit on the day of the crime? Was he at the gym on the day of the Facebook event? These are pertinent questions that need to be answered. Let’s investigate! 
``` sql
WITH morty as (
SELECTWHEREORDER BYfrom person 
WHEREORDER BYdress_street_name LIKE '%Northwestern%'
ORDER BY address_number desc 
LIMIT 1 )

-- was morty at the gym? --
SELECT m.id, m.name, license_id, person_id, membership_start_date, membership_status
FROM get_fit_now_member g
JOIN morty m
ON m.id = g.person_id
```
This query returns an empty result set which means Morty is not a member of the gym, therefore he couldn’t have been there. <br>
He’s a witness so he must have been at the Facebook party, let’s confirm this assumption. 
``` sql 
WITH morty as (
SELECT * from person 
WHERE address_street_name LIKE '%Northwestern%'
ORDER BY address_number desc 
LIMIT 1 )

SELECT *
FROM morty m 
JOIN facebook_event_checkin f 
ON m.id = f.person_id
```
![Picture7](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/8159a7e0-6666-4eb2-ada7-cfd565711e27)

Morty indeed checked in for the Funky Grooves Tour on the 15th of January, the day the crime happened. His whereabouts for the day have been ascertained, and now it’s time to get his witness report. 
``` sql 
WITH morty as (
SELECT * 
FROM person 
WHERE address_street_name LIKE '%Northwestern%'
ORDER BY address_number desc 
LIMIT 1
)
SELECT * 
FROM interview i 
JOIN morty m 
ON i.person_id = m.id
```
> “I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W." 
> 
This was Morty’s report, and he has given some extremely helpful cues to lead me down the criminal’s trail. 
So far, these are the insights drawn from the witnesses’ reports: 
-	The suspect was at the gym on January 9th.
-	The person has a gold membership. 
-	The membership id contains “48Z”.
-	A car with plate number containing “H42W” was used to perpetrate the crime. 

## FINDING THE CRIMINAL(S)
From here on in, finding the prime suspect(s) seems straightforward thanks to Morty’s keen eye for detail. I’ll use some of the descriptions above to narrow down the search.
``` sql 
SELECT * 
FROM get_fit_now_check_in c 
JOIN get_fit_now_member m 
ON m.id = c.membership_id
WHERE check_in_date = '20180109' AND membership_status = 'gold' AND membership_id LIKE '%48Z%'
``` 
![Picture8](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/de1128d9-9e3e-4d7b-a9f9-1589b61e8926)

We have 2 gold members that were at the gym on the 9th of January where Annabel claimed to have seen the suspect. Were these two at the scene of the crime? Let’s find out!
<br>
```
with prime_suspects as(
SELECT * 
FROM get_fit_now_check_in c 
JOIN get_fit_now_member m 
ON m.id = c.membership_id
WHERE check_in_date = '20180109' AND membership_status = 'gold' AND membership_id LIKE '%48Z%')

SELECT *
FROM facebook_event_checkin f 
JOIN prime_suspects pm 
USING (person_id)
```
*Moment of Truth*
	
![Picture9](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/646950fb-e4c2-4e2e-9f1a-5c30dfb042e6)

Wow! Seems we have our CRIMINAL!
Only Jeremy Bowers was at the scene of the crime. Surely this means it was him who committed the crime, right? Well, everyone has a right to fair hearing so, what does he have to say? <br>
Let’s retrieve this from the interview table. 
``` sql
WITH prime_suspects as(
SELECT * 
FROM get_fit_now_check_in c 
JOIN get_fit_now_member m 
ON m.id = c.membership_id
WHERE check_in_date = '20180109' AND membership_status = 'gold' AND membership_id LIKE '%48Z%')

SELECT *
FROM interview i 
JOIN prime_suspects pm 
USING (person_id)
```
> “I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.”
>
You nasty criminal! <br>
I need to figure out who is the lynchpin behind this murder. 
``` sql
SELECT p.id, p.name, p.address_street_name, d.hair_color, d.gender
FROM person p 
LEFT JOIN drivers_license d 
ON d.id = p.license_id
LEFT JOIN income i 
ON i.ssn = p.ssn 
WHERE height BETWEEN 65 and 67 AND d.hair_color = 'red' AND car_make = 'Tesla' and gender = 'female' 
AND p.id IN (SELECT person_id
             FROM facebook_event_checkin 
             WHERE event_name = 'SQL Symphony Concert' and date like '%201712__'
             GROUP by person_id
             HAVING COUNT(*) =3 
            )
```
![Picture10](https://github.com/Saigovernor/Murder-Mystery/assets/118802056/a4ec7302-e1cc-490b-9ae7-2e17e4182cb2)
	
From this query, it seems that Jeremy Bowers was sent by Miranda Priestly. 


