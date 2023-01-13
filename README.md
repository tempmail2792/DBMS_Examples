Examples of Database Management Systems 2nd Edition, Raghu Ramakrishnan.

![dbms4.17](https://raw.githubusercontent.com/NLPpupil/markdown-images/master/dbs4.17.png)

**(Q1)** Find the names of sailors who have reserved boat 103.

```sql
SELECT S.sname 
FROM Sailors S, Reserves R
WHERE S.sid = R.sid and R.bid=103
-- we recommend the explicit use of range variables and full qualification of all occurrences of columns with a range variable to improve the readability of your queries. We will follow this convention in all our examples.

-- or nested
SELECT S.sname
FROM Sailors S
WHERE S.sid IN(SELECT R.sid
              FROM Reserves R 
              WHERE R.bid = 103 )
```



**(Q2)** Find the names of sailors who have reserved a red boat.

```sql
SELECT S.sname
FROM Sailors S, Boats B, Reserves R
WHERE B.color='red' AND B.bid=R.bid AND S.sid = R.sid

-- or nested
SELECT S.sname
FROM Sailors S
WHERE S.sid IN ( SELECT R.sid
                 FROM Reserves R
                 WHERE R.bid IN(SELECT B.bid
                                FROM Boats B
                                WHERE B.color = ‘red’ )
```

**(Q3)** Find the colors of boats reserved by Lubber.

```sql
SELECT B.color
FROM Sailors S, Reserves R, Boats B
WHERE S.sname='Lubber' AND S.sid=R.sid AND R.bid = B.bid 
```

**(Q4)** Find the names of sailors who have reserved at least one boat.

```sql
SELECT S.sname
FROM Sailors S, Reserves R
WHERE S.sid = R.sid
```

**(Q5)** Find the names of sailors who have reserved a red or a green boat.

```sql
SELECT S.sname
FROM Sailors S, Reserves R, Boats B 
WHERE S.sid = R.sid AND R.bid = B.bid
      AND (B.color = ‘red’ OR B.color = ‘green’)

-- or
SELECT S.sname
FROM Sailors S, Reserves R, Boats B
WHERE S.sid = R.sid AND R.bid = B.bid AND B.color = ‘red’
UNION
SELECT S2.sname
FROM Sailors S2, Boats B2, Reserves R2
WHERE S2.sid = R2.sid AND R2.bid = B2.bid AND B2.color = ‘green’
```



**(Q6)** Find the names of sailors who have reserved a red and a green boat. 

```sql
SELECT S.sname
FROM Sailors S, Reserves R1, Boats B1, Reserves R2, Boats B2
WHERE S.sid = R1.sid 
      AND R1.bid = B1.bid
      AND S.sid = R2.sid 
      AND R2.bid = B2.bid
      AND B1.color=‘red’ 
      AND B2.color = ‘green’
      
-- or 
SELECT S.sname
FROM Sailors S, Reserves R, Boats B
WHERE S.sid = R.sid AND R.bid = B.bid AND B.color = ‘red’
INTERSECT
SELECT S2.sname
FROM Sailors S2, Boats B2, Reserves R2
WHERE S2.sid = R2.sid AND R2.bid = B2.bid AND B2.color = ‘green’
 -- This method has a bug. It actually returns the name with the green boat and the name with the red boat, but the same name may not be the same person

**(Q7)** Find the names of sailors who have reserved at least two boats.

```sql
SELECT S.sname
FROM Sailors S, Reserves R, Sailors S2, Reserves R2
WHERE S.sid = R.sid 
  and S2.sid = R2.sid 
  and S.sid = S2.sid 
  and R.bid != R2.bid
```

**(Q8)** Find the sids of sailors with age over 20 who have not reserved a red boat.

```sql
SELECT S.sname
FROM Sailors S, Reserves R, Boats B 
WHERE B.color != 'red' and B.bid = R.bid and S.sid = R.sid and S.sid > 20
```

**(Q9)** Find the names of sailors who have reserved all boats. The use of the word all (or every) is a good indication that the division operation might be applicable:

Division  returns all sids such that there is a tuple ⟨sid,bid⟩ in the first relation for each bid in the second. 

**(Q13)** Find the sailor name, boat id, and reservation date for each reservation.

```sql
SELECT S.sname, R.bid, R.day 
FROM Sailors S, Reserves R
WHERE S.sid = R.sid 
```

**(Q16)** Find the sids of sailors who have reserved a red boat.

```sql
SELECT R.sid
FROM Reverves R, Boats B
WHERE R.bid=B.bid AND B.color='red'
```

**(Q17)** Compute increments for the ratings of persons who have sailed two different boats on the same day 

```sql
SELECT S.sname, S.rating+1 AS rating
FROM Sailors S, Reserves R1, Reserves R2
WHERE S.sid = R1.sid AND S.sid = R2.sid
AND R1.day = R2.day AND R1.bid <> R2.bid
```

**(Q18)** Find the ages of sailors whose name begins and ends with B and has at least

three characters.

```sql
SELECT S.age
FROM Sailors S
WHERE S.sname LIKE' B_%B'
```

**(Q19)** Find the sids of all sailors who have reserved red boats but not green boats.

set-difference

```sql
SELECT S.sid
FROM Sailors S, Reserves R, Boats B
WHERE S.sid = R.sid AND R.bid = B.bid AND B.color = ‘red’
EXCEPT
SELECT S2.sid
FROM Sailors S2, Reserves R2, Boats B2
WHERE S2.sid = R2.sid AND R2.bid = B2.bid AND B2.color = ‘green’
```

**(Q20)** Find all sids of sailors who have a rating of 10 or have reserved boat 104.

```sql
SELECT S.sid
FROM Sailors S 
WHERE S.rating = 10
UNION
SELECT R.sid
FROM Reserves R 
WHERE R.bid = 104
```

**(Q22)** Find sailors whose rating is better than some sailor called Horatio.

```sql
SELECT S.sid
FROM Sailors S
WHERE S.rating > ANY ( SELECT S2.rating
                       FROM Sailors S2
                       WHERE S2.sname = ‘Horatio’ )
```

**(Q24)** Find the sailors with the highest rating.

```sql
SELECT S.sid
FROM Sailors S
WHERE S.rating>=ALL(SELECT S2.rating
                    FROM Sailors S2 )

```

**(Q25)** Find the average age of all sailors.

```sql
SELECT AVG(S.age)
FROM Sailors S
```

**(Q27)** Find the name and age of the oldest sailor.

Consider the following attempt to answer this query:

```sql
SELECT S.sname, MAX (S.age) 
FROM Sailors S
```

this query is illegal in SQL—if the SELECT clause uses an aggregate operation, then it must use only aggregate operations unless the query contains a GROUP BY clause!

correct answer:

```SQL
SELECT S.sname, S.age
FROM Sailors S
WHERE S.age=(SELECT MAX(S2.age)
             FROM Sailors S2 )
--because of the use of the aggregate operation, the subquery is guaranteed to return a single tuple with a single field, and SQL converts such a relation to a field value for the sake of the comparison.

-- by myself
SELECT S.sname, S.age
FROM Sailor S
where S.age >= ALL(SELECT S2.age 
                   FROM Sailors S2)
```

**(Q30)** Find the names of sailors who are older than the oldest sailor with a rating of 10.

```sql
SELECT S.sname
From Sailors S
WHERE S.age> (SELECT MAX(S2.age)
              FROM Sailor S2
              WHERE S2.rating=10)
```

**(Q31)** Find the age of the youngest sailor for each rating level.

```sql
SELECT S.rating, MIN (S.age) 
FROM Sailors S
GROUP BY S.rating
```

**(Q32)** Find the age of the youngest sailor who is eligible to vote (i.e., is at least 18 years old) for each rating level with at least two such sailors.

```SQL
SELECT S.rating, MIN(S.age) AS minage
FROM Sailors S
WHERE S.age>18
GROUP BY S.rating
HAVING COUNT(*) > 1 
```

**(Q33)** For each red boat, find the number of reservations for this boat.

```sql
SELECT B.bid, COUNT (*) AS sailorcount 
FROM Boats B, Reserves R
WHERE R.bid = B.bid AND B.color = ‘red’
GROUP BY B.bid
```

**(Q34)** Find the average age of sailors for each rating level that has at least two sailors.

```sql
SELECT S.rating, AVG(S.age) AS avgage
FROM Sailors S
GROUP BY S.rating
HAVING COUNT(*) > 1 

-- alternative
SELECT S.rating, AVG(S.age) AS avgage
FROM Sailors S
GROUP BY S.rating
HAVING 1 < ( SELECT COUNT (*)
             FROM Sailors S2
             WHERE S.rating = S2.rating )
```

**(Q37)** Find those ratings for which the average age of sailors is the minimum over all ratings.

```sql
SELECT S.rating
FROM Sailors S
WHERE AVG (S.age) = ( SELECT MIN (AVG (S2.age))
                      FROM Sailors S2 GROUP BY S2.rating )
```



