### 哈夫曼树的实现

-------------------------------------------------------

```cpp

		# pragma once 

		# include "Heap.h"
		
		template<class T>
		struct HaffmanTreeNode
		{
		    HaffmanTreeNode<T>* _left;
		    HaffmanTreeNode<T>* _right;
		    T _weight;
		
		    HaffmanTreeNode(const T& weight)
		        :_left(NULL)
		        ,_right(NULL)
		        ,_weight(weight)
		    {}
		};
		
		template<class T>
		struct Compare
		{
		    bool operator()(HaffmanTreeNode<T>* l, HaffmanTreeNode<T>* r)
		    {
		        return l->_weight < r->_weight;
		    }
		};
		
		template<class T>
		class HaffmanTree
		{
		    typedef HaffmanTreeNode<T> Node;
		public:
		    HaffmanTree(const T* arr, size_t size, const T& invalid)
		    {
		        //建小堆
		        Heap<Node*, Compare<T>> minHeap;
		        for (size_t i = 0; i < size; ++i)
		        {
		            if (arr[i] != invalid)
		                minHeap.Insert(new Node(arr[i]));
		        }
		
		        //建哈夫曼树
		        while (minHeap.Size() > 1)
		        {
		            Node* left = minHeap.Top();
		            minHeap.Remove();
		            Node *right = minHeap.Top();
		            minHeap.Remove();
		            
		            Node *parent = new Node(left->_weight + right->_weight);
		            parent->_left = left;
		            parent->_right = right;
		
		            minHeap.Insert(parent);
		        }
		
		        _root = minHeap.Top();
		        minHeap.Remove();
		    }
		
		    Node* GetRoot()
		    {
		        return _root;
		    }
		
		private:
		    Node *_root;
		};
		
		
		
```

