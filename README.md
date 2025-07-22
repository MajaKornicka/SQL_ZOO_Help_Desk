# SQL Zoo Help Desk ℹ️

Data and Questions Source: [https://sqlzoo.net/wiki/Guest_House](https://sqlzoo.net/wiki/Help_Desk)

Solutions written using MySQL

## Scenario
A software company has been successful in selling its products to a number of customer organisations, and there is now a high demand for technical support. There is already a system in place for logging support calls taken over the telephone and assigning them to engineers, but it is based on a series of spreadsheets. With the growing volume of data, using the spreadsheet system is becoming slow, and there is a significant risk that errors will be made.


<img width="752" height="566" alt="image" src="https://github.com/user-attachments/assets/05a03441-5495-4cd9-8424-f27b1a2463d0" />


## Easy questions

1.There are three issues that include the words "index" and "Oracle". Find the call_date for each of them

```sql
SELECT call_date, call_ref 
FROM Issue
WHERE Detail LIKE'%index%'
AND Detail LIKE '%Oracle%'
```
Result
|call_date|call_ref|
|---------|--------|
|Sat, 12 Aug 2017 16:00:00 GMT|1308|
|Wed, 16 Aug 2017 14:54:00 GMT|1697|
|Wed, 16 Aug 2017 19:12:00 GMT|1731|


<br>
2. Samantha Hall made three calls on 2017-08-14. Show the date and time for each

```sql
SELECT Call_date, First_name, last_name
FROM Issue
LEFT JOIN Caller ON Caller.Caller_id = Issue.Caller_id
WHERE First_name = 'Samantha'
AND Last_name = 'Hall'
AND call_date LIKE '2017-08-14%'
```

Result
|Call_date|First_name|Last_name|
|---------|---------|-------|
|Mon, 14 Aug 2017 10:10:00 GMT|Samantha|Hall|
|Mon, 14 Aug 2017 10:49:00 GMT|Samantha|Hall|	
|Mon, 14 Aug 2017 18:18:00 GMT|Samantha|Hall|

<br>

3. There are 500 calls in the system (roughly). Write a query that shows the number that have each status.

```sql
SELECT status, COUNT(*) as total_calls
FROM Issue
GROUP BY status
```
Result

|status|total_calls|
|------|-----------|
|Closed|486|
|Open|10|


<br>
4. Calls are not normally assigned to a manager but it does happen. How many calls have been assigned to staff who are at Manager Level?


```sql
SELECT COUNT(*) as manager_assigned
FROM Issue
WHERE Assigned_to IN (
      SELECT Staff_code
      FROM Staff
      LEFT JOIN Level ON Staff.Level_code = Level.Level_code
      WHERE Manager = 'Y' )

```
Result
|manager_assigned|
|--|
|51|

<br>
5. Show the manager for each shift. Your output should include the shift date and type; also the first and last name of the manager.

```sql
SELECT shift_date, Shift_Type, First_name, Last_name
FROM Shift
LEFT JOIN Staff ON Shift.Manager = Staff.Staff_code
ORDER BY Shift_date
```

Result

|Shift_date|Shift_Type	|First_name	|Last_name|
|-----------|----------|----------|-------------|
|Sat, 12 Aug 2017 00:00:00 GMT	|Early	|Logan	|Butler|
|Sat, 12 Aug 2017 00:00:00 GMT	|Late	|Ava	|Ellis|
|Sun, 13 Aug 2017 00:00:00 GMT	|Early	|Ava	|Ellis|
|Sun, 13 Aug 2017 00:00:00 GMT	|Late	|Ava	|Ellis|
|Mon, 14 Aug 2017 00:00:00 GMT	|Early	|Logan	|Butler|
|Mon, 14 Aug 2017 00:00:00 GMT	|Late	|Logan	|Butler|
|Tue, 15 Aug 2017 00:00:00 GMT	|Early	|Logan	|Butler|
|Tue, 15 Aug 2017 00:00:00 GMT	|Late	|Logan	|Butler|
|Wed, 16 Aug 2017 00:00:00 GMT	|Early	|Logan	|Butler|
|Wed, 16 Aug 2017 00:00:00 GMT	|Late	|Logan	|Butler|



## Medium questions

6. List the Company name and the number of calls for those companies with more than 18 calls.

```sql
SELECT  Company_name, COUNT(call_ref) as total_calls
FROM Issue
LEFT JOIN Caller ON Issue.Caller_id = Caller.Caller_id
LEFT JOIN Customer ON Customer.Company_ref = Caller.Company_ref
GROUP BY  Company_name
HAVING COUNT(call_ref) >18
```

Result
|Company_name	|total_calls|
|------------|-------------|
|Gimmick Inc.	|22|
|Hamming Services	|19|
|High and Co.	|20|

<br>

7. Find the callers who have never made a call. Show first name and last name

```sql
SELECT First_name, Last_name
FROM Caller
LEFT JOIN Issue ON Caller.Caller_id = Issue.Caller_id
WHERE Call_date IS NULL

```
Result

|First_name	|Last_name|
|---------|---------|
|David	|Jackson|
|Ethan	|Phillips|

<br>
8. For each customer show: Company name, contact name, number of calls where the number of calls is fewer than 5

```sql
WITH company_calls AS(
SELECT company_name, COUNT(call_ref) as nc
FROM Customer c 
JOIN Caller cl ON c.company_ref = cl.company_ref
JOIN Issue i ON cl.caller_id = i.caller_id
GROUP BY company_name
HAVING nc < 5)

SELECT Customer.company_name, First_name, Last_name , nc
FROM Customer
LEFT JOIN Caller ON Caller.Caller_id = Customer.contact_Id
INNER JOIN Company_calls cc ON cc.company_name = Customer.company_name
```

Result


|company_name|	First_name	|Last_name	|nc|
|---------|-------------------|----------|----|
|Somebody Logistics	|Ethan	|Phillips	|2|
|Pitiable Shipping	|Ethan	|McConnell	|4|
|Rajab Group	|Emily	|Cooper	|4|


<br>

9. For each shift show the number of staff assigned. Beware that some roles may be NULL and that the same person might have been assigned to multiple roles (The roles are 'Manager', 'Operator', 'Engineer1', 'Engineer2').

```sql
WITH employees AS(
SELECT Shift_date, Shift_type, Manager as employee
FROM Shift

UNION ALL

SELECT Shift_date, Shift_type, Operator as employee
FROM Shift

UNION ALL 

SELECT Shift_date, Shift_type, Engineer1 as employee
FROM Shift

UNION ALL

SELECT Shift_date, Shift_type, ENGINEER2 as employee
FROM Shift)

SELECT Shift_date, Shift_type, COUNT(DISTINCT employee) as total_employees
FROM employees
WHERE employee IS NOT NULL
GROUP BY Shift_date, Shift_type
```

Result

|Shift_date	|Shift_type	|total_employees|
|--------|----------|---------------|
|Sat, 12 Aug 2017 00:00:00 GMT	|Early	|4|
|Sat, 12 Aug 2017 00:00:00 GMT	|Late	|4|
|Sun, 13 Aug 2017 00:00:00 GMT	|Early|	3|
|Sun, 13 Aug 2017 00:00:00 GMT	|Late	|2|
|Mon, 14 Aug 2017 00:00:00 GMT	|Early|	4|
|Mon, 14 Aug 2017 00:00:00 GMT	|Late	|4|
|Tue, 15 Aug 2017 00:00:00 GMT	|Early|	4|
|Tue, 15 Aug 2017 00:00:00 GMT	|Late	|4|
|Wed, 16 Aug 2017 00:00:00 GMT	|Early|	4|
|Wed, 16 Aug 2017 00:00:00 GMT	|Late|	4|

<br>

10. Caller 'Harry' claims that the operator who took his most recent call was abusive and insulting. Find out who took the call (full name) and when.



```sql

SELECT First_name, Last_name, Call_Date
FROM Issue
JOIN Staff ON Staff.Staff_code = Issue.Taken_by
WHERE caller_id IN 
               (SELECT Caller_id FROM Caller WHERE First_name = 'Harry')
ORDER BY Call_date DESC
LIMIT 1
```

Result

|First_name	|Last_name	|Call_Date|
|------------|----------|---------|
|Emily	|Best|	Wed, 16 Aug 2017 10:25:00 GMT|


## Hard questions

11. Show the manager and number of calls received for each hour of the day on 2017-08-12


```sql
WITH Shift_split AS(
SELECT Shift_date, Shift.Shift_type, manager, Start_time, End_time
FROM Shift
LEFT JOIN Shift_type ON Shift.Shift_type = Shift_type.Shift_type
WHERE Shift_date LIKE '2017-08-12%'),

calls_by_hour AS(
SELECT DATE_FORMAT(Call_date,  '%Y-%m-%d') as date, HOUR(Call_date) as time, Call_ref
FROM Issue
WHERE Call_date LIKE '2017-08-12%')

SELECT Manager, CONCAT(date, ' ', time) as hour, COUNT(*) as total_calls
FROM calls_by_hour cbh
LEFT JOIN shift_split ss ON cbh.time BETWEEN ss.start_time AND ss.end_time
GROUP BY Manager, CONCAT(date, ' ', time)
ORDER BY hour
```


Result

|manager|date|time|total_calls|
|-------|----|----|----------|
|LB1	|2017-08-12 |8|   6|
|LB1	|2017-08-12	|9|	16|
|LB1	|2017-08-12	|10|	11|
|LB1	|2017-08-12	|11|	6|
|LB1	|2017-08-12	|12|	8|
|LB1	|2017-08-12	|13|	4|
|LB1	|2017-08-12	|14|	12|
|AE1	|2017-08-12	|14|	12|
|AE1	|2017-08-12	|15|	8|
|AE1	|2017-08-12	|16|	8|
|AE1	|2017-08-12	|17|	7|
|AE1  |2017-08-12	|19|	5|


<br>

12. 80/20 rule. It is said that 80% of the calls are generated by 20% of the callers. Is this true? What percentage of calls are generated by the most active 20% of callers.

```sql
WITH Ranked AS (
  SELECT Caller_id, COUNT(Call_ref) as total_calls,
         ROW_NUMBER() OVER (ORDER BY COUNT(Call_ref) DESC) AS rn,
         COUNT(*) OVER () AS total
  FROM Issue
  GROUP BY Caller_id
)
SELECT SUM(total_Calls) / (SELECT COUNT(*) FROM Issue) *100 as calls_generated_by_top_20_perc_of_callers
FROM Ranked
WHERE rn <= total * 0.2
```

Result

|calls_generated_by_top_20_perc_of_callers|
|-----|
|32.2581|


<br>
13. Annoying customers. Customers who call in the last five minutes of a shift are annoying. Find the most active customer who has never been annoying.


```sql
WITH which_callers AS(
SELECT call_date, CAST(call_date AS TIME) AS time, Caller_id
FROM Issue),

annoying_callers AS(
SELECT Caller_id as nice_caller
FROM which_callers wc
WHERE (HOUR(time) = 13 AND MINUTE(time) >=55)
   OR (HOUR(time) = 19 AND MINUTE(time) >=55) )

SELECT Company_name, COUNT(*) as total_calls
FROM Issue
JOIN Caller ON Issue.Caller_id = Caller.Caller_id
JOIN Customer ON Customer.company_Ref = Caller.company_Ref
WHERE Issue.Caller_id NOT IN (SELECT* FROM annoying_callers)
GROUP BY Company_name
ORDER BY COUNT(*) DESC
LIMIT 1

```

Result


|Company_name	|total_calls|
|--------------|-------------|
|High and Co.	|20|


<br>

14. Maximal usage. If every caller registered with a customer makes at least one call in one day then that customer has "maximal usage" of the service. List the maximal customers for 2017-08-13.






































