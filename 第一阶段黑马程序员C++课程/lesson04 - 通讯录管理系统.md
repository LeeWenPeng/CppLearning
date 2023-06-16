**lesson 72 ~ lesson 83** 

# 01 - 系统需求

使用c++实现通讯录管理系统

需要实现的功能如下：

+ 添加联系人：向通讯录中添加新人，信息包括（姓名、性别、年龄、联系电话、家庭住址）最多记录1000人

+ 显示联系人：显示通讯录中所有联系人信息
+ 删除联系人：按照姓名进行删除指定联系人
+ 查找联系人：按照姓名查看指定联系人信息
+ 修改联系人：按照姓名重新修改指定联系人
+ 清空联系人：清空通讯录中所有信息
+ 退出通讯录：退出当前使用的通讯录

分析：

+ 需要一个数组存储联系人记录
+ 需要一个变量存储联系人记录条数
+ 增加删除操作需要更新联系人记录条数
+ 查找操作需要判断联系人是否存在
+ 使用`switch`选择结构来使用菜单功能
+ 使用新功能前需要对窗口刷新
+ 对每个功能的使用需要用户提示

代码：

# 1.联系人结构体 - structPeople.h

```c++
#pragma once
#include <iostream>;
using namespace std;

struct people {
	//姓名
	string name;
	//性别
	string sex;
	// 年龄
	int age;
	// 联系电话
	int phone;
	// 家庭住址
	string address;
};

```



# 2.菜单 - menu

1. 头文件 - menu.h

   ```c++
   #pragma once
   #include <iostream>
   using namespace std;
   
   void menu();
   ```

   

2. 源文件 - menu.cpp

   ```c++
   #include "menu.h"
   
   void menu() {
   	cout << "菜单：" << endl;
   	cout << "1. 添加联系人" << endl;
   	cout << "2. 显示联系人" << endl;
   	cout << "3. 删除联系人" << endl;
   	cout << "4. 查找联系人" << endl;
   	cout << "5. 修改联系人" << endl;
   	cout << "6. 清空联系人" << endl;
   	cout << "0. 退出通讯录" << endl;
   	cout << endl;
   	cout << "输入数字(0 - 6)并回车，使用相应功能" << endl;
   	cout << endl;
   }
   ```

   

# 3.添加联系人 - addPeople

1. 头文件 - addPeople.h

   ```c++
   #include <iostream>
   #include "structPeople.h"
   using namespace std;
   
   //通讯录添加
   //peopleArr：存放成员信息的数组
   //num：数组中存在的成员个数
   void addPeople(people * peopleArr);
   
   ```

   

2. 源文件 - addPeople.cpp

   ```c++
   #include "addPeople.h"
   
   
   //通讯录添加
   /*
   struct people {
   	//姓名
   	string name;
   	//性别
   	string sex;
   	// 年龄
   	int age;
   	// 联系电话
   	int phone;
   	// 家庭住址
   	string address;
   };
   */
   void addPeople(people* peopleArr) {
   	if (peopleArr[0].age > 999) {
   		cout << "通讯录最多记录一千人" << endl;
   		return;
   	}
   
   
   	//声明变量
   	// 声明对象
   	people newPeople;
   
   
   	//获取用户输入值
   	cout << "请您开始添加用户：" << endl;
   	cout << endl;
   	cout << "请您输入姓名：" << endl;
   	cin >> newPeople.name;
   	cout << "请您输入性别：" << endl;
   	cin >> newPeople.sex;
   	cout << "请您输入年龄：" << endl;
   	cin >> newPeople.age;
   	cout << "请您输入联系电话：" << endl;
   	cin >> newPeople.phone;
   	cout << "请您输入家庭住址：" << endl;
   	cin >> newPeople.address;
   	
   	//将对象添加到数组中,并更新成员数量
   	peopleArr[++peopleArr[0].age] = newPeople;
   	cout << endl;
   	cout << "添加成功" << endl;
   
   }
   ```

   

# 4.显示联系人 - showPeople

1. 头文件 - showPeople.h

   ```c++
   #pragma once
   #include <iostream>
   #include "structPeople.h"
   using namespace std;
   
   void showPeople(people * peopleArr);
   ```

   

2. 源文件 - showPeople.cpp

   ```c++
   #include "structPeople.h"
   
   void showPeople(people* persons) {
   	if (persons[0].age == 0)
   	{
   		cout << "通讯录中尚未存在人员记录" << endl;
   
   	}
   	for (int i = 1; i <= persons[0].age; i++) {
   		cout << i << "."
   			<< "	姓名：" << persons[i].name
   			<< "	性别：" << persons[i].sex
   			<< "	年龄：" << persons[i].age
   			<< "	联系电话：" << persons[i].phone
   			<< "	家庭住址：" << persons[i].address << endl;
   	}
   }
   ```

   

# 5.删除联系人 - deletePeople

1. 头文件 - deletePeople.h

   ```c++
   #pragma once
   #include <iostream>
   using namespace std;
   #include "structPeople.h"
   
   //删除联系人
   //persons：通讯录数组
   void deletePeople(people* persons);
   ```

   

2. 源文件 - deletePeople.cpp

   ```c++
   #include "deletePeople.h"
   
   
   void deletePeople(people* persons) {
   	string name;
   
   	//输入
   	cout << "请输入要删除的联系人的名字" << endl;
   	cin >> name;
   
   	int count = 0;//对查找到的联系人计数
   
   	//遍历查找
   	for (int i = persons[0].age; i > 0; i--) {
   		if (persons[i].name == name) {
   			count++;
   			for (int j = i; j <= persons[0].age; j++) {
   				persons[j] = persons[j + 1];
   			}
   		}
   	}
   	if (count == 0) {
   		cout << "联系人不存在" << endl;
   	}
   	else {
   		persons[0].age -= count;
   		cout << "成功删除联系人" << endl;
   	}
   }
   ```

   

