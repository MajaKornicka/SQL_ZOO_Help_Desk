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




