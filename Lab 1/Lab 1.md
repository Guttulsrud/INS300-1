# INS300-1 LAB 1

## Agenda
- [Installation of R & RStudio](#installation-of-r-and-r-studio)
- [Introduction to R](#introduction-to-r)
    - [Arithmetic Operators](#arithmetic-operators)
    - [Logical operators in R](#logical-operators)
    - [Data Types in R](#data-types)
    - [Creating Variables](#creating-variables)
    - [Vectors](#vectors)


## Installation of R and R Studio
1. Download R-Language from [R-Project](https://cloud.r-project.org/) And follow download instructions
2. Download R-Studio (IDE) from [RStudio](https://www.rstudio.com/products/rstudio/download/#download).

## Introduction to R

### Arithmetic Operators
```R
5 + 2   # Addition 
5 - 2   # Subtraction
5 * 2   # Multiplication
5 / 2   # Division
5 %% 2  # Remainder 
5 ^ 2   # Exponent
5 ** 2  # Exponent
```

### Logical Operators
```R
5 > 2   # Greater than
5 < 2   # Less than
5 >= 2  # Greater than or equal to
5 <= 2  # Less than or equal to
5 == 2  # Exactly equal to
5 != 2  # Not equal to
```

### Data Types
```R
print(class(5))               # Numeric: 1, 2, 3
print(class(5L))              # Integer: 1L, 2L, 3L
print(class(5i))              # Complex: 1i, 2i, 3i
print(class(TRUE))            # Logical: TRUE, T, FALSE, F
print(class("ABC"))           # Character: "ABC", 'A'
print(class(charToRaw('A')))  # RAW: "ABC" is stored as 41 42 43
```



### Creating Variables
```R
variable = 10; variable
variable <- 15; variable
13 -> variable; variable
```

### Vectors
```R
x <- c(1, 2, 3); x
x <- 3:10; x
x <- 10:0; x
x <- c("Hello", "World"); x
x <- c("Banana" = 2, "Apple" = 3, "Orange" = 10); x
```
### Creating vectors with the `seq()` function
```R
x <- seq(1, 10, by = 2); x
x <- seq(1, 10, length.out = 3); x
x <- seq(10); x
x <- seq_len(10); x
```
### Accessing elements within a Vector.
_Vectors in R starts with an index of 1 instead of 0_
```R
x <- c(1, 3, 5, 7, 9); x[1]       # Get the first element
x <- c(1, 3, 5, 7, 9); x[3]       # Get the third element
x <- c(1, 3, 5, 7, 9); x[c(2:3)]  # Get the second and third element
x <- 1:10; x[x < 3]               # Get elements based on conditions
```
#### Changing Vector Values
```R
x <- seq(10); x[1] <- 20; x
x <- seq(10); x[c(1:5)] <- 0; x
```
