#### 1. Review the crime scene report for the murder date.
> [!TIP]
> Check the [ERD](https://github.com/NUKnightLab/sql-mysteries/blob/master/schema.png) for the crime scene report table. Read the instructions for the date of the murder (Jan 15, 2018).

<details>
<summary>Click to see the query</summary>

```sql
select * 
from crime_scene_report 
where date = '20180115' and type = 'murder';
```
</details>

With the query above, I was able to get some information about the witnesses. In order to identify these withness, I explored further with the latest information.

<details>
<summary>Click to view the query result</summary>
	
	1. First witness lives at the last house on "Northwestern Dr".
	2. Second witness, named Annabel, lives somewhere on "Franklin Ave".
_PS: This is a summerised version of the report_
</details>


#### 2. Check the person table for other information about Witness 1. 
> [!TIP]
> Order the table in descending order since the witness lives in the last house.

<details>
<summary>Click to see the query</summary>

```sql
select *
from person
where address_street_name = 'Northwestern Dr'
order by address_number desc;
```
</details>

With the query above, I got more information about the Witness 1.

_PS: The query returned more than 1 rows. I included the correct row on this table for brevity._

|id 		|name            |license_id  |address_number |address_street_name	|        ssn|
|------:|----------------|------------|---------------|---------------------|-----------|
|	 14887|	 Morty Schapiro|    	118009|	          4919|			 Northwestern Dr|  111564949|

#### 3. Check the person table for other information about Witness 2.
> [!TIP]
> Remember to use the tip from the crime scene report about witness 2.

<details>
<summary>Click to see the query</summary>

```sql
select *
from person
where address_street_name = 'Franklin Ave' and name like '%Annabel%'
```
</details>

|id 		|name            |license_id  |address_number |address_street_name	|        ssn|
|------:|----------------|------------|---------------|---------------------|-----------|
|	 16371|	 Annabel Miller|    	490173|	           103|			    Franklin Ave|  318771143|


#### 4. Check and review the interview of the witnesses
> [!TIP]
> Use the info about the witnesses to query the interview table.

<details>
<summary>Click to see the query</summary>

```sql
select interview.person_id, interview.transcript, person.name
from interview
join person
on interview.person_id = person.id
where interview.person_id in (14887, 16371);
```
</details>

Below is the summary of the witness interviews:
1. Account 1:
  * Had a "Get Fit Now Gym" bag.
  * Membership number on the bag started with "48Z".
  * Gold members have those bags.
  * Car plate included "H42W".
2. Account 2:
* Recognized the killer from their gym when they were working out last week on January the 9th.

#### 5. From the information from the query above, check for people who checked in at the gym for Jan 9, 2018.

<details>
<summary>Click to see the query</summary>

```sql
select *
from get_fit_now_check_in
where check_in_date = '20180109' and membership_id like '48Z%'
```
</details>

I got 2 records that match this information. However, I need to examine these two suspects further and figure the killer. See table below:

|membership_id	| check_in_date |	check_in_time |	check_out_time|
|--------------:|---------------|---------------|---------------|
|					 48Z7A| 			20180109|						1600|						1730|
|					 48Z55|			  20180109|				 	  1530|				 		1700|

#### 6. Get more information about the two suspects from the gym member table.

<details>
<summary>Click to see the query</summary>

```sql
select *
from get_fit_now_check_in
join get_fit_now_member
on get_fit_now_member.id = get_fit_now_check_in.membership_id
where get_fit_now_check_in.check_in_date = '20180109' and get_fit_now_check_in.membership_id like '48Z%';
```
</details>

From the query above, I got some information about the suspects that can help figure the person that committed the murder. 

|membership_id| person_id |      name     |	membership_status |
|------------:|-----------|---------------|-------------------|
|			   48Z7A|      28819|   Joe Germuska|								gold|
|			   48Z55|      67318|  Jeremy Bowers|								gold|

#### 7. You can further check the the person table for more information about these suspects.

<details>
<summary>Click to see the query</summary>

```sql
select *
from person
join get_fit_now_member 
on get_fit_now_member.person_id = person.id
where person.id in (28819, 67318)
```
</details>

|id 		|name            |license_id |address_number |address_street_name	 |        ssn|
|------:|----------------|-----------|---------------|---------------------|-----------|
|28819	|Joe Germuska	   |173289	   | 				  	111| 				   		Fisk Rd| 138909730 |
|67318	|Jeremy Bowers   |423327	   | 				  	530|Washington Pl, Apt 3A| 871539279 |

> [!TIP]
> Can you figure the column that we'll need for the next steps? You guessed right - License Id.

#### 8. Use the information from the person table to query the license table for more information about the suspect.
<details>
<summary>Click to see the query</summary>

```sql
select *
from person
join drivers_license 
on drivers_license.id = person.license_id
where person.license_id in (173289, 423327)
and drivers_license.plate_number like '%H42W%';
```
</details>

#### 9. Viola!!! You have the murder!!!

|id 		|name            |license_id |address_number |address_street_name	 |        ssn|
|------:|----------------|-----------|---------------|---------------------|-----------|
|67318	|Jeremy Bowers   |423327	   | 				  	530|Washington Pl, Apt 3A| 871539279 |

#### 10. You can explore further to get the real villain behind this murder. Are you up for it? 
> [!TIP]
> Explore the Interview table for the transcript of the killer.

<details>
<summary>Click to see the query</summary>

```sql
select interview.person_id, interview.transcript, person.name
from interview
join person
on interview.person_id = person.id
where interview.person_id = 67318
```
</details>

#### 10. Finally, you can get the Villain by carefully combining the information you have so far.

<details>
<summary>Click to see the query</summary>
	
```
select person.id, person.name
from person
join drivers_license on drivers_license.id = person.license_id
join facebook_event_checkin on facebook_event_checkin.person_id = person.id
where drivers_license.hair_color = 'red'
and drivers_license.car_make = 'Tesla'
and drivers_license.height between 65 and 67
and drivers_license.car_model = 'Model S'
and facebook_event_checkin.event_name = 'SQL Symphony Concert'
group by person.id, person.name
having count(facebook_event_checkin.event_id) >= 3
```
</details>

There you, Miranda Priestly is the the Villain behind the murder.
|id 		|name             |
|------:|-----------------|
|99716	|Miranda Priestly |


Thanks for checking my solutions to the Murder in [SQL City game](https://mystery.knightlab.com/). Let me know if you have any questions.
