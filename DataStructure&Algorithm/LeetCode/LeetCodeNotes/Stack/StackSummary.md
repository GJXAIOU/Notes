# StackSummary

- Stack 中 add 和 push方法区别

    - 共同点：都可以向栈中添加元素

    - 不同点：

        - add 是继承自 Vector 的方法，且返回值类型是 boolean。即 True or False；
    
        ```java
            public synchronized boolean add(E e) {
            modCount++;
                ensureCapacityHelper(elementCount + 1);
                elementData[elementCount++] = e;
                return true;
            }
            ```
    
        - push 是 Stack 自身的方法，返回值类型是参数类类型。返回添加的元素值；
    
        ```java
            public E push(E item) {
                addElement(item);
                return item;
            }
            ```
    
            

        
    

