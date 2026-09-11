1. In an 8-bit binary number, which is the most significant bit (MSB)?

MSB = 가장 왼쪽 비트 (bit 7)

2. What is the decimal representation of each of the following unsigned binary integers?

a. 00110101
b. 10010110 
c. 11001100 
  
a. 1+4+16+32=53
b. 2+4+16+128=150
c. 4+8+64+128=204
  
3. What is the sum of each pair of binary integers?

a. 10101111 + 11011011
b. 10010111 + 11111111
c. 01110101 + 10101100

a. 110001010 b. 110010110 c. 100100001

4. Calculate binary 00001101 minus 00000111.

00000110

 
5. How many bits are used by each of the following data types?

a. word
b. doubleword
c. quadword
d. double quadword

a. 16bits b. 32 bits c. 64 bits d. 128bits 



6. What is the minimum number of binary bits needed to represent each of the following unsigned decimal integers?

a. 4095
b. 65534
c. 42319

a. 12개 b. 16개 c. 16개

7. What is the hexadecimal representation of each of the following binary numbers?

a. 0011 0101 1101 1010   
b. 1100 1110 1010 0011
c. 1111 1110 1101 1011

a. 35DA b. CEA3 c. FEDB

8. What is the binary representation of the following hexadecimal numbers?

a. 0126F9D4
b. 6ACDFA95
c. F69BDC2A

a. 0000 0001 0010 0110 1111 1001 1101 0100 
b. 0110 1010 1100 1101 1111 1010 1001 0101
c. 1111 0110 1001 1011 1101 1100 0010 1010 

9. What is the unsigned decimal representation of each of the following hexadecimal integers?

a. 3A
b. 1BF
c. 1001

a. 0011 1010(2) -> 2+8+16+32 = 58
b. 0001 1011 1111(2) -> 1+2+4+8+16+32+128+256 = 447
c. 0001 0000 0000 0001(2) -> 1 + 4096 = 4097

------------------------------------------------------------------------------------?

1.7 Review Questions and Exercises


10. What is the unsigned decimal representation of each of the following hexadecimal integers?

a. 62
b. 4B3
c. 29F

a. 0110 0010(2) -> 2+32+64 = 98
b. 0100 1011 0011(2) -> 1+2+16+32+128+1024=1203
c. 0010 1001 1111(2) -> 1+2+4+8+16+128+512=671


11. What is the 16-bit hexadecimal representation of each of the following signed decimal integers?

a. -24
b. -331

a. 11000 -> 0000 0000 0001 1000 -> 1111 1111 1110 1000 -> FFE8
b. 101001011 -> 0000 0001 0100 1011 -> 1111 1110 1011 0101 -> FEB5



12. What is the 16-bit hexadecimal representation of each of the following signed decimal integers?

a. -21
b. -45

a. 0000 0000 0001 0101(2) -> 1111 1111 1110 1011 -> FFEB
b. 0000 0000 0010 1101(2) -> 1111 1111 1101 0011 -> FFD3

13. The following 16-bit hexadecimal numbers represent signed integers. Convert each to decimal.

a. 6BF9
b. C123

a. 0110 1011 1111 1001 -> 1+8+16+32+64+128+256+512+2048+8192+16384 = 27641
b. 1100 0001 0010 0011 -> 0011 1110 1101 1101 -> 1+4+8+16+64+128+512+1024+2048+4096+8192 = -16093


14. The following 16-bit hexadecimal numbers represent signed integers. Convert each to decimal.

a. 4CD2
b. 8230

a. 0100 1100 1101 0010 -> 2+16+64+128+1024+2048+16384 = 19666
b. 1000 0010 0011 0000 -> 0111 1101 1101 0000 ->  16+64+128+256+1024+2048+4096+8192+16384=-32208


15. What is the decimal representation of each of the following signed binary numbers?

a. 10110101
b. 00101010
c. 11110000

a. 0100 1011 -> 1+2+8+64= -75
b. 2+8+32 = 42
c. 0001 0000 -> 16


16. What is the decimal representation of each of the following signed binary numbers?

a. 10000000
b. 11001100
c. 10110111

a. 0111 1111 -> -128
b. 00110100 -> 4+16+32 = -52
c. 01001001 -> 1+8+64 = -73


17. What is the 8-bit binary (two’s-complement) representation of each of the following signed decimal integers?

a. -5
b. -42
c. -16

a. 0000 0101 -> 1111 1011
b. 0010 1010 -> 1101 0110
c. 0001 0000 -> 1111 0000


18. What is the 8-bit binary (two’s-complement) representation of each of the following signed decimal integers?

a. -72
b. -98
c. -26

a. 0100 1000 -> 1011 1000
b. 0110 0010 -> 1001 1110
c. 0001 1010 -> 1110 0110


19. What is the sum of each pair of hexadecimal integers?

a. 6B4 + 3FE
b. A49 + 6BD


9, 11+15 , 14 + 4 -> 10, 27, 2
a. A B 2

b. 16, 15, 22 -> 1, 1, 0, 6


Chapter 1  •  Basic Concepts



20. What is the sum of each pair of hexadecimal integers?

a. 7C4 +  3BE
b. B69 + 7AD


a. 10, 23, 18 -> B82
b. 18, 16 , 22 -> 1316



21. What are the hexadecimal and decimal representations of the ASCII character capital B?


B = 66(10)

66 = 4 * 16 + 2
42(16)


22. What are the hexadecimal and decimal representations of the ASCII character capital G?


G = 71(10)

71 = 4 * 16 + 7
47(16)



23. Challenge: What is the largest decimal value you can represent, using a 129-bit unsigned integer?


2^129 - 1 = 680564733841876926926749214863536422911


24. Challenge: What is the largest decimal value you can represent, using a 86-bit signed integer?

2^(n-1) -1
2^85 - 1 = 38685626227668133590597631




25. Create a truth table to show all possible inputs and outputs for the boolean function described by ¬(A ∨ B).

∨ = or

A B | A ∨ B | ¬(A ∨ B)
0 0 |     0    | 1
0 1 |     1    | 0
1 0 |     1    | 0
1 1 |     1    | 0


26. Create a truth table to show all possible inputs and outputs for the boolean function described by (¬A ∧ ¬B). How would you describe the rightmost column of this table in relation to the table from question number 25? Have you heard of De Morgan’s Theorem?

∧ = and

A    B | ¬A ¬B |(¬A ∧ ¬B)
0    0  |  1   1  | 1
0    1  |  1   0  | 0
1    0  |  0   1  | 0
1    1  |  0   0  | 0

25번과 26번 문제의 마지막 열의 결과 값이 같습니다.
도모르간 법칙 : ¬(A ∨ B) = (¬A ∧ ¬B)


27. If a boolean function has four inputs, how many rows are required for its truth table?

2^4 = 16 rows

A  0  0  0  0  0  0  0  0  1  1  1  1  1  1  1  1
B  0  0  0  0  1  1  1  1  0  0  0  0  1  1  1  1
C  0  0  1  1  0  0  1  1  0  0  1  1  0  0  1  1
D  0  1  0  1  0  1  0  1  0  1  0  1  0  1  0  1


28. How many selector bits are required for a four-input multiplexer?


2^n = 4
2^n = 2^2
n = 2 selector bits
