#### 常数时间插入删除和获取随机元素-380

设计一个支持在平均 时间复杂度 O(1) 下，执行以下操作的数据结构。

1. insert(val)：当元素 val 不存在时，向集合中插入该项。
2. remove(val)：元素 val 存在时，从集合中移除该项。
3. getRandom：随机返回现有集合中的一项。每个元素应该有相同的概率被返回。

```java
// 初始化一个空的集合。
RandomizedSet randomSet = new RandomizedSet();

// 向集合中插入 1 。返回 true 表示 1 被成功地插入。
randomSet.insert(1);

// 返回 false ，表示集合中不存在 2 。
randomSet.remove(2);

// 向集合中插入 2 。返回 true 。集合现在包含 [1,2] 。
randomSet.insert(2);

// getRandom 应随机返回 1 或 2 。
randomSet.getRandom();

// 从集合中移除 1 ，返回 true 。集合现在包含 [2] 。
randomSet.remove(1);

// 2 已在集合中，所以返回 false 。
randomSet.insert(2);

// 由于 2 是集合中唯一的数字，getRandom 总是返回 2 。
randomSet.getRandom();
```

**思路：**常数时间复杂度，考虑使用哈希表。

1. 使用 Set 和 数组 实现：

```java
class RandomizedSet {
    
    private Set<Integer> set;

    /** Initialize your data structure here. */
    public RandomizedSet() {
        set = new HashSet<>();
    }
    
    /** Inserts a value to the set. Returns true if the set did not already contain the 		specified element. */
    public boolean insert(int val) {
        if(set.contains(val)){
            return false;
        }
        else{
            set.add(val);
            return true;
        }
    }
    
    /** Removes a value from the set. Returns true if the set contained the specified 			element. */
    public boolean remove(int val) {
        if(set.contains(val)){
            set.remove(val);
            return true; 
        }
        else{
            return false;
        }
    }
    
    /** Get a random element from the set. */
    public int getRandom() {
        if(set.size() == 0){
            return 0;
        }
        Integer[] data = set.toArray(new Integer[set.size()]);
        int index = (int)(Math.random()*set.size());
        return data[index];
    }
}
```

2. 使用 HashMap 和 ArrayList 实现，ArrayList 删除最后一个元素的时间复杂度为 O (1)，因此用一个 HashMap 来存储元素对应的下标位置：

```java
class RandomizedSet {
    Random rand;
    Map<Integer, Integer> map;
    List<Integer> keys;
    int count;
    
    /** Initialize your data structure here. */
    public RandomizedSet() {
        rand = new Random();
        map = new HashMap<Integer, Integer>();// (key, list index)
        keys = new ArrayList<>();
        count = 0;
    }
    
    /** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
    public boolean insert(int val) {
        if(map.containsKey(val)){   return false;}
        keys.add(val);
        map.put(val, count++);
        return true;
    }
    
    /** Removes a value from the set. Returns true if the set contained the specified element. */
    public boolean remove(int val) {
        if(!map.containsKey(val)){   return false;}
        int index = map.get(val);
        int last = count-1;
        if(index != last){
            //update keys(list)   list at index==(lastkey)
            keys.set(index, keys.get(last));       
            //update map: (lastkey, index) lastkey's index in keys(list)    
            map.put(keys.get(last), index);        
        }
        keys.remove(last);
        map.remove(val);
        count--;
        return true;
    }
    
    /** Get a random element from the set. */
    public int getRandom() {
        return keys.get(rand.nextInt(count));
    }
}
```

