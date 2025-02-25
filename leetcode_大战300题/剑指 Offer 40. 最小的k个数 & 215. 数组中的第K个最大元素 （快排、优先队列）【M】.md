# [215. 数组中的第K个最大元素 ](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)& [剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

这两个题本质上是一个题，典型的topK问题，**一定要掌握**

这个题的考点：多种解法（快排、优先队列），快排的实现

<img src="pic\image-20210315213943738.png" alt="image-20210315213943738"  />

可以想到：暴力解，即将数组sort一下，然后获取index=k-1的数字，就是topk——时间复杂度为O(NlogN)

## 1. 快排

首先，我们知道Java内部的sort就是用了改进版的快排。那么快排的主要思想就是：**分而治之**，核心方法，就是partition，每一次调用partition之后，都能确定一个结点的位置index（之后不会再变了），**而该结点之前都是小于的数，该结点之后都是大于的数**

那么我们可以利用这个index来判断是否已经得到了topk个数：

- 如果index>k-1，那么说明topk出现在index之前，对index再次进行partition；
- 如果index<k-1，那么说明index之前的就是topk的一部分，还需要再后面再找几个，所以要对index+1~right的范围再次进行partition

——如此循环，最后得到index=k-1，那么k-1及其之前的数就是topk

```java
class Solution {
    private int partition(int[] nums, int left, int right){             // 划分
        int pivot = nums[left];
        while(left < right){
            while(left < right && nums[right] < pivot) right--;			// 降序排列
            if(left < right) nums[left++] = nums[right];
            while(left < right && nums[left] > pivot) left++;
            if(left < right) nums[right--] = nums[left];
        }
        nums[left] = pivot;             // 将基准值插入到数组中
        return left;
    }
    private void quickSort(int[] nums, int left, int right, int k){
        if(left > right) return;
        int index = partition(nums, left, right);
        if(index + 1 > k) quickSort(nums, left, index - 1, k);
        else if(index + 1 < k ) quickSort(nums, index + 1, right, k); 
    }
    public int findKthLargest(int[] nums, int k) {
        quickSort(nums, 0, nums.length - 1, k);		// 直接调用
        return nums[k - 1];
    }
}
```

稍微优化一下，不需要使用递归方法：

```java
class Solution {
    private int partition(int[] nums, int left, int right){             // 划分
        int pivot = nums[left];
        while(left < right){
            while(left < right && nums[right] < pivot) right--;
            if(left < right) nums[left++] = nums[right];
            while(left < right && nums[left] > pivot) left++;
            if(left < right) nums[right--] = nums[left];
        }
        nums[left] = pivot;             // 将基准值插入到数组中
        return left;
    }
    public int findKthLargest(int[] nums, int k) {
        int left = 0, right = nums.length - 1;
        while(true){
            int index = partition(nums, left, right);
            if(index == k - 1) break;
            else if(index > k - 1) right = index - 1;
            else left = index + 1;
        }
        return nums[k - 1];
    }
}
```

当然，这个partition还有多种写法：

```java
private int partition(int[] nums, int left, int right){
    int pivot = nums[left];
    int j = left;
    for(int i = left + 1; i <= right; i++){
        if(nums[i] > pivot){			// 将所有大于基准值的数字都和前面小于基准值的数字交换
            j++;
            swap(nums, j, i);
        }
    }
    swap(nums, left, j);			// 将最后一个大于基准值的数字和基准值交换位置，此时的位置就是划分之后的结果
    return j;
}
private void swap(int[] nums, int i, int j){
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

——注意如果是选择固定的基准值时间性能并不好，因为存在极端测试样例，所以需要随机选择基准值：

```java
private static Random random = new Random(System.currentTimeMillis());
private int partition(int[] nums, int left, int right){             // 划分
    if (right > left) {			// 必须不超过范围
        int randomIndex = left + 1 + random.nextInt(right - left);		// 选择随机index数
        int temp = nums[randomIndex];
        nums[randomIndex] = nums[left];
        nums[left] = temp;
    }
    int pivot = nums[left];
	....
}
```

时间复杂度分析：快排：T(N)=2T(N/2) + N，根据主定理，得到：O(NlogN)，而此时每次都只是查找partition之后的一半，其余一半并不考虑，所以T(N)=T(N/2)+N，根据主定理，得到：**O(N)**

空间复杂度为O(1)

## 2. 优先队列

要获取第k大的数，还可以用优先队列——就是排序以后后半部分最小的那个元素

用最小堆（堆顶就是堆中最小的数），堆的大小限制为k：遍历数组

1. 如果堆未满，就不断添加结点（最小堆始终会维护堆顶是最小的数）
2. 如果堆满了，那么，结点和堆顶做比较，如果小于堆顶，就放弃；如果大于堆顶就插入，堆顶被抛出

——遍历结束后，堆中存放的就是最大的k个数，且堆顶就是第k大的数（就是其中最小的数）

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> heap = new PriorityQueue<>();
        for(int i = 0; i < k; i++){
            heap.add(nums[i]);
        }
        for(int i = k; i < nums.length; i++){
            if(nums[i] > heap.peek()){
                heap.poll();
                heap.add(nums[i]);
            }
        }
        return heap.peek();
    }
}
```

