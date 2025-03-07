---
title : First Post
date : 2024-01-01 23:59:00
categories : [Statistical Computing]
---

## Q1. Consider creating a game map of the classic game called `Minesweeper' when complete information of mine locations is provided. Suppose we have an n \* m binary matrix which shows where mines are located (1 = mine, 0 = nothing). A game map displays the number of mines in the surrounding 8 pixels at each pixel.   
Using the following two approaches, write your R functions that produce a game map when an n \* m binary matrix for the mine location is given. 


### i) Scan all pixels from (1, 1) to (n, m) using for loops, and check how many mines are in the surrounding 8 pixels at every location. 

Before creating the for-oop code, a sample matrix is made (‘sample\_mat’) so as to check whether the code works without no error. 

```
set.seed(2011142127)

a <- sample(c(0,1), size = 35, replace = TRUE, prob = c(0.7,0.3))

sample_mat <- matrix(a, ncol = 7, byrow = TRUE)
sample_mat
     [,1] [,2] [,3] [,4] [,5] [,6] [,7]
[1,]    0    0    1    0    0    0    0
[2,]    1    0    0    0    0    0    0
[3,]    1    1    0    0    1    1    0
[4,]    0    0    1    1    0    0    0
[5,]    0    0    1    0    0    0    0
```

The purpose of this code is to count the number of the mines surrounding a pixel. The pixel of the input matrix is 1 when there exists a mine, and 0 otherwise. So, we can obtain the number of mines around the pixel through extracting a 3\*3 matrix whose center is the very pixel and summing all of the numbers of the 3\*3 matrix apart from the center pixel. 

  
However, this method returns an error without manipulating the input matrix, because the pixels in the edges can’t count the number of mines outside the matrix. So, 0 padding should be performed in all of the edges before counting the mines. Since 0 implies there is no mine in this area, the zero padding makes sense. 

Let us see how the function ‘minesweeper\_forloop’ work. The number of rows and columns are defined as ‘r’ and ‘c’ respectively, and a matrix ‘mat\_pad’, which has 2 more columns and rows than the input matrix ‘mat’, is defined by having all of the elements be 0. The reason for creating ‘mat\_pad’ is to add 0’s on every edge of the ‘mat’ matrix. Then, define another matrix ‘mat\_result’ whose size is identical to the original matrix ‘mat’. 

```
  r <- nrow(mat)
  c <- ncol(mat)
  mat_pad <- matrix(0, nrow = r+2, ncol = c+2)
  mat_result <- matrix(0, nrow = r, ncol = c)
```

Using a nested for loop, insert each element of ‘mat’ into the ‘mat\_pad’ starting from mat\_pad\[2,2\] to mat\_pad\[r+1, c+1\]. Then, the edges of mat\_pad is still 0. 

```
  for(i in 1:r){
    for(j in 1:c){
      mat_pad[i+1, j+1] <- mat[i,j]
    }
  }
```

The next step is divided into 2 steps. For one thing, number 99 is put into the pixel of ‘mat\_result’ where a mine exists in the corresponding pixel of ‘mat’. Second, if the pixel of ‘mat’ is empty (the number of it is 0), add up all of the surrounding numbers indicating how many numbers of mine there are and put that number to the corresponding pixel of ‘mat\_result’. These processes are written through if-else statement in the second nested for-loop. 

There is a nested for-loop (3rd) in the else-tatement, working as counting the surrounding 8 pixels. Inside the loop, another if-else statement exists so as to avoid the counting of the current pixel. In this problem, the value of the empty space is 0 so simply adding up all of the 9 elements of the 3\*3 matrix formed by the 3rd nested for-loop has no problem. However, there should be a case where the number of the vacant pixel is assigned a non-zero value, such as NAN or NULL. Thus, the if-tatement containing ‘next’ command is added in order to handle such cases. After counting all of the surrounding digits, the total value saved in the ‘temp’ variable is transferred to the corresponding pixel of ‘mat\_result’. At last, the final ‘mat\_result’ matrix is returned. 

```
  for(i in 1:r){
    for(j in 1:c){
      
      # put 99 to the mine pixel
      if(mat_pad[i+1, j+1] == 1){
        mat_result[i,j] <- 99
      }
      
      # add all of the surrounding numbers 
      else{
        temp <- 0
        
        for(k in 1:3){ # to sum up the enclosing pixels: 3*3
          for(l in 1:3){
            if(k == 2 & l == 2){
              next # not counting the current space
            }
            else{
              temp <- temp + mat_pad[i+k-1,j+l-1]
            }
          }
        }
        # indicates how many mines are in the surroundings
        mat_result[i,j] <- temp
        
      }
    }
  }
```

The following code shows how the function works with the sample matrix. 

