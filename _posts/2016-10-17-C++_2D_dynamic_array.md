---
title: "C++ 2D dynamic array"
layout: single
comments: ture 
tags: C++
categories: programming 
---

### (1) Allocate memory in stack for static 2D array (constant dimensions)  

{% highlight c++ %}
const int row=5;
const int col=3;
double matrix[row][col];
{% endhighlight %}


### (2) Allocate memory in heap for partially dynamic 2D array (if the column dimension is constant)  

```c++
int row=5;
double (*matrix)[COL] = new double[row][COL];
```


### (3) Allocate memory for fully dynamic 2D array (continuous memory) 

```c++
int i,j, row=5, col=3;
double **matrix = new double* [row];
matrix[0] = new double [row*col];
for (i=1; i<row; ++i)
    matrix[i] = matrix[i-1] + col;
for (i=0; i<row; i++) 
    for (j=0; j<col; j++) {
        matrix[i][j] = 0; //do something
    }
delete [] matrix[0];
delete [] matrix;
```


### (4) Allocate memory for fully dynamic 2D array (uncontinuous memory)  

```c++
int i,j, row=5, col=3;
double **matrix = new double* [row];
for(int i=0;i<row;i++)
    matrix[i]=new double[col];
for (i=0; i<row; i++) 
    for (j=0; j<col; j++) 
        matrix[i][j] = 0;

for(i=0;i<row;i++)
    delete [] matrix[i]; 
delete [] matrix; 
```

### (5) Use 1D array as 2D array

```c++
double *array = new double [row*col]; 
for (i=0; i<row; i++) 
    for (j=0; j<col; j++) matrix[i*col+j] = 0;
delete [] array;
```


### (6) Use C++ STL vector container, dimension can be changed during runtime.

```c++
vector<vector<double>> matrix(row,vector<double>(col));
matrix[i][j] = 6;
```

If you want to initialize it then use :

```c++
vector<vector<double>> matrix(row,vector<double>(col,0));
```


For dynamically allocated array in class, see http://www.cs.fsu.edu/~jestes/cop3330/notes/dma.html