时间复杂度为O(N)，空间复杂度为O(N)

当然也可以用最大堆：存在两种方法：

- 将所有结点均加入到最大堆中，然后弹出k-1个结点，那么此时的栈顶就是第k大的结点；

- 可以反着来，第k个大的数 = 第n-k+1个小的数，那么就用最大堆存放n-k+1个数，然后栈顶就是需要的数

  ——最大堆和最小堆可以根据实际情况进行转换，转换的分界线在 k = n - k，看哪个需要的堆的空间小。

具体的实现见参考的题解

参考：

1. https://leetcode-cn.com/problems/kth-largest-element-in-an-array/solution/partitionfen-er-zhi-zhi-you-xian-dui-lie-java-dai-/

# [剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

<img src="pic\image-20210315213819151.png" alt="image-20210315213819151"  />

本题和上题一样，只不过这次需要将前面的数字全部输出，本质没有变化。

下面复习一遍：

1. 快排

```java
class Solution {
    private int partition(int[] nums, int left, int right){
        int pivot = nums[left];
        while(left < right){
            while(left < right && nums[right] > pivot) right--;
            if(left < right) nums[left++] = nums[right];
            while(left < right && nums[left] < pivot) left++;
            if(left < right) nums[right--] = nums[left];
        }
        nums[left] = pivot;
        return left;
    }
    public int[] getLeastNumbers(int[] arr, int k) {
        if(k == 0 || arr.length == 0) return new int[0];
        int left = 0, right = arr.length - 1;
        while(true){
            int index = partition(arr, left, right);
            if(index == k - 1) break;
            else if(index > k - 1) right = index - 1;
            else left = index + 1;
        }
        return Arrays.copyOf(arr, k);
    }
}
```

2. 优先队列

由于要求的是最小的k个数，那么需要用最大堆

```java
class Solution {
    public int[] getLeastNumbers(int[] arr, int k) {
        if(k == 0 || arr.length == 0) return new int[0];
        PriorityQueue<Integer> heap = new PriorityQueue<>(k, (a, b) -> (b - a));
        for(int i = 0; i < k; i++){
            heap.add(arr[i]);
        }
        for(int i = k; i < arr.length; i++){
            if(arr[i] < heap.peek()){
                heap.poll();
                heap.add(arr[i]);
            }
        }
        int[] res = new int[k];
        for(int i = 0; i < k; i++){
            res[i] = heap.poll();
        }
        return res;
    }
}
```

https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/solution/3chong-jie-fa-miao-sha-topkkuai-pai-dui-er-cha-sou/还提出了一个二叉搜索树的解法，本质上和最大堆一样，可以调用库TreeMap，来维护一棵二叉搜索树，具体见题解，主要是对TreeMap的使用。

总结：

1. 快排的模板需要记住，partition不能写错
2. 最大堆最小堆虽然之间可以转换，但是常规来说：求最小的topk问题用最大堆；求最大的topk问题用最小堆

# [347. 前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

<img src="pic\image-20210318101738157.png" alt="image-20210318101738157"  />

本题本质上还是topk问题，只不过解题流程上稍微需要修改

主要还是要**熟悉哈希和优先队列的常见方法**，并能够加以使用

题目要求：统计数组中每个元素出现的频率，然后将出现次数最多的k个数获取出来。

- 首先，我们需要统计元素出现的个数，由于元素的大小在int范围内均可，所以必须要用到哈希表——遍历一次数组，然后将数组中所有的数字出现的次数进行统计，放入哈希表中——key： 数字；value：出现次数
- topk问题的解法是快排和优先队列，而快排是对数组进行排序，如果要用这个我们需要遍历哈希表将数字取出来并且按照key构建数组，然后根据value进行排序；如果是优先队列，我们需要构建的是最小堆，并且需要自定义比较器，然后遍历哈希表将topk数字获取出。

