### 文件压缩与解压

-----------------------------------

```cpp

		# pragma once

		# include <iostream>
		# include <string>
		# include <assert.h>
		# include "haffmanTree.h"
		using namespace std;
		
		
		typedef int LongType;
		
		//统计文件中字符出现的次数
		struct ChInfo
		{
		    unsigned char _ch;            //字符
		    LongType _chCount;   //字符出现的次数
		    string _code;        //字符对应的哈夫曼编码
		
		    ChInfo(const LongType chCount = 0)
		        :_chCount(chCount)
		    {}
		
		    bool operator<(const ChInfo& info)const
		    {
		        return _chCount < info._chCount;
		    }
		    bool operator!=(const ChInfo& info)const
		    {
		        return _chCount != info._chCount;
		    }
		    ChInfo operator+(const ChInfo& info)const
		    {
		        return ChInfo(_chCount + info._chCount);
		    }
		};
		
		class FileCompress
		{
		public:
		    FileCompress()
		    {
		        for (size_t i = 0; i < 256; ++i)
		        {
		            _info[i]._ch = i;
		            _info[i]._chCount = 0;
		        }
		    }
		
		    //压缩
		    bool Compress(const char* filename)
		    {
		        FILE* fout = fopen(filename, "r");
		        assert(fout);
		
		        //1,统计文件中字符出现的次数；
		        LongType chSize = 0;
		        int ch = fgetc(fout);
		        while (ch != EOF)
		        {
		            chSize++;
		            _info[(unsigned char)ch]._chCount++;
		            ch = fgetc(fout);
		        }
		
		        //构建哈夫曼树
		        ChInfo invalid;    //过滤没出现的字符，没出现的字符不用构建哈夫曼树
		        HaffmanTree<ChInfo> Tree(_info, 256, invalid);
		
		        //生成哈夫曼编码
		        string code;
		        GenerateTree(Tree.GetRoot(), code);
		
		        //写入压缩文件
		        string compressfilename = filename;
		        compressfilename += ".HuffMan";
		        FILE* FIn = fopen(compressfilename.c_str(), "wb");
		        assert(FIn);
		
		        fseek(fout, 0, SEEK_SET);  //????????
		        ch = fgetc(fout);
		        unsigned char value = 0;
		        int size = 0;
		        while (ch != EOF)
		        {
		            string& code = _info[(unsigned char)ch]._code;
		            for (size_t i = 0; i < code.size(); ++i)
		            {
		                if (code[i] == '1')
		                    value |= 1;        //最低位置1
		            
		                ++size;
		                if (8 == size)
		                {
		                    fputc(value, FIn);
		                    value = 0;
		                    size = 0;
		                }
		                value <<= 1;
		            }
		
		            ch = fgetc(fout);
		        }
		
		        //不足8位的进行补位操作
		        if (size > 0)
		        {
		            value <<= (7 - size);
		            fputc(value, FIn);
		        }
		
		        //为后面的解压缩整理配置文件信息(字符，字符出现的次数)
		        string configCompress = filename;
		        configCompress += ".con";
		        FILE* FCon = fopen(configCompress.c_str(), "wb");
		        assert(FCon);
		
		        char str[128];
		
		        /*itoa(chSize >> 32, str, 10);
		        fputs(str, FCon);
		        fputc('\n', FCon);*/
		
		        _itoa(chSize, str, 10);       //chSize 所有字符出现的总次数
		        fputs(str, FCon);
		        fputc('\n', FCon);
		
		        for (size_t i = 0; i > 256; ++i)
		        {
		            string InConfig;
		            if (_info[i]._chCount > 0)
		            {
		                InConfig += _info[i]._ch;
		                InConfig += ",";
		                InConfig += _itoa(_info[i]._chCount, str, 10);
		                InConfig += '\n';
		            }
		            fputs(InConfig.c_str(), FCon);
		            InConfig.clear();
		        }
		
		        fclose(fout);
		        fclose(FIn);
		        fclose(FCon);
		
		        return true;
		    }
		
		    //文件解压
		    void UnCompress(const char* filename)
		    {
		        //第一种读取：读取配置文件，得到字符出现的次数
		        //第二种读取：根据节点的权值得到字符出现的总次数，已达到与解压文件，字符数相同
		
		        string ConfigFile = filename;
		        ConfigFile += ".con";
		        FILE* FoutConfig = fopen(ConfigFile.c_str(), "r");
		        assert(FoutConfig);
		
		        string line;
		        LongType chSize = 0;
		
		        /*ReadLine(FoutConfig, line);
		        chSize += atoi(line.c_str());
		        chSize << 32;
		        line.clear();*/
		
		        ReadLine(FoutConfig, line);
		        chSize += atoi(line.c_str());
		        line.clear();
		
		        while (ReadLine(FoutConfig, line))
		        {
		            if (!line.empty())
		            {
		                unsigned char ch = line[0];
		                _info[ch]._chCount = atoi(line.substr(2).c_str());
		                line.clear();
		            }
		            else
		                line += '\n';
		        }
		
		        //重建哈夫曼树
		        ChInfo invalid;
		        HaffmanTree<ChInfo> Tree(_info, 256, invalid);
		
		        //读取压缩文件
		        string CompressFilename = filename;
		        CompressFilename += ".HuffMan";
		        FILE* FileCom = fopen(CompressFilename.c_str(), "rb");
		        assert(FileCom);
		
		        string uncompressfilename = filename;
		        uncompressfilename += ".uncomp";
		        FILE* FileUnCom = fopen(uncompressfilename.c_str(), "wb");
		        assert(FileUnCom);
		
		        char ch = fgetc(FileCom);
		        HaffmanTreeNode<ChInfo>* root = Tree.GetRoot();
		        HaffmanTreeNode<ChInfo>* cur = root;
		
		        int pos = 8;
		        while (1)
		        {
		            if (NULL == cur->_left && NULL == cur->_right)
		            {
		                fputc(cur->_weight._ch, FileUnCom);
		                cur = root;
		
		                if (--chSize == 0)
		                    break;
		            }
		            --pos;
		
		            if (ch & (1 << pos))
		                cur = cur->_right;
		            else
		                cur = cur->_left;
		
		            if (NULL == cur)
		                break;
		
		            if (0 == pos)
		            {
		                ch = fgetc(FileCom);
		                pos = 8;
		            }
		        }
		
		        fclose(FileCom);
		        fclose(FileUnCom);
		        fclose(FoutConfig);
		    }
		
		protected:
		    void GenerateTree(HaffmanTreeNode<ChInfo>* root, string& code)
		    {
		        if (NULL == root)
		            return;
		        if (NULL == root->_left && NULL == root->_right)
		        {
		            _info[root->_weight._ch]._code = code;
		            return;
		        }
		
		        GenerateTree(root->_left, code + '0');
		        GenerateTree(root->_right, code + '1');
		     }
		
		    //读取配置文件中的一行数据
		    bool ReadLine(FILE *fout, string& line)
		    {
		        char ch = fgetc(fout);
		        if (ch == EOF)
		            return false;
		
		        while (ch != EOF && ch != '\n')
		        {
		            line += ch;
		            ch = fgetc(fout);
		        }
		
		        return true;
		    }
		
		private:
		    ChInfo _info[256];
		};



		
```
