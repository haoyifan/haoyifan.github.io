---
layout: post
title: Random note for TAing ve280
category: posts
---

## Misc about project 3

##### Ve280 TA Group

##### 7/13/2017

---

Congratulations on finishing project 3! If you have little programming experience before, it is probably the most complicated program you have ever written by now. You should already solved substantial amount of problems all the way through it. We also noticed several points worth mentioning. This is a summary to help you better prepared for the following projects, or even your career. It is not guaranteed that this summary will cover all the problems you met. If you found other problems you'd like to share, feel free to email the TA group.

---

#### Runtime Error

Several conversations that came up really really often in my office hour was the following:

> "What is segmentation fault?"
>
>  "My program crashed and it told me there's a segmentation fault. I don't know how to debug it."
>
>  "Why the program passes easy cases, but gave me segmentation fault in large cases?"
>
>  "Why the program gave me segmentation fault randomly...."

Well, if you want to efficiently debug your program, master C++, or any other low level system programming language, you need to know what is segmentation fault, or runtime error in a general sense.

There're many different runtime errors, and segmentation fault(段错误) is one of them. Generally speaking, __runtime error is the problems your program met while it's running, and segmentation fault means Your program just accessed memory that does not belong to your program__.  Segmentation is akin to memory. You can also think of __segmentation fault as memory fault__. Here are several examples that might gave your program segmentation fault.

* Access array out of boundary

```c++
int main() {
  int small_array[5];
  
  // you access small_array[5], which is out of boundary.
  // That memory does not necessarily belong to your program.
  for(int i = 0; i <= 5; ++i)
    small_array[i] = i; 
  
  return 0;
}
```

* Dereference a dangling pointer

```c++
int main() {
  
  // ptr does not point to anything, nor does it have a default value.
  // It potentially points memory that belongs to other program.
  int *ptr;
  *ptr = 0;
  
  // 99.99 percent of time you will get problem if you dereference null 
  // pointer. If you happen to get through it, it's because either your
  // roommate messed up with your operating system, or you are dreamming.
  int null_ptr = NULL;
  *null_ptr = 0;
  
  return 0;
}
```

* Try to overwrite read-only memory

```c++
int main()
{
  // if you declare c string this way, the compiler will put the c
  // string in read only memory. It's prohibited to change its content.
  char *c = "hello world";
  c[0] = 'H';
  
  // However, this is OK.
  char c2[] = "hello world";
  c2[0] = 'H';
  
  return 0;
}
```

* Pointer out of function scope

  This is perhaps the most commonly met bug in Project 3. I saw a lot of students initializing some data structures (especially those containing pointers) in one function, but still got segmentation fault afterwards. The problem is that if you initialize a pointer with a local variable inside a function, when the function returns, the variable evaporates, so does the pointer of the variable. Basically your pointer will point to something non-sense, and dereferencing it will probably gave you segmentation fault.

  In order to get around this issue, you need to have your pointers __point to something that is persistent__ through your program. For example, you could point to fields in "world" in your project 3 without worrying this problem, because world will exist to the end of the program. Another way would be dynamically malloc/new memory by yourself. Memory from malloc/new will not disapper until you explicitly call free/delete.

```c++
typedef struct s {
  string name;
  int *data_ptr;
} c_data;

void get_data_wrong(c_data &data) {
  int data = 0;
  string name = "1s";
  data.data_ptr = &data;
  data.name = name;
  
  return;
}

void get_data_correct(c_data &data) {
  string name = "1s";
  data.data_ptr = new int;
  data.name = name;
  
  return;
}

int main() {
  c_data data;
  
  // *data.data_ptr is problematic
  get_data_wrong(data);
  cout << *data.data_ptr << " " << data.name << endl; 
  
  // This is correct
  get_data_correct(data);
  cout << *data.data_ptr << " " << data.name << endl; 
  
  delete data.data_ptr; // prevent memory leak
  
  return 0;
}
```



Some other runtime errors that needs attention:

* Stack Overflow

```c++
int main() {
  // arr1 needs more memory than allowed. Those statically allocated
  // memory resides in stack, and the dynamically allocated memory 
  // is in heap. By default, heap is much much larger than stack. So
  // if you want a large chunk of memory, you can either use malloc/new,
  // or adjust stack size through compiler flags.
  int arr1[1000000000]; // Runtime error. stack overflow.
  int *arr2 = new int[1000000000]; // OK
  return 0;
}
```

