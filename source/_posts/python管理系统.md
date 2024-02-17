---
title:  基于python的简单学生信息管理系统
date: 2024/2/17 23:00:00  # 文章发表时间
tags:
thumbnail: https://lixudongself.eu.org/hexo_image/post9.png
---

​		针对传统的学生信息管理方式，效率低下，不易存储，和数字化等问题，开发基于 Python 的学生信息管理系统，用于管理学生的个人信息和学习记录。它可以帮助教师和管理员更有效地管理学生信息，如学生基本信息、成绩、课程安排、考试记录等。同时，信息化、数字化的管理方法符合高校建设数字校园的宗旨，有利于促进信息的保存和共享。而Python相对于其他语言具有简单易学，易维护等特点，在系统开发领域应用十分广泛。本系统是由Python语言进行开发的一套简易学生信息管理系统，功能齐全，操作简便，实用性高。

## 1系统方案设计

​		学生信息管理系统功能如下表所示。该系统包括学生信息输入、信息查找、信息的删除与修改、信息展示等一系列功能。最终使用[Pyinstaller](https://so.csdn.net/so/search?q=Pyinstaller&spm=1001.2101.3001.7020)将程序打包为exe程序，方便在没有部署python环境的计算机上正常使用我们的管理系统。

![](https://lixudongself.eu.org/hexo_image/post7.png)

## 2功能设计

​		我们将每一个所有功能进行拆分，将每一种功能单独定义为一个函数，然后将主函数main在系统中循环调用，以实现选择功能的作用。

​		Menu函数主要用来输出管理系统的可视化菜单页面，配合main函数来完成功能的选择。

​		Insert函数用来实现输入学生信息的功能。依次输入学号姓名和语数英成绩，并且在输入成绩时，因为是int型，所以使用了try和except，以保证正确输入不会出错。将输入的信息储存为一个字典，并最终保存在一个名为student.txt的文件中。为了方便使用，在函数的末尾加上了循环使用的功能，可以连续输入多名学生信息。

​		Search函数主要用来查找学生信息。该函数首先会查找是否存在学生信息文件“student.txt” 如果存在会以学号为索引，查找文件中所储存的信息，并调雍show_student函数将其以特定格式输出。删除函数delete、修改函数edit基本也是这个逻辑。

## 3编程实现与调试

```python
import os.path
 
filename = 'student.txt'
 
 
def menu():
    print('======学生信息管理系统======')
    print('1.输入学生信息')
    print('2.查找信息')
    print('3.删除信息')
    print('4.修改信息')
    print('5.显示出所有信息')
    print('0.退出系统')
    print('=========================')
 
 
def main():
    while True:
        menu()
        selection = int(input('请选择'))
        if selection in [0, 1, 2, 3, 4, 5]:
            if selection == 0:
                ans = input('是否要退出？y/n')
                if ans == 'y' or ans == 'Y':
                    print('感谢使用!!')
                    break
                else:
                    continue
            elif selection == 1:
                insert()
            elif selection == 2:
                search()
            elif selection == 3:
                delete()
            elif selection == 4:
                edit()
            elif selection == 5:
                display()
        else:
            print('选择错误')
 
 
# noinspection PyBroadException
def insert():
    student_list = []
    while True:
        stu_id = input('请输入学号:')
        if not stu_id:
            break
        name = input('请输入姓名:')
        if not name:
            break
        try:
            english = int(input('请输入英语成绩'))
            chinese = int(input('请输入语文成绩'))
            math = int(input('请输入数学成绩'))
        except:
            print('错误的成绩（不是整数）')
            continue
        student = {'stu_id': stu_id, 'name': name, 'english': english, 'chinese': chinese, 'math': math}
        student_list.append(student)
        ans = input('是否继续添加y/n\n')
        if ans == 'y' or ans == 'Y':
            continue
        else:
            break
    save(student_list)
    print('输入完毕!')
 
 
# noinspection PyBroadException
def save(lst):
    try:
        stu_txt = open(filename, 'a', encoding='utf-8')
    except:
        stu_txt = open(filename, 'w', encoding='utf-8')
    for item in lst:
        stu_txt.write(str(item) + '\n')
    stu_txt.close()
 
 
def search():
    stu_query = []
    while True:
        stu_id = ''
        if os.path.exists(filename):
            id = input('请输入学生学号:')
            with open(filename, 'r', encoding='utf-8') as rfile:
                student = rfile.readlines()
                for item in student:
                    d = dict(eval(item))
                    if id != '':
                        if d['stu_id'] == id:
                            stu_query.append(d)
            show_student(stu_query)
            stu_query.clear()
            ans = input('是否继续查询?y/n')
            if ans == 'y':
                continue
            else:
                break
        else:
            print('暂无学生信息')
            return
 
 
def show_student(lst):
    if len(lst) == 0:
        print('没有查询到数据！')
        return
    format_title = '{:^6}\t{:^12}\t{:^8}\t{:^9}\t{:^5}\t{:^2}\t'
    print(format_title.format('ID', '姓名', '英语成绩', '语文成绩', '数学成绩', '总成绩'))
    format_data = '{:^6}\t{:^12}\t{:^8}\t{:^10}\t{:^15}\t{:^6}\t'
    for item in lst:
        print(format_data.format(item.get('stu_id'),
                                 item.get('name'),
                                 item.get('english'),
                                 item.get('chinese'),
                                 item.get('math'),
                                 int(item.get('english')) + int(item.get('chinese')) + int(item.get('math'))))
 
 
def delete():
    while True:
        stu_id = input('请输入要删除的学号：')
        if stu_id != '':
            if os.path.exists(filename):
                with open(filename, 'r', encoding='utf-8') as file:
                    stu_old = file.readlines()
            else:
                stu_old = []
            flag = False
            if stu_old:
                with open(filename, 'w', encoding='utf-8') as wfile:
                    d = {}
                    for item in stu_old:
                        d = dict(eval(item))
                        if d['stu_id'] != stu_id:
                            wfile.write(str(d) + '\n')
                        else:
                            flag = True
                    if flag:
                        print(f'id为{stu_id}的学生信息已经被删除')
                    else:
                        print(f'没有找到{stu_id}的学生信息')
            else:
                print('无学生信息')
                break
            display()
            ans = input('是否继续删除？y/n\n')
            if ans == 'y' or ans == 'Y':
                continue
            else:
                break
 
 
def edit():
    display()
    if os.path.exists(filename):
        with open(filename, 'r', encoding='utf-8') as rfile:
            student_old = rfile.readlines()
    else:
        return
    student_id = input('请输入要修改的学生学号：')
    with open(filename, 'w', encoding='utf-8') as wfile:
        for item in student_old:
            d = dict(eval(item))
            if d['stu_id'] == student_id:
                print('已经找到学生信息，请修改：')
                while True:
                    try:
                        d['name'] = input('请输入姓名')
                        d['english'] = input('请输入英语成绩')
                        d['chinese'] = input('请输入语文成绩')
                        d['math'] = input('请输入数学成绩')
                    except:
                        print('输入有误，重新输入')
                    else:
                        break
                wfile.write(str(d) + '\n')
                print('修改成功')
            else:
                wfile.write(str(d) + '\n')
        ans = input('是否继续修改？y/n')
        if ans == 'y':
            edit()
 
 
def display():
    student_list=[]
    if os.path.exists(filename):
        with open(filename,'r',encoding='utf-8') as rfile:
            students=rfile.readlines()
            for item in students:
                student_list.append(eval(item))
            if student_list:
                show_student(student_list)
    else:
        print('没有学生信息')
 
 
if 1 == 1:
    main()
```

## 4结果分析

### 1打开程序

![](https://lixudongself.eu.org/hexo_image/post9.png)

### 2选择输入学生信息功能

![](https://lixudongself.eu.org/hexo_image/post10.png)

依次输入学生信息完毕之后返回主菜单再次进行功能选择。

### 3查找信息

![](https://lixudongself.eu.org/hexo_image/post11.png)

### 4删除信息

![](https://lixudongself.eu.org/hexo_image/post12.png)

### 5显示出所有信息

![](https://lixudongself.eu.org/hexo_image/post13.png)

### 6最终会在同目录生成一个student.txt文件储存信息

![](https://lixudongself.eu.org/hexo_image/post14.png)

​		经过以上的测试结果表明，本系统运行情况比较良好，设计功能都能实现，系统反应速度较快，用户使用也相对简便，达到了程序设计前所设定的要求。总体而言，是一款比较成功的系统程序。

