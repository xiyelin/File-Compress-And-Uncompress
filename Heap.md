### 二叉堆的实现

-------------------------------------------------------

```cpp


		# include <iostream>
        using namespace std;
        # include <vector>
        # include <assert.h>

        //小堆
        struct Less
        {
        public:
            bool operator()(const int& l, const int& r)
            {
                return l < r;
            }
        };
        //大堆
        struct Greater
        {
        public:
            bool operator()(const int& l, const int& r)
            {
                return l > r;
            }
        };

        template<class Compare>      //仿函数实现比较器
        class Heap
        {
        public:
            Heap()
            {}
            Heap(int *str, int size)
            {
                for (int i = 0; i < size; ++i)
                {
                    _array.push_back(str[i]);
                }

                int begin = size / 2 - 1;
                for (; begin >= 0; --begin)
                {
                    _AdjustDown(begin);
                }
            }
            Heap(vector<int> str)
            {
                _array.swap(str);

                int begin = (_array.size() / 2) - 1;
                for (; begin >= 0; --begin)
                {
                    _AdjustDown(begin);
                }
            }

            void Insert(const int x)
            {
                _array.push_back(x);
                int size = _array.size() - 1;
                _AdjustUp(size);
            }

            void Remove()
            {
                _array[0] = _array[_array.size() - 1];
                _array.pop_back();

                _AdjustDown(0);
            }

            const int Top()
            {
                return _array[0];
            }

            bool Empty()
            {
                return _array.empty();
            }
            
            size_t Size()
            {
            	return _array.size();
            }

        protected:
            void _AdjustDown(int root)             //向下调整
            {
                int size = _array.size();

                while (root < size)
                {
                    int left = root * 2 + 1;
                    int right = left + 1;
                    int key = left;
                    if (right < size)
                    {
                        key = Compare()(_array[left], _array[right]) ? left : right;
                    }

                    if (key < size && Compare()(_array[key], _array[root]))
                    {
                        swap(_array[root], _array[key]);
                        root = key;                                    //注意要调整完
                    }
                    else
                        break;
                }
            }

            void _AdjustUp(int child)                                  //向上调整
            {
                while (child >= 0)
                {
                    int root = (child - 1) / 2;

                    if (Compare()(_array[child], _array[root]))
                    {
                        swap(_array[root], _array[child]);
                        child = root;
                    }
                    else
                        break;
                }
            }

        private:
            vector<int> _array;
        };
        


```