* Divide by zero

```c++
int main() {
  int x = 1 / 0; // math
  return 0;
}
```



__Note. It is not guaranteed that you would have the same problem if you run the above segmentation fault code in your own computer. Even worse, the program might even crash randomly. They are undefined behavior, which is covered in the next section.__



##### How to debug segmentation fault

* Add a bunch of cout statments ended with "endl" to your program. If the first cout is printed and the second doesn't, the problem is between these two couts.
* If you tried the first method, you'd find it's not efficient to do so. I'd recommend you to learn to use debugger such as gdb to debug your program. If you don't know how to use gdb in command line, we recommend you to use CLion, as it's a light weight and useful IDE for developing C++ program, and it also has gdb with a nice user interface. Learning debugger for 2 hours will save you hundreds of sleepless nights.

---

#### Undefined Behavior

We have already talked about undefined behavior in recitation class, but it is so hard to cover it thouroughly. In fact, undefined behavior is an important feature (not a problem, although some programmers think this feature is nightmare...) in C and C++.

* What is undefined behavior?

  > C and C++ are programming languages, and there are specific commitees and organizations that maintain their language standard (syntax is part of it). __The programming language standard dictates what behaviors should the program have in certain circumstances, but there're some behaviors that are left undefined, that is called undefined behavior__. Undefined means anything could happen, it could behave as you wish, or it can also 炸了你的电脑 without your permission (fine I'm exaggerating..), and that's the essence of undefined behavior.

* Why do we need to have undefined behavior? Why not define everything in the standard?

  > Because of undefined behavior, compilers can do a lot of aggressive optimizations to your program.

* Common undefined behaviors

  * Dereference dangling/null pointer.  (could give you seg fault)
  * Access memory out of its boundary.  (could give you seg fault)
  * Use pointers to point to variables that has ended its lifetime.  (could give you seg fault)
  * Integer overflow / underflow (1 + INT_MAX could give you INT_MIN, or any number)
  * Don't have a return statement in a non-void function (the return value is garbage)
  * Buffer overflow (covered in the following section)
  * Default values of built in types (covered in the following section)
  * i = ++i; (this is an interview question, what is the value of i?)
  * There are thousands of other undefined behaviors...



__Note. Next time when you ask "why my program passes in my roommates' computer, but crashes in my own computer with the same test case", check for undefined behavior in your code. It is also the reason why segmentation fault may appear randomly in different computers__

---

#### Buffer Overflow

For the following program, what is the output?

```c++
#include <iostream>
using namespace std;

int main()
{
  int a[5];
  int y = 0;
  a[5] = 1024;
  cout << y << endl;
}

```

For some, it outputs 0. For others, it outputs 1024. Why is the case?

If you look at a[5] = 1024, it's actually an out of boundary problem we talked about in the first section. After reading the second section, you should know it is also an undefined behavior. But why y could be 1024? Is it an incident?

Actually, if you get output as 1024, it is because the memory space of variable y and a[5] overlaps in your program. So if you change a[5] as 1024, you are also changing y into 1024. This problem is called __Buffer Overflow__, which means you changed memory space that is out side of the buffer. In the above program, the buffer has size 5, but a[5] actually modifies the 6th element of the buffer, it "overflows" the buffer.

I mention this problem because in my office hour last night, a student approached me with the following code:

```c++
int main() {
  //...
  init_world();
  string x;
  for(int i = 0; i < num_of_species; ++i)
    x[i] = 1;
  simulate(world);
  //...
}
```

We found that even though the code initializes the world structure correctly, for some reason, the fields inside the world magically changed...

The problem with the above code is that, __the program doesn't allocate space for string x, so it has size of 0, but the program changed the first, second, third ... elements of the string, which does not belong to the string. It is a buffer overflow, and it changes data inside the world structure__

Buffer overflow is also a security problem in computer science. It can be used to peak into confidential data, or gain root access to some machine. This example only serves the purpose to introduce buffer overflow. If you are interested to use this hack, google it.

---

#### Use Enum correctly

Nearly all the students came to my office hour have the following code:

```c++
enum ability_t[FLY, ARCHERY, ABILITY_SIZE];

int func(const int i) {
  world.creatures[i].abilities[0] = false;
  world.creatures[i].abilities[1] = false;
}

enum direction_t[WEST, EAST, NORTH, SOUTH];

int next_point(const direction_t dir) {
	switch(dir) {
      case 0: //...
      case 1: //...
      case 2: //...
      case 3: //...
	}
}
```

