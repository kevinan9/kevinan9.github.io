---
layout: post
title: "Classical Quick Sort Algorithm"
date: 2015-07-12
comments: true
categories: ["Algorithm"]
---

Analysis
--------
Acctually [Quick Sort](https://en.wikipedia.org/wiki/Quicksort) is hard to write code directly according to that definition. We need more practical way, e.g. partition can be merged into sorting process.


Source Code
--------
```c
/*
**  @file $RCSfile: qsort.c,v $
**  quick sort
**  @author $Author: Kevin $
**  @date $Date: 2015/07/12 23:02:41 $
**  @version $Revision: 0.01 $
**  @note Editor: Vim 6, Gcc 4.4.6, tab=4
**  @note Platform: CentOS release 6.3 (Final)
*/
#include <stdio.h>
#include <stdlib.h>

void 
output(int *list, size_t len) /* only for testing */
{
    int i;
    for (i = 0; i < len; i++) {
        printf("%d ", list[i]);
    }
    printf("\n");
}

void
quick_sort(int *list, int len)
{
    #define SWAP(a, b) { int tmp = list[a]; list[a] = list[b]; list[b] = tmp; }
    
    if (1 >= len)
        return;

    int i, pivot;
    /* classical partition algorithm */
    for (pivot = i = 0; i < len - 1; i++) {
        if (list[i] > list[len-1])  /* order, >:ascend; <:descend  */
            continue;
        SWAP(i, pivot);
        pivot++;
    }
    
    SWAP(len-1, pivot);
    
    quick_sort(list, pivot);
    
    quick_sort(list + pivot, len - pivot);
}

int 
main(int argc, char ** argv) 
{
    int s[] = {6,9,19,13,3,0,7,11,15,12,12,14,19,5,7,15,4,5,13,18};
    int len = sizeof(s) / sizeof(int);
    output(s, len);
    quick_sort(s, len);
    output(s, len);

    exit(EXIT_SUCCESS);
}


```

