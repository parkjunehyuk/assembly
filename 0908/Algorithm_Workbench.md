1.7.2 Algorithm Workbench



Use any high-level programming language you wish for the following programming exercises. Do not call built-in library functions that accomplish these tasks automatically. (Examples are sprintf and sscanf from the Standard C library.)


1. Write a function that receives a string containing a 16-bit binary integer. The function must return the string’s integer value.

def binary_to_int(binary):
    result = 0

    for digit in binary: # 1 0 1 1
        result = result * 2 # 0 2 4 10

        if digit == '1':
            result = result + 1 # 1 ' 5 10

    return result


print(binary_to_int("1011"))



2. Write a function that receives a string containing a 32-bit hexadecimal integer. The 
function must return the string’s integer value.

def hex_to_int(hex_string):
    result = 0

    for digit in hex_string:
        if '0' <= digit <= '9':
            value = ord(digit) - ord('0')
        elif 'A' <= digit <= 'F':
            value = ord(digit) - ord('A') + 10
        elif 'a' <= digit <= 'f':
            value = ord(digit) - ord('a') + 10

        result = result * 16 + value

    return result


print(hex_to_int("1A"))



3. Write a function that receives an integer. The function must return a string containing the binary representation of the integer.

def int_to_binary(number):
    if number == 0:
        return "0"

    result = ""

    while number > 0:
        remainder = number % 2
        result = str(remainder) + result
        number = number // 2

    return result


print(int_to_binary(13))



4. Write a function that receives an integer. The function must return a string containing the hexadecimal representation of the integer.

def int_to_hex(number):
    if number == 0:
        return "0"

    result = ""
    digits = "0123456789ABCDEF"

    while number > 0:
        remainder = number % 16
        result = digits[remainder] + result
        number = number // 16

    return result


print(int_to_hex(26))



5. Write a function that adds two digit strings in base b, where 2 ≤ b ≤ 10 . Each string may contain as many as 1,000 digits. Return the sum in a string that uses the same number base.

def add_strings(a, b, base):
    result = ""
    carry = 0

    i = len(a) - 1
    j = len(b) - 1

    while i >= 0 or j >= 0 or carry > 0:

        x = 0
        y = 0

        if i >= 0:
            x = int(a[i])
            i = i - 1

        if j >= 0:
            y = int(b[j])
            j = j - 1

        total = x + y + carry

        digit = total % base
        carry = total // base

        result = str(digit) + result

    return result


print(add_strings("1011", "110", 2))



6. Write a function that adds two hexadecimal strings, each as long as 1,000 digits. Return a hexadecimal string that represents the sum of the inputs.

def hex_value(digit):
    if '0' <= digit <= '9':
        return ord(digit) - ord('0')
    elif 'A' <= digit <= 'F':
        return ord(digit) - ord('A') + 10
    else:
        return ord(digit) - ord('a') + 10


def hex_digit(value):
    if value < 10:
        return str(value)

    return chr(ord('A') + value - 10)


def add_hex(a, b):
    result = ""
    carry = 0

    i = len(a) - 1
    j = len(b) - 1

    while i >= 0 or j >= 0 or carry > 0:

        x = 0
        y = 0

        if i >= 0:
            x = hex_value(a[i])
            i = i - 1

        if j >= 0:
            y = hex_value(b[j])
            j = j - 1

        total = x + y + carry

        digit = total % 16
        carry = total // 16

        result = hex_digit(digit) + result

    return result


print(add_hex("1A", "2F"))



7. Write a function that multiplies a single hexadecimal digit by a hexadecimal digit string as long as 1,000 digits. Return a hexadecimal string that represents the product.

def multiply_hex_digit(digit, number):
    x = hex_value(digit)

    result = ""
    carry = 0

    i = len(number) - 1

    while i >= 0:

        y = hex_value(number[i])

        total = x * y + carry

        value = total % 16
        carry = total // 16

        result = hex_digit(value) + result

        i = i - 1

    if carry > 0:
        result = hex_digit(carry) + result

    return result


print(multiply_hex_digit("A", "1F"))



1.7 Review Questions and Exercises



8. Write a Java program that contains the calculation shown below. Then, use the javap –c command to disassemble your code. Add comments to each line that provide your best guess as to its purpose.

int Y;
int X = (Y + 4) * 3;


Y = 5
X = (Y + 4) * 3

print(X)


9. Devise a way of subtracting unsigned binary integers. Test your technique by subtracting binary 00000101 from binary 10001000, producing 10000011. Test your technique with at least two other sets of integers, in which a smaller value is always subtracted from a larger one.

def binary_subtract(A, B):
    n = len(A)

    # B의 1의 보수
    complement = ""

    for bit in B:
        if bit == '0':
            complement += '1'
        else:
            complement += '0'

    # 2의 보수 = 1의 보수 + 1
    value = int(complement, 2) + 1

    complement = format(value, '0' + str(n) + 'b')

    # A + B의 2의 보수
    result = int(A, 2) + int(complement, 2)

    # 범위를 넘어간 자리 버리기
    result = result % (2 ** n)

    return format(result, '0' + str(n) + 'b')


print(binary_subtract("10001000", "00000101"))
