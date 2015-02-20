---
layout: post
title: COUNT INVERSIONS IN JAVA
published: true
---

### Count inversions definition
Counting inversions on an array shows how far an array is from being sorted. If the array is sorted then the count is 0. If the array is sorted in reverse order then the count is the maximum.
In real life counting inversions can serve to identify how close the preferences of certain groups of users are and based on that for example create suggestions.
Counting inversions, like merge sort presented in the previous post, is a Divide and Conquer algorithm.

### Divide and Conquer definition
Divide and Conquer is an algorithm paradigm based on multi branched recursion. The algorithm works by recursively breaking down a problem into two or more sub-problems of the same type, until these become simple enough to be solved directly. The solutions to the sub-problems are then combined to give a solution to the original problem.

### Count inversions algorithm explanation
The base principle is the same as with merge sort. We split the array in multiple sub-arrays and count the number of inversions in each of the sub-arrays. But we also have to keep in mind that inversions might occur also on merging. So, the total number of inversions is composed by the number of inversions in the left sub-array plus the number of inversions in the right sub-array plus the number of inversions that occur during merging.

### Java implementation
We start by creating the divide method that splits the initial array into smaller sub-arrays.

``` java
public int[] divideAndConquer(int[] inputArray) {
  int n = inputArray.length;
  if(n == 1) {
    return inputArray;
  }
  int mid = n/2;
  int[] leftArray = new int[mid];
  int[] rightArray = new int[n - mid];
  System.arraycopy(inputArray, 0, leftArray, 0, leftArray.length);
  System.arraycopy(inputArray, leftArray.length, rightArray, 0, rightArray.length);
  divideAndConquer(leftArray);
  divideAndConquer(rightArray);
  merge(leftArray, rightArray, inputArray);
  return inputArray;
}
```

Next we move forward to merging and counting the number of inversions.

``` java
public void merge(int[] leftArray, int[] rightArray, int[] sortedArray) {
  int leftArrayLength = leftArray.length;
  int rightArrayLength = rightArray.length;
  int i = 0;
  int j = 0;
  int k = 0;
  while(i < leftArrayLength && j < rightArrayLength) {
    if(leftArray[i] < rightArray[j]) {
      sortedArray[k] = leftArray[i];
      i++;
    } else {
      sortedArray[k] = rightArray[j];
      j++;
      total = total.add(new BigDecimal(leftArray.length - i));
    }
    k++;
  }
  while(i < leftArrayLength) {
    sortedArray[k] = leftArray[i];
    i++;
    k++;
  }
  while(j < rightArrayLength) {
    sortedArray[k] = rightArray[j];
    j++;
    k++;
  }
}
```

You can find the full source code on my github repository in the Algorithms project.
<a href="https://github.com/andreivisan/Algorithms">GitHub - Algorithms</a>

### Complexity analysis
Count inversions counts in worst case in O(nlogn) time.
