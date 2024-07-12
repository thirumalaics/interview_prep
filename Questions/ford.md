


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