---
layout: post
title: "Find Median Value"
date: 2015-07-11
comments: true
categories: ["Algorithm"]
---

* Coding Interview Question 02<br>
Write a C/C++ program that calls rand() 1,000,000 times and finds the median value

Analysis
--------
First we need define 'Median' clearly:
[Median](https://en.wikipedia.org/wiki/Median)

...
If n is odd then Median (M) = value of ((n + 1)/2)th item term.

If n is even then Median (M) = value of [((n)/2)th item term + ((n)/2 + 1)th item term ]/2
...

Source Code
--------
```c
/*
**  @file $RCSfile: q8.c,v $
**  finding median value of 1M random integers
**  @author $Author: Kevin $
**  @date $Date: 2015/07/01 08:02:41 $
**  @version $Revision: 0.01 $
**  @note Editor: Vim 6, Gcc 4.4.6, tab=4
**  @note Platform: CentOS release 6.3 (Final)
*/
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <assert.h>

#define NUMS 1000000

static int
average(int a, int b) {
    return (a >> 1) + (b >> 1) + (a & b & 0x1);
}

static int 
compare(const void *p1, const void *p2) 
{
    int a = *(const int *)p1;
    int b = *(const int *)p2;
    if (a == b)
        return 0;
    else
        return (a < b)? -1 : 1;
}

void 
output(int *list, size_t len) /* only for testing */
{
    int i;
    for (i = 0; i < len; i++) {
        printf("%d ", list[i]);
    }
    printf("\n");
}

int 
quick_select(int *list, int len, int n)
{
    #define SWAP(a, b) { int tmp = list[a]; list[a] = list[b]; list[b] = tmp; }
    
    int i, pivot;
    /* classical partition algorithm */
    for (pivot = i = 0; i < len - 1; i++) {
        if (list[i] > list[len-1]) 
            continue;
        SWAP(i, pivot);
        pivot++;
    }
    
    SWAP(len-1, pivot);
    
    if (n == pivot)
        return list[pivot];
    else if (pivot > n)
        return quick_select(list, pivot, n);
    else
        return quick_select(list + pivot, len - pivot, n - pivot);
}

void
unittest () 
{
    int s[] = {6,9,19,13,3,0,7,11,15,12,12,14,19,5,7,15,4,5,13,18};
    int t[] = {6,9,19,13,3,0,7,11,15,12,12,14,19,5,7,15,4,5,13,18};
    int t2[] = {6,9,19,13,3,0,7,11,15,12,12,14,19,5,7,15,4,5,13,18};
    int len = sizeof(s) / sizeof(int);
    qsort(s, len, sizeof(int), compare);
    // output(s, len);

    int median2 = quick_select(t, len, len/2);
    int median1 = quick_select(t2, len, len/2-1);
    int median = average(median1, median2);

    // printf("len:%d\n %d=(%d+%d)/2\n", len, average(s[len/2], s[len/2-1]), s[len/2], s[len/2-1]);

    assert(average(s[len/2], s[len/2-1]) == median);

    printf("<unittesting passed!>\n");
}

int 
main(int argc, char ** argv) 
{
    #define SWAP(a, b) { int tmp = list[a]; list[a] = list[b]; list[b] = tmp; }
    #define TIME_ELAPSE(start, end) 1000000 * ( end.tv_sec - start.tv_sec ) + end.tv_usec - start.tv_usec
    unittest();

    srand(time(NULL));
    struct timeval start, end;

    /* initializing */
    int s[NUMS];
    int * t  = (int *)calloc(NUMS, sizeof(int));
    int * t2 = (int *)calloc(NUMS, sizeof(int));
    assert(t!=NULL && t2!=NULL);

    int i;
    for (i = 0; i < NUMS; i++) {
        s[i] = rand();
        t[i] = s[i]; /* memcpy is ok too */
        t2[i] = s[i];
    }

    printf("TOTAL NUMBERS: %d\n", NUMS);

    /* quick sort : easiest way to write and the result can be as benchmark */
    gettimeofday(&start, NULL);
    qsort(s, NUMS, sizeof(int), compare); /* use glibc's qsort directly */
    int median_base = average(s[NUMS/2], s[NUMS/2-1]);
    gettimeofday(&end, NULL);
    
    printf("\n<quick   sort> MEDIAN: %d=(%d+%d)/2, TIME: %ld us\n", median_base, s[NUMS/2-1], s[NUMS/2], TIME_ELAPSE(start, end));

    /* quick_select: optimizing based on quick sort algorithm */
    gettimeofday(&start, NULL);
    int median2 = quick_select(t, NUMS, NUMS/2);
    int median1 = quick_select(t2, NUMS, NUMS/2-1); /* 1000000 is even number, so we need 2 median value */
    int median = average(median1, median2);
    gettimeofday(&end, NULL);
    
    printf("\n<quick select> MEDIAN: %d=(%d+%d)/2, TIME: %ld us\n\n", median, median1, median2, TIME_ELAPSE(start, end));
    free(t);
    free(t2);

    exit(EXIT_SUCCESS);
}

```

