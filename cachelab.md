# Cache lab

## PART I 

第一部分想让我们实现LRU算法实现的查询

模仿一个高速缓冲记忆体

LRU算法是我在操作系统里分页算法里面学过的 也就是

```
RU:最近最少使用(Least Recently Used).替换上次使用距离当前最远的页。根据局部性原理：替换最近最不可能 访问到的页。性能最接近OPT，但难以实现。可以维护一个关于访问页的栈或者给每个页添加最后访问的时间标签，但开销都很大。

 分析：（F表示页帧最初填满时出现page fault）

 a.需要页面2，内存中还有空闲位置，直接加入页面2

 b.需要页面3，内存中还有空闲位置，直接加入页面3

 c.需要页面2，内存中已经存在页面2，不加入任何页面，这里重置了2的最近历史访问记录

 d.需要页面1，内存中还有空闲位置，直接加入页面1

 e.需要页面5，页面2,3,1距离最近一次历史访问的距离依次为2,3,1（步骤c中重置了2的访问记录）.所以替换掉页        面3.

 f.需要页面2，内存中已经存在页面2，不加入任何页面，重置页面2的访问记录

 g.需要页面4，页面2,5,1距离最近一次历史访问的距离依次是1,2,3，所以替换掉页面1

 h.需要页面5，内存中存在页面5，不改变，重置页面5的访问记录

 i.需要页面3，页面2,5,4距离最近一次历史访问的距离依次为3,1,2，所以替换掉页面2

 j.需要页面2，页面3,5,4距离最近一次历史访问的距离依次为1,2,3，所以替换掉页面4

 k.需要页面5，内存中存在页面5，不改变

 L.需要页面2，内存中存在页面2，不改变
```

```
在csim.c中写一个Cache，使用LRU替换策略，以valgrind的memory trace作为输入，模拟Cache的hit和miss，输出hit、miss和eviction的总数。
```

感觉还是得再好好看书 ， 大概对照网上的代码抄了一遍。理解了实现的意思。

其中学到了很多

首先是处理输入格式

网上有很多种处理方法，但这一种是最简洁明了的。

其次是LRU的实现方案 这里开一个结构体的二维组， 每次都是先拓展出 2**s 次方 。 s为文件bit 。

然后进行比对。

其中visitCache 函数主要是以LRU 每次 更新的时候LRU 就会++ ，

以便后续进行类似时间戳的判断。

看起来很简单的代码构思比较麻烦。

之后在自己写一遍。

```
#include "cachelab.h"
#include <getopt.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <math.h>

typedef unsigned long int uint64_t;

typedef struct {
    int valid;
    int lru;
    uint64_t tag;
}cacheLine;

typedef cacheLine* cacheSet;
typedef cacheSet* Cache;

const char* usage = "Usage: %s [-hv] -s <s> -E <E> -b <b> -t <tracefile>\n";

int verbose = 0; //verbose flag 
int s;  //number of set index bits 
int E;  //number of lines per set
int b;  //number of block bits
FILE* fp = NULL;
Cache cache;

int hits = 0;
int misses = 0;
int evictions = 0;

void parseArgument(int argc, char* argv[]);
int visitCache(uint64_t address);
int simulate();

int main(int argc, char* argv[])
{
    parseArgument(argc, argv);
    simulate();
    printSummary(hits, misses, evictions);
    return 0;
}

void parseArgument(int argc, char* argv[])
{
    int opt;
    while ((opt = getopt(argc, argv, "hvs:E:b:t:")) != -1)
    {
        switch(opt)
        {
            case 'h':
                fprintf(stdout, usage, argv[0]);
                exit(1);
            case 'v':
                verbose = 1;
                break;
            case 's':
                s = atoi(optarg);
                break;
            case 'E':
                E = atoi(optarg);
                break;
            case 'b':
                b = atoi(optarg);
                break;
            case 't':
                fp = fopen(optarg, "r");
                break;
            default:
                fprintf(stdout, usage, argv[0]);
                exit(1);
        }
    }
}

int simulate()
{
    int S = pow(2, s);
    cache = (Cache)malloc(sizeof(cacheSet) * S);
    if (cache == NULL) return -1;
    for (int i = 0; i < S; i++)
    {
        cache[i] = (cacheSet)calloc(E, sizeof(cacheLine));
        if (cache[i] == NULL) return -1;
    }

    char buf[20];
    char operation;
    uint64_t address;
    int size;

    while (fgets(buf, sizeof(buf), fp) != NULL)
    {
        int ret;

        if (buf[0] == 'I') //ignore instruction cache accesses
        {
            continue;
        }
        else
        {
            sscanf(buf, " %c %lx,%d", &operation, &address, &size);

            switch (operation)
            {
                case 'S':
                    ret = visitCache(address);
                    break;
                case 'L':
                    ret = visitCache(address);
                    break;
                case 'M':
                    ret = visitCache(address);
                    hits++;
                    break;
            }

            if (verbose)
            {
                switch(ret)
                {
                    case 0:
                        printf("%c %lx,%d hit\n", operation, address, size);
                        break;
                    case 1:
                        printf("%c %lx,%d miss\n", operation, address, size);
                        break;
                    case 2:
                        printf("%c %lx,%d miss eviction\n", operation, address, size);
                        break;
                }
            }
        }
    }

    for (int i = 0; i < S; i++)
        free(cache[i]);
    free(cache);
    fclose(fp);
    return 0;
}

/*return value
      0             cache hit
      1             cache miss
      2             cache miss, eviction
*/
int visitCache(uint64_t address)
{
    uint64_t tag = address >> (s + b);
    unsigned int setIndex = address >> b & ((1 << s) - 1);

    int evict = 0;
    int empty = -1;
    cacheSet cacheset = cache[setIndex];

    for (int i = 0; i < E; i++)
    {
        if (cacheset[i].valid)
        {
            if (cacheset[i].tag == tag)
            {
                hits++;
                cacheset[i].lru = 1;
                return 0;
            }

            cacheset[i].lru++;
            if (cacheset[evict].lru <= cacheset[i].lru) // =是必须的,why?
            {
                evict = i;
            }
        }
        else
        {
            empty = i;
        }
    }

    //cache miss
    misses++;

    if (empty != -1)
    {
        cacheset[empty].valid = 1;
        cacheset[empty].tag = tag;
        cacheset[empty].lru = 1;
        return 1;
    }
    else
    {
        cacheset[evict].tag = tag;
        cacheset[evict].lru = 1;
        evictions++;
        return 2;
    }
}
```

