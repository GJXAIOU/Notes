---
tags : 
- java基础

flag: blue
---

@toc

# JavaDay14-球员管理项目


![球员]($resource/%E7%90%83%E5%91%98%202.jpg)

![球队]($resource/%E7%90%83%E9%98%9F.jpg)

![项目架构]($resource/%E9%A1%B9%E7%9B%AE%E6%9E%B6%E6%9E%84.jpg)


具体代码：
player
```java
package src.com.qfedu.entity;

/*
 球员实体类：
 	成员变量：
 		姓名，编号，年龄，工资，位置
 	成员方法：
 		投篮，传球
 */
public class Player {
	//成员变量
	private String name;
	private int id; //实现ID的自动增长，这里要使用计数器
	private int age;
	private double salary;
	private String location;
	
	private static int count = 1; //计数器
	
	//构造代码块来完成计数器操作
	{
		this.id = count++;
	}
	
	public Player() {}
	
	public Player(String name, int age, double salary, String location) {
		//这里使用自己的set方法，来完成赋值成员变量的操作
		this.setName(name);
		this.setAge(age);
		this.setSalary(salary);
		this.setLocation(location);
	}
	
	public int getId() {
		return id;
	}
	
	public String getName() {
		return name;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	
	public void setAge(int age) {
		if (age <= 0 || age > 45) {
			this.age = 19;
		} else {
			this.age = age;
		}
	}
	
	public double getSalary() {
		return salary;
	}
	
	public void setSalary(double salary) {
		if (salary <= 0) {
			this.salary = 1;
		} else {
			this.salary = salary;
		}
 	}
	
	public String getLocation() {
		return location;
	}
	
	public void setLocation(String location) {
		this.location = location;
	}
	
	//普通的成员方法页搞定
	public void shot() {
		System.out.println(this.getName() + "投篮练习中~~~");
	}
	
	public void passBall() {
		System.out.println(this.getName() + "传球练习中~~~");
	}

	//重写的Java中超类【Object】类中toString,toString是该对象的描述
	//当通过System.out.println(player)会自动调用的方法
	@Override
	public String toString() {
		return "[ID:" + this.id + " Name:" + this.name + " Age:" + this.age
				+ " Salary:" + this.salary + " Location:" + this.location + "]";
	}
}

```


 TeamManager
