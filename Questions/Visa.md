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