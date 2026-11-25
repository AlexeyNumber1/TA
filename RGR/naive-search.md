Naive-Search(T, P)
1  n = length(T)
2  m = length(P)
3  matches = []
4  for i = 0 to n - m
5      j = 0
6      while j < m and T[i + j] == P[j]
7          j = j + 1
8      if j == m
9          matches.append(i)
10 return matches
