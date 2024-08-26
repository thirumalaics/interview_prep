emp_id, dept, salary
101, Sales, 1000
107, HR, 980
113, HR, 1100
141, Finance, 800
109, Sales, 950



SELECT e1.emp_id, e1.dept,e1.salary, (tmp.max_sal - e1.salary) as salary_diff
from employees e1 join
(SELECT dept, max(salary) max_sal
FROM Employees e
group BY dept) tmp ON
e1.dept = tmp.dept
- can we restrict dataproc resources for different job requests
- masking data pii
- spark resource allocation question 16cores, 64 gigb
- 