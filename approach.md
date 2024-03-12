-- Review the Crime Scene report for the murder date (Jan 15, 2018)
'select *
from crime_scene_report
where date = '20180115' and type = 'murder''

-- I was able to get some info about the witnesses 
First witness lives at the last house on "Northwestern Dr". 
Second witness, named Annabel, lives somewhere on "Franklin Ave".

-- Check person table to see info about witness 1 
select *
from person
where address_street_name = 'Northwestern Dr'
order by address_number desc

Witness 1: 14887, 'Morty Schapiro', 118009, 4919, 'Northwestern Dr',	111564949.

-- Check person table for info about witness 2 
select *
from person
where address_street_name = 'Franklin Ave' and name like '%Annabel%'

Witness 2: 16371,	'Annabel Miller',	490173,	103,	'Franklin Ave',	318771143.

-- Check and review interview of the witnesses
select interview.person_id, interview.transcript, person.name
from interview
join person
on interview.person_id = person.id
where interview.person_id in (14887, 16371);

---- Account 1: Had a "Get Fit Now Gym" bag. 
                Membership number on the bag started with "48Z". 
                Gold members have those bags. 
                Car plate included "H42W".
---- Account 2: Recognized the killer from my gym when I was working out last week on January the 9th.

-- Check the checkin at the gym for Jan 9, 2018
select *
from get_fit_now_check_in
where check_in_date = '20180109' and membership_id like '48Z%'

48Z7A	20180109	1600	1730
48Z55	20180109	1530	1700

-- Get info about the suspects from the gym table
select *
from get_fit_now_check_in
join get_fit_now_member
on get_fit_now_member.id = get_fit_now_check_in.membership_id
where get_fit_now_check_in.check_in_date = '20180109' and get_fit_now_check_in.membership_id like '48Z%'

48Z7A	28819	Joe Germuska	20160305	gold
48Z55	67318	Jeremy Bowers	20160101	gold

-- Name of the suspects from the person table
select *
from person
join get_fit_now_member 
on get_fit_now_member.person_id = person.id
where person.id in (28819, 67318)

28819	'Joe Germuska'	173289	111	'Fisk Rd'	138909730
67318	'Jeremy Bowers'	423327	530	'Washington Pl, Apt 3A'	871539279	

-- Check the License table for info about the supect
select *
from person
join drivers_license 
on drivers_license.id = person.license_id
where person.license_id in (173289, 423327)

-- The suppect appears to be Jeremy Bowers
67318	Jeremy Bowers	423327	530	Washington Pl, Apt 3A	871539279

-- To get the real villain, explore the Interview table further.
select interview.person_id, interview.transcript, person.name
from interview
join person
on interview.person_id = person.id
where interview.person_id = 67318

-- The Villain
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

99716	Miranda Priestly


