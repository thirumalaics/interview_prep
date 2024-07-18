- date partition on a timestamp col


Gender     c
M    		75
F    		64
Male    	10
Female    	12



Gender    Count(*)
M    		76
F    		85



SELECT CASE WHEN GENDER IN ('M','Male') THEN 'F' ELSE 'M' END Gender, sum(c) as count From x group by 1;


Emp

Emp    Salary    Month
1    1000    January
2    1500    January
3    1250    January
1    440    February
3    2980    February
2    1352    February
1    1250    March
4    440    March
2    2980    April
1    1352    April
3    950    April


Average salary that the company paid to the employees per month, who has received the minimum salary For that month


SELECT Emp.Month, avg_sal, Emp as min_wage_emp 
From Emp join
(
SELECT Month, AVG(Salary) as avg_sal, MIN(Salary) as m From Emp group by Month
) tmp 
on m = Emp.Salary and tmp.Month = Emp.Month





```
l = """one
two
three
four
five
six
seven
eight
nine
ten""".split('\n')
# with open('path', 'r') as f:
#     l = f.readlines()
c = 1
for line_num in range(1,len(l)+1):
    if line_num % 3 == 0:
        print(f'write to file(open in append mode): {l[line_num-1]}\n')
        c+=1
        l[line_num-1] = ''
    
    
print('\n'.join(l))
```
```
input1 = 'aaabbbbcccaaddd'
# output = 'a3b4c3a2d3'

ch = input1[0]
c = 0
output = ''
for i in input1:
    if i == ch:
        c+=1
    else:
        output+=ch+str(c)
        ch = i
        c = 1
else:
    output+=ch+str(c)
print(output)
```


SELECT * From tboy.partitioned_Food_events

where DATE(lastUpdateTs) = (CASE WHEN EXTRACT(DAYOFWEEK From CURRENT_DATE()) = 1 THEN CURRENT_DATE -2 WHEN EXTRACT(DAYOFWEEK From CURRENT_DATE())= 7 THEN CURRENT_DATE() - 1 ELSE CURRENT_DATE() END)


fF

Car no int, Dealer string, Price float
Composer : This job runs between Monday to Friday
Per day average volume : 1 Million records
This has 10 years of historical data
Ask: Create view to Just show me only one day of data from this table
*latest
cre_ts timestamp



CREATE TABLE dataset.vehicle (
Car_Num INTEGER,
Dealer STRING,
Price FLOAT,
Creation_Date DATE
)
PARTITION BY CREATION_DATE;


CREATE OR REPLACE VIEW dataset.vehicle as 
SELECT * From dataset.vehicle where Creation_date = (CASE WHEN DAYOFWEEK(CURRENT_DATE()) = 1 THEN CURRENT_DATE -2 WHEN DAYOFWEEK(CURRENT_DATE())= 7 THEN CURRENT_DATE() - 1 ELSE CURRENT_DATE() END)