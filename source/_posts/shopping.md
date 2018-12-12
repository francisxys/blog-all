---
title: python购物车程序
copyright: true
date: 2018-12-07 17:09:41
abbrlink:
tags:
- python
categories:
- code
- python
---

## 程序：购物车程序
需求
* 1、启动程序后，让用户输入工资，然后打开商品列表
* 2、允许用户根据商品编号购买商品
* 3、用户选择商品后，检测余额是否够，够就直接扣款，不够就提醒
* 4、可随时退出，退出时，打印已购商品和余额

## code
``` python
# -*- coding:utf-8 -*-
#Author:Francis
product_list = [
    ('Iphone',5800),
    ('Mac Pro',9800),
    ('Bike',800),
    ('Watch',10600),
    ('Coffee',31),
    ('Alex Python',120),
]
shopping_list = []
salary = input("Input your salary:")
if salary.isdigit():
    salary = int(salary)
    while True:
        for index,item in enumerate(product_list):
            #print(product_list.index(item),item)
            print(index,item)
        user_choice = input("选择要买嘛？>>>:")
        if user_choice.isdigit():
            user_choice = int(user_choice)
            if user_choice < len(product_list) and user_choice >=0:
                p_item = product_list[user_choice]
                if p_item[1] <= salary: #买的起
                    shopping_list.append(p_item)
                    salary -= p_item[1]
                    print("Added %s into shopping cart,your current balance is \033[31;1m%s\033[0m" %(p_item,salary) )
                else:
                    print("\033[41;1m你的余额只剩[%s]啦，还买个毛线\033[0m" % salary)
            else:
                print("product code [%s] is not exist!"% user_choice)
        elif user_choice == 'q':
            print("--------shopping list------")
            for p in shopping_list:
                print(p)
            print("Your current balance:",salary)
            exit()
        else:
            print("invalid option")
```