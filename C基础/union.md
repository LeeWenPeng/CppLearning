## 1  概念

- 一种允许用户自定义的数据类型
- union 中，所有成员共享同一个位置
- 换句话说，所有成员的存储地址都是从相同地址开始
- 因此，对于 union，在同一时刻，只允许有一个成员允许含有一个值
- 通过 union，程序员可以通过不同方式使用相同一个内存地址

## 2  语法

```C
union tag{
	type name1;
	...
};
```

## 3  sizeof

`sizeof(union)` 的结果为 union 中最宽成员（包括数组成员）所占的内存大小

## 4  定义和成员赋值

```C

// 1 定义并对联合体中第一个成员进行赋值
union un_name variable = {vaule};

// 2 定义联合体，并对联合体中指定成员进行赋值
union un_name variable = {.member = vaule};

// 3  分步
union un_name variable;
variable.member = vaule;
```

## 5  实验

通过联合体判断字节序存储为大端存储还是小端存储

```C
#include <stdio.h>

int main(){
    union{
        short x;
        char byt[sizeof(short)];
    }test;

    test.x = 0x0102;

    if(test.byt[0] == 1 && test.byt[1] == 2){
        printf("大端字节序\n");
    }else if(test.byt[0] == 2 && test.byt[1] == 1){
        printf("小端字节序\n");
    }else{
        printf("未知\n");
    }
    return 0;
}
```