```java
package src.com.qfedu.dao;

import java.util.Scanner;

import src.com.qfedu.entity.Player;

public class TeamManager {
	//球队的名字
	private String teamName;
	//保存球员信息的数组
	private Player[] allPlayers = new Player[defaultCount];

	//统计当前球队中有多少球员
	private static int itemCount = 0; //元素个数
	//球队的默认球员个数
	private static final int defaultCount = 10; 

	public TeamManager() {}

	public TeamManager(String teamName) {
		this.setTeamName(teamName);
	}

	public void setTeamName(String teamName) {
		this.teamName = teamName;
	}

	public String getTeamName() {
		return teamName;
	}

	/*
	 * 添加球员
	 * 解雇球员
	 * 修改球员信息
	 * 查询球员信息
	 * 
	 * 排序算法
	 * 
	 * 数组增长
	 */
	/**
	 * 添加新球员到数组中
	 * @param playerToAdd 要添加的球员类对象
	 */
	public void addPlayer(Player playerToAdd) {
		//参数合法性判断(以后叫【异常处理】)
		if (null == playerToAdd) {
			System.out.println("球员信息为空，不可添加");
			return;
		}

		//类内有一个itemCount的静态成员变量，是用来统计插入的元素个数，而且也是下个元素保存的下标位置
		//因为是插入操作，所以要考虑数组的容量问题，如果插入的数据个数已经大于了数组的长度，需要扩容
		if (itemCount >= allPlayers.length) {
			//扩容
			grow();
		}

		allPlayers[itemCount++] = playerToAdd;
	}

	/**
	 * 通过球员的ID删除球员
	 * @param playerID
	 */
	public void layoffPlayerByPlayerID(int playerID) {
		//需求查询方法，调用类内私有化通过球员ID查询球员在数组中位置的方法，获取下标
		int index = findPlayerIndexByPlayerID(playerID);

		if (index >= 0) {
			//删除该位置的球员，数组整体左移
			/*   1 2 3 4 5
			 index 1   << 3
			 index 3   << 1
			 */
			for (int i = index; i < itemCount - 1; i++) {
				allPlayers[i] = allPlayers[i + 1];
			}
			//原本最后一个有效元素赋值为null
			allPlayers[itemCount - 1] = null;

			//球员的球员个数 - 1
			itemCount--;
		} else {
			System.out.println("查无此人，无法删除");
		}
	}

	/**
	 * 通过球员的ID，来查询球员的信息
	 * @param playerID 要展示的球员ID号
	 */
	public void showPlayerInfoByPlayerID(int playerID) {
		int index = findPlayerIndexByPlayerID(playerID);

		if (index > -1) {
			System.out.println(allPlayers[index]);
		} else {
			System.out.println("查无此人");
		}
	}

	/**
	 * 通过球员ID 修改球员信息
	 * @param playerID 需要修改信息的球员ID
	 */
	public void modifyPlayerInfoByPlayerID(int playerID) {
		int index = findPlayerIndexByPlayerID(playerID);
		Scanner sc = new Scanner(System.in);
		//表示找到球员，进行修改操作
		if (index > -1) {

			//while(true) switch - case
			int flag = 0;
			int choose = -1;
			Player temp = allPlayers[index];

			while (true) {
				System.out.println("修改" + temp.getId() + ":" + temp.getName() + "的信息");
				System.out.println("***Age:" + temp.getAge());
				System.out.println("***Salary:" + temp.getSalary());
				System.out.println("***Location:" + temp.getLocation());

				System.out.println("1. 修改球员姓名");
				System.out.println("2. 修改球员年龄");
				System.out.println("3. 修改球员工资");
				System.out.println("4. 修改球员位置");
				System.out.println("5. 退出");

				choose = sc.nextInt();
				sc.nextLine();

				switch (choose) {
				case 1:
					System.out.println("请输入球员的姓名:");
					String name = sc.nextLine();
					temp.setName(name);
					break;
				case 2:
					System.out.println("请输入球员的年龄:");
					int age = sc.nextInt();
					temp.setAge(age);
					break;
				case 3:
					System.out.println("请输入球员的工资:");
					double salary = sc.nextDouble();
					temp.setSalary(salary);
					break;
				case 4:
					System.out.println("请输入球员的位置:");
					String location = sc.nextLine();
					temp.setLocation(location);
					break;
				case 5:
					flag = 1;
					break;
				default:
					System.out.println("选择错误");
					break;
				} //switch(choose) - case

				if (1 == flag) {
					allPlayers[index] = temp;
					System.out.println("保存退出");
					break;
				}

			} //while (true)

		} else {
			System.out.println("查无此人");
		}
		//sc.close();
	} 

	
	
	/**
	 * 工资的降序排序
	 */
	public void descendingSelectSortBySalary() {
		//保护源数据！！！
		//1. 准备一个数组专门用来做排序操作，数组的大小和源数据有效元素个数一致
		Player[] sortArray = new Player[itemCount];

		//2. 数据拷贝
		for (int i = 0; i < itemCount; i++) {
			sortArray[i] = allPlayers[i];
		}

		//外层控制选择排序的次数
		for (int i = 0; i < itemCount - 1; i++) {
			int index = i;

			for (int j = i + 1; j < itemCount; j++) {
				if (sortArray[index].getSalary() < sortArray[j].getSalary()) {
					index = j;
				}
			}

			if (index != i) {
				Player temp = sortArray[index];
				sortArray[index] = sortArray[i];
				sortArray[i] = temp;
			}
		}

		showSortResult(sortArray);
	}

	/**
	 * 年龄的升序排序
	 */
	public void ascendingSelectSortByAge() {
		//保护源数据！！！
		//1. 准备一个数组专门用来做排序操作，数组的大小和源数据有效元素个数一致
		Player[] sortArray = new Player[itemCount];

		//2. 数据拷贝
		for (int i = 0; i < itemCount; i++) {
			sortArray[i] = allPlayers[i];
		}

		//外层控制选择排序的次数
		for (int i = 0; i < itemCount - 1; i++) {
			int index = i;

			for (int j = i + 1; j < itemCount; j++) {
				if (sortArray[index].getAge() > sortArray[j].getAge()) {
					index = j;
				}
			}

			if (index != i) {
				Player temp = sortArray[index];
				sortArray[index] = sortArray[i];
				sortArray[i] = temp;
			}
		}

		showSortResult(sortArray);
	}

	/**
	 * 自己研究一下冒泡排序
	 */
	public void descendingBubbleSortByAge() {
		//保护源数据！！！
		//1. 准备一个数组专门用来做排序操作，数组的大小和源数据有效元素个数一致
		Player[] sortArray = new Player[itemCount];

		//2. 数据拷贝
		for (int i = 0; i < itemCount; i++) {
			sortArray[i] = allPlayers[i];
		}

		//外层控制比较的轮次
		for (int i = 0; i < itemCount - 1; i++) {
			//内层控制两两比较次数
			for (int j = 0; j < itemCount - i - 1; j++) {

				if (sortArray[j].getAge() < sortArray[j + 1].getAge()) {
					Player temp = sortArray[j];
					sortArray[j] = sortArray[j + 1];
					sortArray[j + 1] = temp;
				}
			}
		} 

		showSortResult(sortArray);

	}



	public void showAllPlayers() {
		for (Player player : allPlayers) {
			if (null == player) {
				break;
			}
			System.out.println(player);
		}
	}

	/**
	 * 需要一个扩容方法，方法不需要参数，不要类外调用，只是在类内使用
	 */
	private void grow() {
		//1. 获取原数组元素个数
		int oldCapacity = this.allPlayers.length;

		//2. 通过原数组元素计算新的元素个数 , 大约相对于原本元素个数的1.5倍
		int newCapacity = oldCapacity + (oldCapacity >> 1);

		//3. 创建新的数组，元素格式是原本的1.5倍
		Player[] newArray = new Player[newCapacity];

		//4. 利用循环拷贝数据
		for (int i = 0; i < oldCapacity; i++) {
			newArray[i] = this.allPlayers[i];
		}

		//5. 地址交换
		this.allPlayers = newArray;
	}

	/**
	 * 私有化的方法，只提供给类内使用，用来获取指定球员ID在数组中下标位置
	 * @param playerID 要查询的球员ID号
	 * @return int类型，返回查询的数据在数组中的下标位置，如果没有找到，返回-1
	 */
	private int findPlayerIndexByPlayerID(int playerID) {
		//参数合法性判断
		if (playerID < 1 || playerID > 100) {
			System.out.println("传入球员ID不合法");
			return -1;
		}

		//用来保存球员的下标位置
		int index = -1;

		for (int i = 0; i < itemCount; i++) {
			//拿球员的ID和传入的ID进行比较
			if (allPlayers[i].getId() == playerID) {
				index = i;
				break;
			}
		}

		return index;
	}

	private void showSortResult(Player[] sortArray) {
		//增强for循环
		for (Player player : sortArray) {
			System.out.println(player);
		}
		//普通for循环
		//		for (int i = 0; i < sortArray.length; i++) {
		//			System.out.println(sortArray[i]);
		//		}
	}
}






















```


