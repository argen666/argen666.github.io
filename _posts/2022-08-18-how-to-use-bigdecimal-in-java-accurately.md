---
layout: post
title: How to use BigDecimal in Java accurately
author: argen666
categories: [java]
tags: [java, money, fintech, bigdecimal, double, float]
image: assets/images/posts/2022-08-18/header.jpg
toc: true
hidden: true
---

Want to know why float and double data types should never be used for floating-point calculations?
Check out this quick overview to learn more about using BigDecimal to perform precise operations. 

## Introduction

Most enterprise applications operate floating-point values.   
Fintech, e-commerce, finance, and other applications deal with floating-point operations daily and need complete precision control for all calculations.    
The developers need to know what data type of Java is appropriate for representing floating-point values, especially monetary values.   
Let's try to figure it out.

## Why double is not accurate

The first thing that comes to mind is to use float and double data types.   
However, these data types are not recommended if precise values are required.
The fact is that the float and double types are intended primarily for scientific and engineering calculations.   
They implement binary floating-point arithmetic carefully designed to quickly obtain the correct approximation for a wide range of values.    
Unfortunately, these types do not give accurate results and therefore are unsuitable for monetary calculations:   
```java
double val = 1.03 - .42;
System.out.println(val); // 0.6100000000000001 
```
To get a deeper understanding of how floating-point numbers are represented, take a look at [IEEE Standard for Floating-Point Arithmetic](https://en.wikipedia.org/wiki/IEEE_754).

## How does BigDecimal ensure accuracy

Going back to the introduction, let us see how BigDecimal represents a number and how it guarantees precision.    
Looking at the source code of BigDecimal, we can find that a BigDecimal is represented by a number by an unscaled value and a scale:
```java
public class BigDecimal extends Number implements Comparable<BigDecimal> {
  private final BigInteger intVal; // unscaled value
  private final int scale; // scale
  private final transient long intCompact;
}
```
The scale field represents the scale of BigDecimal.     
The unscaled values use a slightly more complex representation.     
When the unscaled value exceeds the threshold (the default is Long.MAX_VALUE), the intVal field is used to store the value, and the intCompact field is stored Long.MIN_VALUE, for indicating the significand information is only available from intVal.    
Otherwise, the unscaled value is compactly stored in the long-type intCompact field for the subsequent computation, and the intVal is empty.

BigDecimal also provides a scale method to return the scale of the BigDecimal:
```java
    public int scale() {
        return scale;
    }
```
The comment on that method gives a detailed description of how the scale value is used:

>Returns the scale of this BigDecimal. If zero or positive, the scale is the number of digits to the right of the decimal point. If negative, the unscaled value of the number is multiplied by ten to the power of the negation of the scale. For example, a scale of -3 means the unscaled value is multiplied by 1000.

Going back to the example above, the number 0.61, which cannot be precisely represented by binary, can be represented using BigDecimal, with unscaled value 61 and scale 2.

## How to create BigDecimal properly

Looking at BigDecimal's source code, we can figure out four significant constructors:
```java
public BigDecimal(int val)
public BigDecimal(long val) 
public BigDecimal(double val)
public BigDecimal(String val)
```
The scale of the BigDecimal created by the above four constructors is different.
`BigDecimal(int)` and `BigDecimal(long)` are more straightforward and take integer input parameters, so their scales are both 0.        
`BigDecimal(double)` and `BigDecimal(String)` scales require more detailed consideration.

### What is wrong with BigDecimal(double)

BigDecimal provides a method to create BigDecimal from double, but it is not recommended because the results can be somewhat unpredictable.
The BigDecimal created using the double value 0.61 is not exactly equal to 0.61 because 0.61 cannot be represented precisely as a finite length binary.
The actual representation is:
```java
BigDecimal val = new BigDecimal(0.61); // 0.60999999999999998667732370449812151491641998291015625
```
Thus, using BigDecimal(double), especially for calculations involving financial transactions, is extremely dangerous because it may lose precision.

### Creation using BigDecimal(String)

The `BigDecimal(String)` constructor is completely predictable.
The line `new BigDecimal("0.61")` creates a BigDecimal, which is exactly equal to 0.61:
```java
BigDecimal val = new BigDecimal("0.61"); // 0.61
```

However, it must be noted that the scales of `new BigDecimal("0.610000")` and new `BigDecimal("0.61")` are 6 and 2, respectively.
The result of the equals method comparison of these BigDecimals is false.

Accordingly, there are two following methods to create a BigDecimal that can accurately represent 0.61:
```java
BigDecimal recommend1 = new BigDecimal("0.1");
BigDecimal recommend2 = BigDecimal.valueOf(0.1);
```
The method `BigDecimal.valueOf(double)` uses two-step process and implemented by calling the `Double.toString(double)` method.
The first step is to convert the Double to String. The second step is to convert String to BigDecimal:
```java
public static BigDecimal valueOf(double val) {
        return new BigDecimal(Double.toString(val));
}
```

## Conclusion
Since computers use binary code to store and process data, many floating-point numbers cannot be represented exactly as a binary fraction of any finite length.         
Therefore, a way of representing floating-point numbers in computers was adopted through approximations, such as single-precision floating-point and double-precision floating-point formats.       
Unfortunately, these types do not give accurate results and are unsuitable for precise calculations, especially affecting financial transactions.       
Accordingly, using `BigDecimal(double)` is extremely dangerous because it may lose precision and unpredictable results.       
Thus, the generally preferred and strongly recommended way to create BigDecimal is using a `BigDecimal(String)` and `BigDecimal.valueOf(double)` methods.

## References

[1] Link to the [IEEE Standard for Floating-Point Arithmetic][ieee-link]


[ieee-link]: https://en.wikipedia.org/wiki/IEEE_754

