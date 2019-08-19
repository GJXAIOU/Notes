---
pin: true
---
# 错误


## 1.源代码
```java
			String origin = scanner.nextLine();
			String handle = origin.toLowerCase();
			String ans = handle.replace("marshtomp", "fjxmlhx");
			System.out.println(ans);	
```
报错：`Resource leak: 'scanner' is never closed	`

解决之后代码：
```java
		try {
			String origin = scanner.nextLine();
			String handle = origin.toLowerCase();
			String ans = handle.replace("marshtomp", "fjxmlhx");
			System.out.println(ans);
		} finally {
			scanner.close();
		}
```
