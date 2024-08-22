https://leetcode.com/discuss/interview-question/3715285/Block-or-OA-or-Build-obstacles-and-check-if-new-constructions-can-take-place
```
def obstacle(queries):
    s = SortedSet()
    
    res = ''
    for i in queries:
        if i[0] == 1:
            s.add(i[-1])
        else:
            _,bound, size = i
            if bound-size in s:
                res+='0'
                print(i, res[-1])
                continue
            
            ind = s.bisect_right(bound - size -1)
            print(ind)
            if ind >= len(s) or (s[ind] < bound-size or s[ind] > bound-1):
                res+='1'
            else:
                res+='0'
            print(i, res[-1])
    return res
obstacle([[1, 2], [1, 5],[2, 3, 2], [2,0,2],[1,-1],[2, 3, 3], [2,0,2], [2, 1, 1], [2, 1, 2]])
obstacle([[2,0,2]])
```



a= [school , sckool , Kiran, Karan, Amol , Amul , Anil ]
 
Input :
School
Sckool
Amol
Anil
 
Ouput:
Sch
Sck
Amo
An
            s -2 
           c - 2
         h-1   k - 1



n length of input string -> O(n)

class Tree:
	def __init__(self, val):
		self.children = {}
		self.val = val
		self.count = 1
	def addWord(self, word):
		curr = self
		for i in word:
			if i in curr.children:
				curr.children[i].count+=1
			else:
				node = Tree(i)
				curr.children[i] = node
			curr = curr.children[i]
		

node = Tree('')
for i in a:


- data quality
	- accuracy
		- extent to which data accurately represents real-word values or events
		- data validation rules to prevent inaccurate info from entering our sys
	- completeness
		- whether a dataset contains all necessary records, without missing values or gaps
		- filling missing values, merging multiple information sources or utilizing external reference datasets
	- timeliness and currency
		- up to date data makes it relevant for analysis and decision making purposes
		- incremental updates, scheduled refreshes
	- consistency
		- the extent to which data values are coherent and compatible across different datasets or systems
		- using consistent naming conventions, formats and units of measurement
	- uniqueness
		- dups can skew analysis by over-representing specific data points or trends
		- deduplication
	- data granularity and relevance
		- striking a balance between granular low level data and high level data
		- too granular can make it complex to analyze
		- high granularity might loose some details and make the data useless for specific use cases
	- 