view
```java
package src.com.qfedu.view;

import java.util.Scanner;

import src.com.qfedu.dao.TeamManager;
import src.com.qfedu.entity.Player;

public class View {
	public static void main(String[] args) {

		TeamManager tm = new TeamManager("孤狼B组");

		Player p1 = new Player("森林狼", 21, 10000, "PG");
		Player p2 = new Player("西伯利亚狼", 19, 9000, "SF");
		Player p3 = new Player("鸵鸟", 20, 9500, "SG");
		Player p4 = new Player("卫生院", 22, 9800, "SG");
		Player p5 = new Player("恶狼", 22, 8800, "PF");
		Player p6 = new Player("老炮", 22, 9900, "C");

		tm.addPlayer(p1);
		tm.addPlayer(p2);
		tm.addPlayer(p3);
		tm.addPlayer(p4);
		tm.addPlayer(p5);
		tm.addPlayer(p6);

		int flag = 0;
		int choose = 0;
		int playerID = 0;
		Scanner sc = new Scanner(System.in);
		
		while (true) {
			
			
			System.out.println("欢迎来到孤狼B组");
			System.out.println("1. 展示所有成员");
			System.out.println("2. 添加新的成员");
			System.out.println("3. 退役老成员");
			System.out.println("4. 查询成员资料");
			System.out.println("5. 修改成员资料");
			System.out.println("6. 按照年龄排序");
			System.out.println("7. 按照收入排序");
			System.out.println("8. 退出");
		
			choose = sc.nextInt();
			sc.nextLine();
			
			switch (choose) {
				case 1:
					//展示所有队员
					tm.showAllPlayers();
					break;
				case 2:
					//添加新成员
					System.out.println("清输入新成员的名字");
					String name = sc.nextLine();
					
					System.out.println("请输入新成员年龄");
					int age = sc.nextInt();
					sc.nextLine();
					
					System.out.println("请输入新成员工资");
					double salary = sc.nextDouble();
					sc.nextLine();
					
					System.out.println("请输入新成员的位置");
					String location = sc.nextLine();
					
					Player playerToAdd = new Player(name, age, salary, location);
					tm.addPlayer(playerToAdd);
					break;
				case 3:
					//退役
					System.out.println("请输入要退役的老成员ID");
					playerID = sc.nextInt();
					sc.nextLine();
					
					tm.showPlayerInfoByPlayerID(playerID);
					
					System.out.println("是否要确定删除? Y or N");
					char ch = sc.nextLine().charAt(0);
					if ('Y' == ch || 'y' == ch) {
						tm.layoffPlayerByPlayerID(playerID);
					}
					
					break;
				case 4:
					//查询
					System.out.println("请输入要查询的老成员ID");
					playerID = sc.nextInt();
					sc.nextLine();
					
					tm.showPlayerInfoByPlayerID(playerID);
					
					break;
				case 5:
					System.out.println("请输入要修改成员ID");
					playerID = sc.nextInt();
					sc.nextLine();
					
					tm.modifyPlayerInfoByPlayerID(playerID);
					break;
				case 6:
					tm.ascendingSelectSortByAge();
					
					break;
				case 7:
					tm.descendingSelectSortBySalary();
					break;
				case 8:
					flag = 1;
					break;
	
				default:
					System.out.println("选择错误");
					break;
			}
			
			if (1 == flag) {
				System.out.println("啊，朋友再见~~~");
				break;
			}
		}
	}
}

```
