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
Before moving on to the next witness, what was said in Annabel’s witness report?
<br>

