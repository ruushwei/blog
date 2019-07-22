---
title: concurrentmodificationexception源码解析
date: 2019-05-23T12:54:24+02:00
tags: 
- java
- List
categories: java
---

##### Java安全开发手册中

- `7. 【强制】不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁。`



##### 个人总结

- 在迭代器中走list的remove大部分情况都会抛 ConcurrentModificationException 异常，从迭代器的next、remove中抛出，因new迭代器类的时候，私有变量expectedModCount会记录修改次数，当List的modCount不一致时会抛出异常，而List的add,remove等修改操作都会增加modCount
- 也会有在删除某些元素后导致迭代器cursor和size正好相等，hasnext返回false, 不再遍历就不抛异常，但不会遍历到后面移动到已删除位置的元素

- List接口中有iterator()接口，因此List的子类都要注意这点

<!--more-->

##### 迭代器源码

```java
a.iterator()
-->
public Iterator<E> iterator() {
        return new Itr();
}
```



```java
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```



##### ArrayList源码

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
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```



##### 情况1. 特殊情况例如删除倒数第二个元素不抛异常，但会漏遍历倒数第一个元素

下面这段代码不会出异常，异常都是走到了迭代器的 next 或 remove 方法，通过 checkForComodification 方法抛出，此例删除倒数第二个，cursor变为1 (初始0)，size也变为1，在hasNext方法中返回false，则不会走到"2"， 也不会走到next, 但这种漏了"2" ,是有很大问题的

```java
List<String> a = new ArrayList<>();
a.add("1");
a.add("2");
Iterator<String> iterator = a.iterator();
while (iterator.hasNext()) {
    String temp = iterator.next();
    if("1".equals(temp)){
        a.remove(temp);
    }
}
System.out.println(a.toString());
```



##### 情况2.大多数情况，因为list.remove增加了modcount,  导致next中判断modcount 与 expectmodcount不相等抛异常

下面这段代码会抛异常，因为最后cursor变为了2，size经过remove变成了1，根据hasNext方法cursor!=size，会返回true，走到next方法，然后next方法中 checkForComodification 会因为modcount 与 expectmodcount不相等而抛异常 ConcurrentModificationException

```java
List<String> a = new ArrayList<>();
a.add("1");
a.add("2");
Iterator<String> iterator = a.iterator();
while (iterator.hasNext()) {
    String temp = iterator.next();
    if("2".equals(temp)){
        a.remove(temp);
    }
}
System.out.println(a.toString());
```