What if I have the following enumeration in the code? Can you tell me what abilities does the creature have in less than 2 second? This becomes painful if the enumeration gets larger.

```c++
enum ability_t[FLY, ARCHERY, STRONG, FAST, BLOW, MAGIC, STEAL, JUMP, TALK, KILL, DEFEND, PHASE_SHIFT, REN_YI_MEN, ABILITY_SIZE];

creature.abilities[0] = true;
creature.abilities[1] = false;
creature.abilities[7] = true;
creature.abilities[11] = false;
creature.abilities[2] = true;
creature.abilities[4] = false;
creature.abilities[3] = true;
creature.abilities[12] = true;
creature.abilities[5] = false;
creature.abilities[9] = false;
creature.abilities[6] = true;
creature.abilities[10] = true;
creature.abilities[8] = true;
```

If I changed the starting number of FLY, you need to change everythign

```c++
enum ability_t[FLY = 100, ARCHERY, STRONG, FAST, BLOW, MAGIC, STEAL, JUMP, TALK, KILL, DEFEND, PHASE_SHIFT, REN_YI_MEN, ABILITY_SIZE];

creature.abilities[100] = true;
creature.abilities[101] = false;
creature.abilities[107] = true;
creature.abilities[111] = false;
creature.abilities[102] = true;
creature.abilities[104] = false;
creature.abilities[103] = true;
creature.abilities[112] = true;
creature.abilities[105] = false;
creature.abilities[109] = false;
creature.abilities[106] = true;
creature.abilities[110] = true;
creature.abilities[108] = true;
```

What about the following, is it clear?

```c++
enum ability_t[FLY, ARCHERY, STRONG, FAST, BLOW, MAGIC, STEAL, JUMP, TALK, KILL, DEFEND, PHASE_SHIFT, REN_YI_MEN, ABILITY_SIZE];

creature.abilities[FLY] = true;
creature.abilities[ARCHERY] = false;
creature.abilities[STRONG] = true;
creature.abilities[FAST] = true;
creature.abilities[BLOW] = false;
creature.abilities[MAGIC] = false;
creature.abilities[STEAL] = true;
creature.abilities[JUMP] = true;
creature.abilities[TALK] = true;
creature.abilities[KILL] = false;
creature.abilities[DEFEND] = true;
creature.abilities[PHASE_SHIFT] = false;
creature.abilities[REN_YI_MEN] = true;
```

We talk about integer as underlying representation of enumeration, but we are not teaching you to use integer as enumeration. __When there are labels belonging to the same group logically, we group them together as an enumeration, such that you don't need to use integer to encode those labels by yourselves.__ If you define the enumeration, and then use integer, you are defeating the purpose of enumeration... Enumeration prevents the presence of magic number in your program, and makes your code clean. __We will refuse to help those students who use integer as enumeration next time, because you are wasting the time of the person who reviews your code__.

---

#### g++ optimization flags

One thing we didn't tell you beforehand is that we enabled the level 2 optimization flag when we compile your program in online judge.

```g++
g++ -O2 -o p3 simulation.cpp p3.cpp
```

Notice the "-O2" flag in g++ compiler, it is the optimization flag we talk about.

* What is optimization flag?

  > The `-Ox` (where `x` is a number like `1`, `2`, or `3`) is an optimization level. The absence of `-O` or `-Ox` means no optimizations. `-O` is equivalent to `-O1`. Enabling the optimization will tell the compiler to make your program run faster, takes less space at the expense of compilation time. The larger the number, the better the performance/space will be.

* Why the program behaves differently before and after enabling -O2

  > Enabling optimization will not change the correctness of your program (except the compiler has bug, but we tend to believe in compiler than your code...). If the behavior is different before and after enabling optimization, it means your code has undefined behavior, and thus the compiler can do whatever optimization it wants.

---

#### Files from Windows

Fine I know most of you will still code in Windows...

Several students come to me, saying that the program behaves differently in his/her Windows and his/her Linux. If that's the case, and the first thing came up in your mind is undefined behavior, congratulations, you already beat many junior and senior students by reading my gibberish in the previous sections....

However, there's another possibility, if the difference only occurs between Windows and Linux OS.

