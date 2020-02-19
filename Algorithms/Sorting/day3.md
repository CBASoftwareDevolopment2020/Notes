# [Divide-and-conquer Sorting](https://datsoftlyngby.github.io/soft2020spring/ALG/week-08/#3-divide-and-conquer-sorting)

19-02-2020

## Merge sort

-   Divide and break
    -   Break the problem in to smalle sub-problem recursively
    -   Sub-problem should represent a part of the original problem
    -   Keep on dividing until no more division is possible
-   Conquer/Solve
    -   Smalles sub-problem are solved
    -   Solutions of all the sub-problems are merged
-   Merge/Combine
    -   Recursively combines small solutions to the big solutions

## Quick sort

Quick sort doesn’t use auxiliary space

1.  Choose a pivot p, e.g: first element of array
2.  Bring all element less than the pivot to one end of the array and all elements greater to the other.
3.  Place the pivot in between.
4.  Sort items left of the pivot
5.  Sort items right of the pivot

## Bucket sort

Time complexity: O(n+bB(n/b))
Space complexity: O(n+bB(n))
Bucket: Linked List, Array List

## Counting sort

Find max: O(n)  
Create array of size max +1 fill with zeros: O(m)  
Count occurences of each entry: O(n)
Accumulate: O(m)  
Move each item in place: O(n)  
Time complexity: O(n+m)

## Radix sort

Sorting integers  
|0|1|2|3|
|-|-|-|-|
|32<span style="color:red">9<span>|7<span style="color:red">2<span><span style="color:green">0<span>|<span style="color:red">7<span><span style="color:green">20<span>|<span style="color:green">329<span>|
|45<span style="color:red">7<span>|3<span style="color:red">5<span><span style="color:green">5<span>|<span style="color:red">3<span><span style="color:green">29<span>|<span style="color:green">555<span>|
|65<span style="color:red">7<span>|4<span style="color:red">3<span><span style="color:green">6<span>|<span style="color:red">4<span><span style="color:green">36<span>|<span style="color:green">436<span>|
|83<span style="color:red">9<span>|4<span style="color:red">5<span><span style="color:green">7<span>|<span style="color:red">8<span><span style="color:green">39<span>|<span style="color:green">457<span>|
|43<span style="color:red">6<span>|6<span style="color:red">5<span><span style="color:green">7<span>|<span style="color:red">3<span><span style="color:green">55<span>|<span style="color:green">657<span>|
|72<span style="color:red">0<span>|3<span style="color:red">2<span><span style="color:green">9<span>|<span style="color:red">4<span><span style="color:green">57<span>|<span style="color:green">720<span>|
|35<span style="color:red">5<span>|8<span style="color:red">3<span><span style="color:green">9<span>|<span style="color:red">6<span><span style="color:green">57<span>|<span style="color:green">839<span>|

Time complexity: O(n log(m))  
Space complexity: O(n)

## Alphabet

Finite set of characters

-   English letters {A,B,...,Y,Z}
-   Decimal digits {1,2,...,8,9}
-   Binary digits {0,1}
-   Colors on a trafic light

## Hashtable

Insertion: O(1)  
Search: O(1)

## Tries

Insertion: O(1)  
Search: O(1)  
Sort n elements: O(n)  
Range of m elements: O(m)

Like hashtables  
Keys must be alphabetic
