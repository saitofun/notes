# STL

## 容器选择

  标准顺序容器: vector, string, deque, list
  标准关联容器: set, multiset, map, multimap
  非标准顺序容器: slist, rope
  非标准关联容器: hash_set, hash_multiset, hash_map, hash_multimap
  其他非标准容器: bitset <-> vector<bool>; valarray; stack; queue; priority_queue
  
  场景:
  1. 默认顺序类型: vector `顺序存储` + `动态分配整块存储区`
  2. 频繁随机插入: list `节点存储` + `动态分配单个存储节点`
  3. 频繁头尾插入: deque
  4. 兼容C的内存布局: vector
  5. @todo 后面补充
  
## 关于容器无关代码

  
## 轻量拷贝

  ``` c
  class c_widget
  {
  public:
      c_widget (const c_widget&);           // constructor
      c_widget& operator= (const c_widget&); // operator override
  }
  ```

  ``` c
  vector<c_widget> w_obj_vec;
  class c_special_widget;       // derived from class c_widget
  c_special_widget sw_obj;
  w_obj_vec.push_back(sw_obj);  // @note some extra part lost
  ```
