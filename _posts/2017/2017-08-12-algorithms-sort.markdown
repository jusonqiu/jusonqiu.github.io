---
layout: "post"
title: "algorithms sort"
date: "2017-08-12 11:36"
---

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [排序算法](#排序算法)
	- [常见的排序算法时间空间](#常见的排序算法时间空间)
	- [排序算法对应程序](#排序算法对应程序)
		- [BubbleSort](#bubblesort)
		- [QuickSort](#quicksort)
		- [Radix sort](#radix-sort)
		- [Shell Sort](#shell-sort)
		- [Merge Sort](#merge-sort)

<!-- /TOC -->

# 排序算法

## 常见的排序算法时间空间

|     Class     |                                      Worst-case                                       |                                       Best-case                                        |     Average     |      Worst-case <br/>space complexity       |                             wiki                             |
| ------------- | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | --------------- | ------------------------------------------- | ------------------------------------------------------------ |
| QuickSort     | $$O(n^2)$$                                                                            | $$O(nlog(n))_{simple partition}$$ <br/>  $$O(n)_{three-way partition and equal keys}$$ | $$O(nlog(n))$$  | $$O(log(n))$$<br/>$$O(n)_{native}$$         | [Quicksort](https://www.wikiwand.com/en/Quicksort)           |
| Radix sort    | $$O(wn)$$                                                                             | -                                                                                      | -               | $$O(w+N)$$                                  | [Radix sort](https://www.wikiwand.com/en/Radix_sort)         |
| ShellSort     | $$O(n^2)_{worst knoew gap sequence}$$ <br/> $$O(nlog^2n)_{best known gap sequence} $$ | $$O(nlog n)$$                                                                          | 取决于间隙序列  | $$O(n)_{total}$$ <br/> $$O(1)_{auxiliary}$$ | [ShellSort](https://www.wikiwand.com/en/Shellsort)           |
| MergeSort     | $$O(n log n)$$                                                                        | $$O(nlog n)_{typical}  $$ <br/> $$O(n)_{natural variant}$$                             | $$ O(nlog n) $$ | $$O(n)_{total}$$ <br/> $$O(n)_{auxiliary}$$ | [MergeSort](https://www.wikiwand.com/en/Merge_sort)          |
| SelectionSort | $$n^2$$                                                                               | $$O^2$$                                                                                | $$O^2$$         | $$O(1)$$                                    | [Selection Sort](https://www.wikiwand.com/en/Selection_sort) |

## 排序算法对应程序

### BubbleSort

Bubble sort, sometimes incorrectly referred to as sinking sort, is a simple
sorting algorithm that works by repeatedly stepping through the list to be
sorted, comparing each pair of adjacent items and swapping them if they are
in the wrong order. The pass through the list is repeated until no swaps are
needed, which indicates that the list is sorted. The algorithm gets its name
from the way smaller elements "bubble" to the top of the list. Because it
only uses comparisons to operate on elements, it is a comparison sort.
Although the algorithm is simple, most of the other sorting algorithms are
more efficient for large lists.

[wiki](http://en.wikipedia.org/wiki/Bubble_sort)

```c
template<typename T>
		static void	BubbleSort(T list[], int start, int end){
			int i;
			bool swapped;

			assert(start < end);

			do {
				swapped = false;
				for(i = start+1; i <= end; i++) {
					if(list[i-1] > list[i]) {
						// swap them and remember something changed
						swap(list[i-1], list[i]);
						swapped = true;
					}
				}
			} while(swapped);
		}
```

### QuickSort

 1. sort array in $$O(nlogn)$$ time.
 2. most generic fast sorting algorithm

 [wiki](http://en.wikipedia.org/wiki/Quick_sort)

```c
	/**
	 * the quick-sort partition routine
	 */
	template<typename T>
		static int partition_(T list[],int begin, int end) {
			int pivot_idx = RANDOM(begin,end);
			T pivot = list[pivot_idx];
			swap(list[begin], list[pivot_idx]);

			int i = begin + 1;
			int j = end;

			while(i <= j) {
				while((i <= end) && (list[i] <= pivot))
					i++;
				while((j >= begin) && (list[j] > pivot))
					j--;
				if(i < j)
					swap(list[i],list[j]);
			}

			swap(list[begin],list[j]);
			return j; // final pivot position
		}

	/**
	 * quick sort an array of range [begin, end]
	 */
	template<typename T>
		static void quicksort(T list[],int begin,int end) {
			if( begin < end) {
				int pivot_idx = partition_<T>(list, begin, end);
				quicksort(list, begin, pivot_idx-1);
				quicksort(list, pivot_idx+1, end);
			}
		}
```

### Radix sort

1. sort unsigned 32-bit array in $$O(n)$$ time
2. subset sorted with couting sort.

[wiki](http://en.wikipedia.org/wiki/Radix_sort)

```c
/**
	 * couting sort
	 */
	static void radix_(int byte, const unsigned N, const uint32_t *source, uint32_t *dest) {
		unsigned count[256];
		unsigned index[256];
		memset(count, 0, sizeof (count));

		unsigned i;
		for(i=0; i<N; ++i)
			count[((source[i])>>(byte*8))&0xff]++;

		index[0]=0;
		for(i=1; i<256; ++i)
			index[i] = index[i-1] + count[i-1];

		for(i=0; i<N; ++i)
			dest[index[((source[i])>>(byte*8))&0xff]++] = source[i];
	}

	/**
	 * radix sort a given unsigned 32-bit integer array of size N
	 */
	static void radix_sort(uint32_t *source, const unsigned N) {
		uint32_t * temp = new uint32_t[N];
		radix_(0, N, source, temp);
		radix_(1, N, temp, source);
		radix_(2, N, source, temp);
		radix_(3, N, temp, source);

		delete [] temp;
	}

	/**
	 * check whether the array is in order
	 */
	static void check_order(const uint32_t *data, unsigned N) {
		for(--N ; N > 0; --N, ++data)
			assert(data[0] <= data[1]);
	}
```

### Shell Sort

1. sort array in $$O(n^{3/2})$$ time.

[wiki](https://en.wikipedia.org/wiki/Shellsort)

```c
/**
	 * shell sort an array
	 */
	template<typename T>
		static void shell_sort(T *array, int len) {
			int h = 1;
			while (h < len / 3) {
				h = 3 * h + 1; // 1, 4, 13, 40, 121, ...
			}
			while (h >= 1) {
				for (int i = h; i < len; i++) {
					int cur = array[i];
					int j = i - h;
					while (j >= 0 && array[j] > cur) {
						array[j + h] = array[j];
						j = j - h;
					}
					array[j + h] = cur;
				}
				h = h / 3;
			}
		}
 ```

### Merge Sort

This is divide and conquer algorithm. This works as follows.
(1) Divide the input which we have to sort into two parts in the middle. Call it the left part
    and right part.
        Example: Say the input is  -10 32 45 -78 91 1 0 -16 then the left part will be
        -10 32 45 -78 and the right part will be  91 1 0 6.
(2) Sort Each of them separately. Note that here sort does not mean to sort it using some other
         method. We already wrote function to sort it. Use the same.
(3) Then merge the two sorted parts.

```
------------
Worst case performance		O(n log n)
Best case performance
								O(n log n) typical,
								O(n) natural variant
Average case performance		O(n log n)
Worst case space complexity	O(n) auxiliary
------------
```
Merge sort can easily be optmized for parallized computing.

[wiki](http://en.wikipedia.org/wiki/Merge_sort)

```c
	/**
	 * Merge functions merges the two sorted parts. Sorted parts will be from [left, mid] and [mid+1, right].
	 */
	template<typename T>
		static void merge_(T *array, int left, int mid, int right) {
			/*We need a Temporary array to store the new sorted part*/
			T tempArray[right-left+1];
			int pos=0,lpos = left,rpos = mid + 1;
			while(lpos <= mid && rpos <= right) {
				if(array[lpos] < array[rpos]) {
					tempArray[pos++] = array[lpos++];
				}
				else {
					tempArray[pos++] = array[rpos++];
				}
			}
			while(lpos <= mid)  tempArray[pos++] = array[lpos++];
			while(rpos <= right)tempArray[pos++] = array[rpos++];
			int iter;
			/* Copy back the sorted array to the original array */
			for(iter = 0;iter < pos; iter++) {
				array[iter+left] = tempArray[iter];
			}
			return;
		}

	/**
	 * sort an array from left->right
	 */
	template<typename T>
		static void merge_sort(T *array, int left, int right) {
			int mid = (left+right)/2;
			/* We have to sort only when left<right because when left=right it is anyhow sorted*/
			if(left<right) {
				/* Sort the left part */
				merge_sort(array,left,mid);
				/* Sort the right part */
				merge_sort(array,mid+1,right);
				/* Merge the two sorted parts */
				merge_(array,left,mid,right);
			}
		}

  ```
