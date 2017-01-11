---
layout: post
title:  malloc stimulation in C
date:   2016-10-14 01:08:00 +0800
categories: HOW-TO
tag: c-malloc
---

* content
{:toc}



``` java
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int _DISK_CAPACITY = 100;

char disk[100];

typedef struct Xalloc_struct{
  int ptr;
  int size;
} Xalloc_struct;

void disk_init(){
//mark all disk space as free.
  int i;
  for (i = 0; i < _DISK_CAPACITY; ++i) {
    disk[i] = 0;
  }
}

Xalloc_struct* Xalloc(int size){
//allocate the appropriate space and return a pointer to the first byte of that space in disk.
// if there is not enough space, return NULL.
  int i = 0;
  while (i < _DISK_CAPACITY) {
    while (disk[i]) {
      i++;
    }
    int j = i;
    // printf("the first empty spot at %d\n", j);
    // see if we have this much space
    if (j + size >= _DISK_CAPACITY) {
          // printf("This allocation failed because there's not enough space.\n");
          return NULL;
        }

    int empty_count = 0;
    while (!disk[j] && empty_count < size) {
      empty_count++;
      j++;
    }

    if (empty_count == size) {
      // printf("haha\n");
      Xalloc_struct * x = (Xalloc_struct *)malloc(sizeof(Xalloc_struct));
      x->ptr = i; // points to the beginning of the chunk of memory taken
      x->size = size;
      int k;
      for (k = 0; k < x->size; k++) {
        disk[i + k] = 1;
      }
      // printf("allocation finished at %d\n", k);
      return x;
    }
    i = j; // if empty_count < size, meaning current slot is not big enough, check out the next one.
  }
  // printf("This allocation failed because there's not enough space.\n");
  return NULL;
}

int Xfree(Xalloc_struct* ptr){
//free the space pointed to by ptr.  if the pointer is NULL do nothing.  return 0 on success and 1 on an error.
  if (ptr == NULL) {
    return 1;
  }
  int i;
  for (i = 0; i < ptr->size; i++) {
    if (!disk[ptr->ptr + i]) {
      return 1;
    }
    disk[ptr->ptr + i] = 0;
  }
  return 0;
}

void print_disk_map(){
//print as a series of 0 and 1 the status of the disk, for example if the first three bytes are allocated and the rest are free print 1110000000000. . .

  int i;
  for (i = 0; i < _DISK_CAPACITY; i++) {
    printf("%d ", disk[i]);
    if (((i + 1) % 10) == 0) {
      printf("\n");
    }
  }
  return;
}
```