If you don't want to know why, just type the following command in your directory when you transfer files from Windows to Linux.

```bash
dos2unix *
```

Here's the brief explanation.

For some historical reasons, the representation of new line (回车) is different in Windows and Linux. In windows, "\r\n" represents the new line, while in Linux, "\n" is enough. If you ever decompress, or change files in Windows, it will automatically add "\r" before every "\n" in the file. When you copy the files to Linux later, it will contain "\r" in the file, which would be handled differently by Linux OS. This would cause problems if your program has I/O with those files, because several functions will not work as you expect in this case.

---

#### Default value of variables

I don't know if this is covered in vg101, but just remember, __C and C++ does not have default value for built in types.__

If you declare a pointer, don't expect it will be initialized to NULL by default. If you declare an int, don't expect it will be initialized to 0 by default. If you declare an array and print it, it will give you surprise. If you declare a structure, it's just some memory in your computer, and it can contain anything if you don't initialize it.

Class is different because class can have constructor. The object of certain class will be initialized through constructor when you declare it. However, if the class contains built-in types such as int or pointer, they will not be automatically initialized.

The reason behind this is that C and C++ are designed to be high performance programming language. The compiler will do everything to make C/C++ program run faster. If the program does not initialize the varialbe, it saves some time :)

Also, this is also part of __undefined behavior__. The default value of built in types are undefined, and it could be anything.

---

#### Size of array

This should have also been mentioned in vg101, that __you should declare an array with number that is statically decidable.__

For example:

```c++
#define SIZE 5
int main() {
  // valid, 5 can be determined at compilation time
  int a[5]; 
  // valid, SIZE is a defined macro. The compiler can determine
  // it. It is not a variable.
  int b[SIZE]; 
  
  // invalid, because size is variable, and cannot be determined by compiler.
  int size = 5;
  int c[size]; 
}
```

Still, some of you might try to compile a program which has an array of variable size. Some compiler will compile it, but it is not supported by all the compilers, because __it does not belong to the standard of the language__. So next time if you found that your program cannot compile in Online Judge, check for this problem.

---

#### INT_MIN

Well, this is actually not a problem in project 3, but I found it interesting to mention because it appears in the exam...

In the exam, a student wrote -2147483648 in the exam paper, and argued that it is equal to INT_MIN.

__It's not universally true that INT_MIN is equal to -2147483648. The value of INT_MIN is determined by many things__. It is a macro defined by standard library, but standard library can differ a lot with different computer hardwares, operating systems, or even compilers!

INT_MIN is equal to -2147483648 in your computer, because you happen to use the most popular computer hardware (chip from Intel), OS(32 or 64 bit Linux) and compilers (5.4.0 g++)...

If you look at the standard of C++ or C, it only requires INT_MIN to be in certain range, not a fixed number.

---

#### What is return code of main

One thing you might ask in ve280, or vg101, is that __what's the meaning of the return value of the main function__? Is there any function that will use it?

__The answer is yes, and there's a way to check what main returns in Linux terminal__

The return value of main function indicates the exit status of the program. Generally speaking, __normal exit is represented as return value 0. Any non-zero values indicate problems in your program. There's no standard mapping from the return values to specific errors. Also, "void main()" is not standard main function signature. You should define main function as "int main(int argc, char *argv[])" in order to conform to the standard.__ 

The reason we mentioned this point is because the program of some students are mistakenly categorized as RTE(runtime error), while the true reason is WA(wrong answer). This is the pseudocode how OJ judge your code:

```c++
return_code, student_solution = student_solution();
if (return_code == 0)
    if(student_solution == correct_solution)
        ACCEPT
    else
        WRONG_ANSWER
else
    if(student_solution == correct_solution)
        ACCEPT
    else
        RUNTIME_ERROR
```

If your program exits with non-zero status, and the answer differs from the correct solution, it will categorize your program as runtime error rather than wrong answer.

This is a legacy bug from the designer of the Online Judge. We will fix this as soon as possible, but for now, just always return 0 in main function of your project code regardless of the reason. __In the case of project 4, your test suite program should return value according to the test result, but for the main function of your code, just return 0.__



If you are interested to check the return value of your program, you can run the following commands in terminal:

```bash
$ ./p3 species world-test/world1 5 v    # run your program normally
.
.
.
$ echo $?  	# $? is an environment variable representing the return code of last command
  0
```

