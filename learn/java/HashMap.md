### HashMap的使用经验

---

`computeIfAbsent`get的时候如果获取的值为null 可以指定默认值 

```java
map.computeIfAbsent(key,k->defaultValue);
```

hashMap的树化条件或者树操作

- 链表的长度达到9后 `意味着如果一条链表上只有8个节点是不会树化的` (如果槽的节点减少的时候可能会链表化)
- 当前的table数组的长度(就是槽)需要大于等于64 才能进行树化或者树操作

hashMap的去树化的条件

- 调用resize方法 并且 当前槽的节点小于等于6的时候
- 还有一个地方 老子看不懂 md明明debug是进不去的

```java
if (root == null
    || (movable&& (root.right == null || (rl = root.left) == null || rl.left == null))) {
  tab[index] = first.untreeify(map);  // too small
  return;
}
```