优先队列版本：

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        HashMap<Integer, Integer> store = new HashMap<>();
        for(int num: nums){                 // 统计数组中出现的次数
            if(store.containsKey(num)){
                store.put(num, store.get(num) + 1);
            }
            else{
                store.put(num, 1);
            }
        }
        PriorityQueue<Integer>queue = new PriorityQueue<>(new Comparator<Integer>(){        // 最小堆，需要重写比较器
            @Override
            public int compare(Integer o1, Integer o2){
                return store.get(o1) - store.get(o2);				// 比较的是出现的频率，通过调用哈希表来获得频率
            }
        });
        int i = 0;
        for(Integer key: store.keySet()){           // 哈希表遍历
            if(i < k){					// < k直接填充
                queue.add(key);
            }
            if(i >= k){
                int num = store.get(queue.peek());			// >=k 需要与堆顶进行比较，出现频率高才能进行替换
                if(num < store.get(key)){
                    queue.poll();
                    queue.add(key);
                }
            }
            i++;
        }
        int[] res = new int[k];					// 取出堆中元素
        for(int j = 0; j < k; j++){
            res[j] = queue.poll();
        }
        return res;
    }
}
```

收获：

哈希表的遍历方法：`for(Integer key: map.keySet())`

自定义堆的比较器：

```java
PriorityQueue<Integer> queue = new PriorityQueue<>(new Comparator<Integer>(){
	@Override
    public int compare(Integer o1, Integer o2){
        return map.get(o1) - map.get(o2);
    }
});
```

快排版本：

```java
class Solution {
    private int partition(HashMap<Integer, Integer>store, int[] nums, int left, int right){

        int pivot = nums[left];
        while(left < right){
            while(left < right && store.get(nums[right]) < store.get(pivot)) right--;
            if(left < right) nums[left++] = nums[right];
            while(left < right && store.get(nums[left]) > store.get(pivot)) left++;
            if(left < right) nums[right--] = nums[left];
        }
        nums[left] = pivot;
        return left;
    }
    public int[] topKFrequent(int[] nums, int k) {
        HashMap<Integer, Integer> store = new HashMap<>();
        for(int num: nums){                 // 统计数组中出现的次数
            if(store.containsKey(num)){
                store.put(num, store.get(num) + 1);
            }
            else{
                store.put(num, 1);
            }
        }
        int[] res = new int[store.size()];      
        int i = 0;
        for(Integer key: store.keySet()){           // 遍历哈希表，将所有的key写入一个数组中
            res[i++] = key; 
        }
        int left = 0, right = res.length - 1;
        while(true){
            int mid = partition(store, res, left, right);
            if(mid == k - 1) break;
            else if(mid > k - 1) right = mid - 1;
            else left = mid + 1;
        }
        return Arrays.copyOf(res, k);
    }
}
```

参考题解，还提出了一个 解法：**桶排序**（即用哈希表统计每个数字的出现次数后，自己再后构建一个建议的哈希表），长度为nums.length+1的数组（包括0，那么方便插入），每个index代表出现次数，所以最多是整个数组都是同一个数，那么最大出现次数为nums.length。然后遍历哈希表，根据出现次数进行index对应，如果出现相同的出现次数，就创建链表挂在index上。最后从后向前遍历获得k个结点即可

```java
//基于桶排序求解「前 K 个高频元素」
class Solution {
    public List<Integer> topKFrequent(int[] nums, int k) {
        List<Integer> res = new ArrayList();
        // 使用字典，统计每个元素出现的次数，元素为键，元素出现的次数为值
        HashMap<Integer,Integer> map = new HashMap();
        for(int num : nums){
            if (map.containsKey(num)) {
               map.put(num, map.get(num) + 1);
             } else {
                map.put(num, 1);
             }
        }
        
        //桶排序
        //将频率作为数组下标，对于出现频率不同的数字集合，存入对应的数组下标
        List<Integer>[] list = new List[nums.length+1];		// 数组，里面的元素是arraylist
        for(int key : map.keySet()){
            // 获取出现的次数作为下标
            int i = map.get(key);
            if(list[i] == null){
               list[i] = new ArrayList();
            } 
            list[i].add(key);
        }
        
        // 倒序遍历数组获取出现顺序从大到小的排列
        for(int i = list.length - 1;i >= 0 && res.size() < k;i--){
            if(list[i] == null) continue;
            res.addAll(list[i]);
        }
        return res;
    }
}
```

参考：

https://leetcode-cn.com/problems/top-k-frequent-elements/solution/leetcode-di-347-hao-wen-ti-qian-k-ge-gao-pin-yuan-/