---
layout: post
title:  "TW_STIME API: User Defined Simulation Time"
---

To allow for user-defined simulation time, a new TW_STIME API has been put in place.
This API is implemented through various `#define` macros, which should allow users flexibility while maintaining the performance of ROSS when not using this feature.
Any models which have already been developed using the original `double` type for `tw_stime` are unchanged.

## API Components

The TW_STIME API consists of 3 components:
1. The `tw_stime` type definition.
   In ROSS, this has historically been a `double` type.
   Now, users can define any valid C datatype.
2. 5 macros implemented with `#define`.
   These macros allow the C-preprocessor to replace various `tw_stime` manipulations with in-line statements or function calls.
   They are used throughout ROSS, but do not need to be called from model code.
3. A definition of the `MPI_TYPE_TW_STIME`, which is needed for MPI communication during parallel simulations.
   If needed, new `tw_stime` implementations must implement a derived MPI datatype.

## The Macros

The following macros are used throughout ROSS to ensure consistent interaction with `tw_stime` objects.

### Create: `TW_STIME_CRT(x)`

Where x is of type `double`.
Given a `double` value, this macro should create and return a `tw_stime` object.

### Double Conversion: `TW_STIME_DBL(x)`

Where x is of type `tw_stime`.
Given a `tw_stime` object, this macro should return a `double` value.
This is often used for printing or statistics counting.
In implementing this API, some global variables have been converted to `double` types.

### Compare: `TW_STIME_CMP(x, y)`

Where x and y are of type `tw_stime`.
Given two `tw_stime` objects, return a comparison between them.
If `x` is less than `y`, the return value is less than zero.
If `x` is equal to `y`, the return value is zero.
If `x` is greater than `y`, the return value is greater than zero.
This is the most frequently used macro throughout ROSS.

### Addition: `TW_STIME_ADD(x, y)`

Where x and y are of type `tw_stime`.
Given two `tw_stime` objects, return a new `tw_stime` object which is the addition of the two.

### Maximum Value: `TW_STIME_MAX`

This macro takes no arguments and should return a maximum `tw_stime` object.
In the included double implementation, the value used is `DBL_MAX`.

## Example

The reference implementation defines `tw_stime` as a `double` type.
Thus, it is able to use the built-in `MPI_DOUBLE` type, without the need for creating a derived type.
The comparison operation can become a one-liner using the C ternary operator.
The other API functions are trivial.

```C
typedef double tw_stime;
#define MPI_TYPE_TW_STIME   MPI_DOUBLE
#define TW_STIME_CRT(x)     (x)
#define TW_STIME_DBL(x)     (x)
#define TW_STIME_CMP(x, y)  (((x) < (y)) ? -1 : ((x) > (y)))
#define TW_STIME_ADD(x, y)  ((x) + (y))
#define TW_STIME_MAX        DBL_MAX
```
