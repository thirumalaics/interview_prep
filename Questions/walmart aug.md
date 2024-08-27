Linked List = 11->11->11->21->43->43->60
node
node.next
prev = None
while node:
    print(node.val)
    prev = node.val
    while node and node.val == prev:
        node = node.next



last k elements of a linked list
deduplicate linked list


roll_num 
1 1
2 2
3 3 
5 4
7



from rn


select month()
from orders 
group by month()

SELECT
extract('MONTH' FROM order_date) as month_val, 
sum(amount) as monthly_amnt
FROM orders
group by extract('MONTH' FROM order_date)








Given an array candidates of distinct integers and a target integer target, we need to find all unique combinations of numbers from candidates that add up to target .
We can use the same number from candidates 
multiple times in a combination.
Two combinations are considered unique if the frequency of at least one of the chosen numbers is different.
 
Input: candidates = [2, 3, 6, 7], target = 7 Output: [[2, 2, 3], [7]]







def recursion(i, path,s):
	if s == target:
		ans.append(path)
		return
	if s > target or i >= n:
		return
	recursion(i, path+[candidates[i]], s+candidates[i])
      
	


ans = []
recursion()



 