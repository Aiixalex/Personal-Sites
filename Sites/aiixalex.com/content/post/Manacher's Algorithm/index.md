---
title: Manacher's Algorithm
date: 2020-02-12
math: true
diagram: true
authors:
- admin
---

The 5th problem of Leetcode, **Longest Palindromic Substring**, is the first problem in the set of problems about dynamic programming.

In the process of completing and looking through the community, I would like to share a really interesting algorithm named **Manacher's algorithm** as well as the basic solution by dynamic programming.

## Problem

Given a string **s**, find the longest palindromic substring in **s**. You may assume that the maximum length of **s** is 1000.

Example:

```md
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```

## Dynamic Programming

### Algorithm

To improve over the brute force solution, we first observe how we can avoid unnecessary re-computation while validating palindromes. 

Consider the case "ababa". If we already knew that "bab" is a palindrome, it is obvious that "ababa" must be a palindrome since the two left and right end letters are the same.

$$
P\left (i,j \right ) = \left (P \left (i+1, j-1 \right )\ and\ \mathbf S_{i} == \mathbf S_{j}\right )
$$

### Code

c++:

```c++
class Solution {
public:
    string longestPalindrome(string s) {
        if(s.empty()) return "";
        int n = s.length();
        bool isPalindrome[n][n];
        fill_n(&isPalindrome[0][0],n*n,false);
        int maxLength = 0;
        int startIndex = 0;
        string result;
                
        int i, j;
        
        for(j=0; j<n; j++){
            for(i=0; i<=j; i++){
                isPalindrome[i][j] = (s[i] == s[j]) && 
                    (j-i<=2 || isPalindrome[i+1][j-1] );
                
                if(isPalindrome[i][j] && j-i+1 > maxLength){
                    maxLength = j-i+1;
                    startIndex = i;
                }
            }
        }
        result = s.substr(startIndex, maxLength);
        return result;
    }
};
```

1. The fill_n() function in C++ STL is used to fill some default values in a container. The fill_n() function is used to fill values upto first n positions from a starting position. It accepts an iterator begin and the number of positions n as arguments and fills the first n position starting from the position pointed by begin with the given value.

```c++
void fill_n(iterator begin, int n, type value);
```

2. In the traversal process, variable j loops in the outer layer, while variable i loops in the inner layer. The reason is as follows.

    For example, when we want to the value of isPalindrome[2][7], we must know the value of isPalindrome[3][6] and isPalindrome[4][5], which means that we must traverse in the order of variable j.

## Manacher's Algorithm

### Algorithm

Firstly, we put aside even palindromes and focus on odd palindromes.

Every palindrome has a center and a right boundary, and a distance between center and right boundary, which was described as the 'expansion length' of a palindrome. In this algorithm, we create an array **P[i]** to record the expansion length of an i-centered palindrome, and let Integer **c** and **r** to present the index of center and right boundary.

**Eg: “abacabacabb”**

When going from left to right, when i is at index 3, the longest palindromic substring is “abacaba”, P[3] = 3.

Normally, we have to traverse from P[1] to P[string.length()-2], but we don’t need to manually go to each index and expand to check the expansion length every time. This is exactly where Manacher’s algorithm optimizes better than brute force, by using some insights on how palindromes work. Let’s see how the optimization is done.

When i = 4, the index is inside the scope of the current longest palindrome, i.e., i < r. So, instead of naively expanding at i, we want to know the minimum expansion length that is certainly possible at i, so that we can expand on that minimum P[i] and see, instead of doing from start. So, we check for mirror i’.

As long as the palindrome at index i’ does NOT expand beyond the left boundary (l) of the current longest palindrome, we can say that the minimum certainly possible expansion length at i is P[i’].

Remember that we are only talking about the minimum possible expansion length, the actual expansion length could be more and, we’ll find that out by expanding later on. In this case, P[4] = P[2] = 0. We try to expand but still, P[4] remains 0.

Now, if the palindrome at index i’ expands beyond the left boundary (l) of the current longest palindrome, we can say that the minimum certainly possible expansion length at i is r-i.

**Eg: “acacacb”**

P[3] = 2

P[2] = 2

P[4] = 5–4 = 1

You could ask but why can’t the palindrome centered at i expand after r in this case? If it did, then it would have already been covered with the current center c only. But, it didn’t. So, P[i] = r-i.

So, we can sum the two situations:

```c++
if(i>=right) {
    P[i] = 0;
}else{
    P[i] = min(P[MirrorIndex], right-i);
}
```

### Code

c++:

```c++
class Solution {
public:
    string longestPalindrome(string s) {
        string t;
        int i;
        for(i=0; i<s.length(); i++){
            t += "#";
            t += s[i];
        }
        t+="#";
        
        vector<int> P(t.length(),0);
        int center = 0;
        int right = 0;
        int maxLen = 0;
        int resultCenter = 0;
        for(i=1; i<t.length()-1; i++){
            int MirrorIndex = center - (i-center);
            if(i>=right) {
                P[i] = 0;
            }else{
                P[i] = min(P[MirrorIndex], right-i);
            }
            
            while(i-1-P[i]>=0 && i+1+P[i]<t.length() && t[i-1-P[i]] == t[i+1+P[i]]){
                P[i]++;
            }
            
            if(i+P[i]>center){
                center = i;
                right = i+P[i];
            }
            if(P[i]>maxLen){
                maxLen = P[i];
                resultCenter = i;
            }
        }
        return s.substr((resultCenter - maxLen)/2, maxLen);
    }
};
```

## References and Websites

[Manacher’s Algorithm Explained— Longest Palindromic Substring](https://medium.com/hackernoon/manachers-algorithm-explained-longest-palindromic-substring-22cb27a5e96f)

### Did you find this page helpful? Consider sharing it 🙌
