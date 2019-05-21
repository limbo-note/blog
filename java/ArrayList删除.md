ConcurrentModificationException一般在集合迭代过程中被修改时抛出，所有的集合都不允许在迭代器模式中修改集合的结构，如put()、remove()操作



ArrayList中Iterator类的主要三个方法实现(hasNext、next、remove)：

```java
public boolean hasNext() {
    return cursor != size; // cursor代表下一个即将遍历到的元素下标
}

@SuppressWarnings("unchecked")
public E next() {
    checkForComodification();
    int i = cursor; // 调用next时，就是取cursor对应的元素
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1; // cursor 需要更新，指向下一个位置
    return (E) elementData[lastRet = i];  // 返回时，会将此次返回的元素位置记录在lastRet
}

public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.remove(lastRet); // 解释了为什么每次调用Iterator.remove()方法时，都需要先调用next()方法，因为其默认是删除lastRet指向的元素
        cursor = lastRet; // 删除后，原cursor代表的元素就到了lastRet所在的位置，需要更新cursor
        lastRet = -1; // lastRet复位，下一次调用Iterator.remove()前若不调用next()方法，其删除就会出错，因为不存在下标为-1的元素
        expectedModCount = modCount; // 两者相等，所以利用Iterator本身来删除元素，不会有ConcurrentModificationException
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```

ArrayList的remove(Object o)方法实现：

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);                
                return true;
            }
    }
    return false;
}
```

```java
private void fastRemove(int index) {
    modCount++; // 会修改modCount，但没修改expectedModCount
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

综上的代码，当使用for-each或者Iterator遍历时，若在此过程中调用了ArrayList.remove(Object o)方法或其他修改集合的方法，或使得集合内的modCount变量加1，而expectedModCount在这些方法中却保持不变；而Iterator的next()方法需要检查modCount和expectedModCount的相等性，若不等则抛出ConcurrentModificationException。所以，主要的矛盾在于，在调用只修改modCount的情况下，不能调用Iterator的next()方法

而正确的，使用Iterator的remove的方法进行元素删除时，会保证modCount和expectedModCount的相等性，所以此方法允许在Iterator遍历时使用；但需要注意，每次删除元素前需调用next()方法保证删除的元素下标的正确性

#### 一个偶然的情况：

若在ArrayList的Iterator遍历中，使用ArrayList.remove(Object o)方法来删除元素，而且删除的元素恰好时倒数第二个元素时，不会抛异常。

因为调用ArrayList.remove(Object o)后，集合大小变量size减1；若上一个遍历到的元素是倒数第二个元素，那此时cursor的值刚好与size相等，这就使得在遍历循环体中的hasNext()方法直接返回fasle，循环终止，不会触发下一次的next()调用，也就不会触发modCount和expectedModCount的相等性检查，就不会抛出ConcurrentModificationException异常

#### 最安全的删除方法

最安全的删除方法就是调用Iterator的remove方法，此方法能保证下标的正确性，保证modCount和expectedModCount相等

假设一个`ArrayList<String>`初始为["1","2","2","4"]

```java
for(Iterator<String> iter = list.iterator();iter.hasNext();){
    String s = iter.next();
    if("2".equals(s))
    	iter.remove();
}  // 删除后，集合为正确的["1","4"]
```

而，

```java
for(int i=0;i<list.size();i++){
    String s = list.get(i);
    if("2".equals(s))
          list.remove(s);
} // 虽然不会出现ConcurrentModificationException异常，但结果为["1","2",4"]
```

第二种方法不正确的原因是，ArrayList.remove(Object o)或ArrayList.remove(int index)方法在删除元素之后，其后的元素会向前移动，而遍历指针i并没有相应的减1，就会漏检查一个元素；而Iterator的remove方法会保证删除后下标不会发生这种情况（即源码中的cursor = lastRet语句）

