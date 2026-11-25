Boyer-Moore-Search(T, P) 
1 n = length(T) 
2 m = length(P) 
3 matches = [] 
4 for each char c in Alphabet 
5 shift[c] = m 
6 for j = 0 to m - 2 
7 shift[P[j]] = m - 1 - j 
8 i = 0 
9 while i <= n - m 
10 j = m - 1 
11 while j >= 0 and P[j] == T[i + j] 
12 j = j - 1 
13 if j < 0 
14 matches.append(i) 
15 i = i + 1 
16 else 
17 step = shift[T[i + j]] - (m - 1 - j) 
18 i = i + max(1, step) 
19 return matches