# 6.查找联系人 - findPeople

1. 头文件 - findPeople.h

   ```c++
   #pragma once
   #include <iostream>
   using namespace std;
   #include "structPeople.h"
   
   //查找联系人
   //persons：通讯录数组
   void findPeople(people* persons);
   ```

   

2. 源文件 - findPeople.cpp

   ```c++
   #include"findPeople.h"
   
   void findPeople(people* persons) {
   	string name;
   
   	//输入
   	cout << "请输入要查找的联系人的名字" << endl;
   	cin >> name;
   	cout << endl;
   
   
   	int count = 0;//对查找到的联系人计数
   
   	//遍历查找
   	for (int i = 1; i <= persons[0].age; i++) {
   		if (persons[i].name == name) {
   			count++;
   			cout << i << "."
   				<< "	姓名：" << persons[i].name
   				<< "	性别：" << persons[i].sex
   				<< "	年龄：" << persons[i].age
   				<< "	联系电话：" << persons[i].phone
   				<< "	家庭住址：" << persons[i].address << endl;
   		}
   	}
   	cout << endl;
   	if (count == 0) {
   		cout << "联系人不存在" << endl;
   	}
   	else {
   		cout << "成功查找到联系人" << endl;
   	}
   }
   ```

   

# 7.修改联系人 - changePeople

1. 头文件 - changePeople.h

   ```c++
   #pragma once
   #include <iostream>
   using namespace std;
   #include "structPeople.h"
   
   void changePeople(people* persons);
   ```

   

2. 源文件 - changePeople.cpp

   ```c++
   #include "changePeople.h"
   
   void changePeople(people* persons) {
   	string name;
   
   	//输入
   	cout << "请输入要修改的联系人的名字" << endl;
   	cin >> name;
   
   	int count = 0;//对查找到的联系人计数
   
   	//遍历查找
   	for (int i = 1; i <= persons[0].age; i++) {
   		if (persons[i].name == name) {
   			count++;
   			//获取用户输入值
   			cout << "请您开始添加用户：" << endl;
   			cout << endl;
   			cout << "请您输入姓名：" << endl;
   			cin >> persons[i].name;
   			cout << "请您输入性别：" << endl;
   			cin >> persons[i].sex;
   			cout << "请您输入年龄：" << endl;
   			cin >> persons[i].age;
   			cout << "请您输入联系电话：" << endl;
   			cin >> persons[i].phone;
   			cout << "请您输入家庭住址：" << endl;
   			cin >> persons[i].address;
   		}
   	}
   	if (count == 0) {
   		cout << "联系人不存在" << endl;
   	}
   	else {
   		cout << "成功修改联系人" << endl;
   	}
   }
   ```

   

# 8.清空联系人 - cleanPeople

1. 头文件 - cleanPeople.h

   ```c++
   #pragma once
   #include <iostream>
   using namespace std;
   #include "structPeople.h"
   
   void cleanPeople(people* persons);
   ```

   

2. 源文件 - cleanPeople.cpp

   ```c++
   #include "cleanPeople.h"
   
   void cleanPeople(people* persons) {
   
   	for (int j = persons[0].age; j > 0 ; j--) {
   		persons[j] = persons[j + 1];
   	}
   	persons[0].age = 0;
   
   	cout << "成功清空通讯录" << endl;
   }
   ```

   

# 9.退出系统 - outSystem

1. 头文件 - outSystem.h

   ```c++
   #include <iostream>
   using namespace std;
   
   //退出系统
   void outSystem();
   
   ```

   

2. 源文件 - outSystem.cpp

   ```c++
   #include "outSystem.h"
   
   //退出系统
    void outSystem(){
   	 cout << "感谢使用此系统" << endl;
   	 exit(0);
   }
   ```

   

# 10.主函数 - index.cpp

```c++
#include <iostream>
using namespace std;
#include "structPeople.h"//联系人成员结构体
#include "menu.h"//菜单
#include "addPeople.h"//添加联系人
#include "showPeople.h"//显示所有联系人
#include "deletePeople.h"//删除联系人
#include "findPeople.h"//查找联系人
#include "changePeople.h"//修改联系人
#include "cleanPeople.h"//清空联系人
#include "outSystem.h"//退出通讯录


int main() {
	people persons[1001];
	persons[0].age = 0;//使用第一个元素保存通讯录中成员个数
	int tag = -1;
	menu();
	while (1) {
		cin >> tag;
		switch (tag) {	
		case 0 :
			system("cls");
			menu();
			outSystem();
			break;
		case 1:
			system("cls");
			menu();
			addPeople(persons);
			break;
		case 2:
			system("cls");
			menu();
			showPeople(persons);
			break;
		case 3:
			system("cls");
			menu();
			deletePeople(persons);
			break;
		case 4:
			system("cls");
			menu();
			findPeople(persons);
			break;
		case 5:
			system("cls");
			menu();
			changePeople(persons);
			break;
		case 6:
			system("cls");
			menu();
			cleanPeople(persons);
			break;
		default:
			system("cls");
			menu();
			cout << "请根据菜单输入" << endl;
			break;
		}	
	}
	system("pause");
	return 0;
}
```