```
> sample_mat
     [,1] [,2] [,3] [,4] [,5] [,6] [,7]
[1,]    0    0    1    0    0    0    0
[2,]    1    0    0    0    0    0    0
[3,]    1    1    0    0    1    1    0
[4,]    0    0    1    1    0    0    0
[5,]    0    0    1    0    0    0    0

> minesweeper_forloop(sample_mat)
     [,1] [,2] [,3] [,4] [,5] [,6] [,7]
[1,]    1    2   99    1    0    0    0
[2,]   99    4    2    2    2    2    1
[3,]   99   99    3    3   99   99    1
[4,]    2    4   99   99    3    2    1
[5,]    0    2   99    3    1    0    0
```

### ii) Consider summing up all the matrices that have shifted the original binary matrix by one pixel in all 8 directions. 

| (1,1) | (1,2) | (1,3) |
| --- | --- | --- |
| (2,1) | ★ | (2,3) |
| (3,1) | (3,2) | (3,3) |

Take the left picture as an example. When there is a mine (expressed as a black star) at (2,2) in this example, the number of all the pixels surrounding the star should rise to 1. In other words, the number of (3,3) pixel should increase by 1 because of the mine in (2,2), and the number of (2,1) pixel should increase by 1 as well for the same reason, and so on. Likewise, one mine affects all of the surrounding 8 spaces. So, the result of the originally empty pixel is same as that of a matrix made by adding up all of the 8 matrices which have moved to each 8 directions. Then, insert the number 99 to the mine pixel. At last, return the matrix whose edges are eliminated (to match the size of the input matrix). 

The number of rows and columns of input matrix is designated as ‘r’ and ‘c’ like the minesweeper\_forloop function. As mentioned in the question, this code does not make use of any for-oop; rather, it will use ‘rbind’ and ‘cbind’ to move the input matrix and pad it with zeros. 

The following conditions will be assumed in the implementation of the code; when a matrix is received in the function, it is padded with 0’s along the edges. Moving the matrix to right is same as adding a row of 0 on the edges of top and bottom side and then adding 2 successive columns of 0’s on the left side. Moving the matrix to upper right means adding 2 consecutive columns of 0’s on the left side and then add 2 consecutive rows of 0’s on the bottom side. This methodology is applied to the other 6 directions. Then, define a matrix which is the summation of the 8 matrices. Then, put the number 99 to the pixels corresponding to the mine pixel. At last, return the matrix, omitting the 4 edges. 

```
minesweeper_sumup <- function(mat){
  
  r <- nrow(mat)
  c <- ncol(mat)
  
  # indicate 8 directions respectively
  mat_right <- rbind(rbind(0, cbind(matrix(0, ncol = 2, nrow = r), mat)), 0)
  mat_upright <- rbind(cbind(matrix(0, ncol = 2, nrow = r), mat),
                       matrix(0, ncol = c+2, nrow = 2))
  mat_up <- cbind(cbind(0, rbind(mat, matrix(0, ncol = c, nrow = 2))), 0)
  mat_upleft <- rbind(cbind(mat, matrix(0, ncol = 2, nrow = r)),
                      matrix(0, ncol = c+2, nrow = 2))
  mat_left <- rbind(rbind(0, cbind(mat, matrix(0, ncol = 2, nrow = r))),0)
  mat_downleft <- rbind(matrix(0, ncol = c + 2, nrow = 2),
                        cbind(mat, matrix(0, ncol = 2, nrow = r)))
  mat_down <- cbind(cbind(0, rbind(matrix(0, ncol = c, nrow = 2), mat)), 0)
  mat_downright <- rbind(matrix(0, ncol = c+2, nrow = 2),
                         cbind(matrix(0, ncol = 2, nrow = r), mat))
  
  # add all of the 8 direction matrices
  mat_sum <- mat_right + mat_upright + mat_up + mat_upleft + mat_left + mat_downleft + mat_down + mat_downright
  
  # simply pad 0's to the original matrix's edge
  mat_pad <- rbind(rbind(0, cbind(cbind(0, mat), 0)), 0)
  # to find where the mines exist easily
  mat_sum[mat_pad == 1] <- 99
  
  mat_sum <- mat_sum[-c(1,r+2), -c(1,c+2)]
  # clear out the redundant edges of mat_sum matrix
  return(mat_sum)
  
}
```

The following code shows how the function works with the sample matrix. 

```
> sample_mat
     [,1] [,2] [,3] [,4] [,5] [,6] [,7]
[1,]    0    0    1    0    0    0    0
[2,]    1    0    0    0    0    0    0
[3,]    1    1    0    0    1    1    0
[4,]    0    0    1    1    0    0    0
[5,]    0    0    1    0    0    0    0

> minesweeper_sumup(sample_mat)
     [,1] [,2] [,3] [,4] [,5] [,6] [,7]
[1,]    1    2   99    1    0    0    0
[2,]   99    4    2    2    2    2    1
[3,]   99   99    3    3   99   99    1
[4,]    2    4   99   99    3    2    1
[5,]    0    2   99    3    1    0    0
```
