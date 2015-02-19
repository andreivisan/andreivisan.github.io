---
layout: post
title: MERGE SORT IN JAVA
published: true
---

### Merge sort definition
Merge sort is a comparison based sorting algorithm. Merge sort is also considered a Divide and Conquer algorithm.

### Divide and Conquer definition
Divide and Conquer is an algorithm paradigm based on multi branched recursion. The algorithm works by recursively breaking down a problem into two or more sub-problems of the same type, until these become simple enough to be solved directly. The solutions to the sub-problems are then combined to give a solution to the original problem.

### Merge sort algorithm explanation
Let's consider an array with the following values: 1, 5, 3, 4, 2, 6. The image bellow shows how using the divide and conquer paradigm we implement the merge sort algorithm in order to sort the array.

![Merge sort](/public/images/Divide_and_Conquer.png)

The image above shows how the array is broken down into sub-arrays until the most basic array with one element is reached. After that we repeatedly merge the sub-arrays into ordered sub-arrays until we compose back the initial array, but this time sorted.

### Java implementation
We start by creating the divide method that splits the initial array into smaller sub-arrays.

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

Next we move forward to merging and sorting the sub-arrays.

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
      		}
      		k++;
    	}
        //copy the rest of the first half if there is anything left
    	while(i < leftArrayLength) {
      		sortedArray[k] = leftArray[i];
      		i++;
      		k++;
    	}
        //copy the rest of the second half if there is anything left
    	while(j < rightArrayLength) {
      		sortedArray[k] = rightArray[j];
      		j++;
      		k++;
    	}
  	}

You can find the full source code on my github repository in the Algorithms project.
<a href="https://github.com/andreivisan/Algorithms">GitHub - Algorithms</a>

### Complexity analysis
Mergesort sorts in worst case in O(nlogn) time, which makes it one of the fastest array sorting algorithms.
