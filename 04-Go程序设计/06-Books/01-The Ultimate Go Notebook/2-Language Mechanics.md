[TOC]

### Language Mechanics 

#### 2.1 Built-in Types 

1. Types provide integrity and readability by asking 2 questions: 类型通过两个问题提供完整性和可读性
    - [ ] What is the amount of memory to allocate? (e.g. 1, 2, 4, 8 bytes) 分配的内存大小
    - [ ] What does that memory represent? (e.g. int, uint, bool,..) 内存的外在表现形式

2. Types can be specific to a precision such as int32 or int64: 类型有特定的精度
    - [ ] uint8 represents an unsigned integer with 1 byte of allocation  【uint8】一个字节的无符号整型
    - [ ]  int32 represents a signed integer with 4 bytes of allocation  【int32】四个字节的有符号整型

When I declare a type using a non-precision based type (unit, int) the size of the 

values constructed for these types are based on the architecture being used to build 

the program: 

\- 32 bit arch: int represents a signed int at 4 bytes of memory allocation 

\- 64 bit arch: int represents a signed int at 8 bytes of memory allocation 