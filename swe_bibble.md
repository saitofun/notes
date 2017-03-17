```
#include "swe_vector.h"
#include <string.h> /* sample中使用了 memset */
#include <stdlib.h> /* sample中使用了 rand */

// swe_vector_sample.cpp

typedef struct{
    int     a;
    void*   b;
    int     c[10];
}swe_vector_stru;


void vector_sample_basic()
{
    // 定义一个vector类型
    DEFINE_VECTOR(swe_vector_stru, vec_a);
    swe_vector_stru st_a, st_b;
    unsigned int u_size;

    st_a.a = 1;
    st_a.b = &st_a;
    memset(st_a.c, 0x1, 10*sizeof(int));

    // 新增一个元素
    VECTOR_ADD(vec_a, st_a);

    // 获得元素个数
    u_size = VECTOR_SIZE(vec_a);

    // 访问第0个元素
    st_b = VECTOR_GET(vec_a, 0);

    st_b.b = &st_b;
    // 设置第0个元素
    VECTOR_SET(vec_a,0,st_b);

    // 新增一个元素
    VECTOR_ADD(vec_a, st_a);

    // 删除第0个元素
    VECTOR_DEL(vec_a,0);
}

/* 查找时使用的查找目标函数定义                                         */
DEFINE_VECTOR_FIND_TARGET_FUNC_BEGIN(find_stru_b_is_not_null, swe_vector_stru, p)
    if(p.b != NULL)
        return 1;
    else
        return 0;
DEFINE_VECTOR_FIND_TARGET_FUNC_END

DEFINE_VECTOR_FIND_TARGET_FUNC_BEGIN(find_stru_c_sum_is_greater_than_100, swe_vector_stru, p)
    int sum = 0;
    
    for(int i=0; i< 10; i++)
    {
        sum += p.c[i];
    }

    if(sum>100)
        return 1;
    else
        return 0;
DEFINE_VECTOR_FIND_TARGET_FUNC_END

void vector_sample_find()
{
    // 定义一个vector类型
    DEFINE_VECTOR(swe_vector_stru, vec_a);
    swe_vector_stru st_a;
    unsigned int u_id;

    for(int i=0; i<100; i++)
    {
        st_a.a = i;
        st_a.b = &st_a;
        memset(st_a.c, i, 10*sizeof(int));

        VECTOR_ADD(vec_a, st_a);
    }

    // 仅查找一次
    u_id = VECTOR_FIND(vec_a, find_stru_b_is_not_null);
    if(u_id == VECTOR_SIZE(vec_a))
    {
        printf("can not find element meets target function using comp func %s\n", "find_stru_b_is_not_null");
    }
    else
    {
        printf("find element meets target function, the element index is %u \n", u_id);
    }
    
    u_id = VECTOR_FIND(vec_a, find_stru_c_sum_is_greater_than_100);
    if(u_id == VECTOR_SIZE(vec_a))
    {
        printf("can not find element meets target function using comp func %s\n", "find_stru_c_sum_is_greater_than_100");
    }
    else
    {
        printf("find element meets target function, the element index is %u \n", u_id);
    }

    // 多次查找
    u_id = 0;

    while(true)
    {
        u_id = VECTOR_FIND_FROM(vec_a, u_id, find_stru_b_is_not_null);

        if(u_id == VECTOR_SIZE(vec_a))
        {
            printf("can not find element meets target function using comp func %s\n", "find_stru_b_is_not_null");
            break;
        }
        else
        {
            printf("find element meets target function, the element index is %u \n", u_id);

            // 从下一个位置开始再次搜索
            u_id++;
        }

    }
}

void vector_sample_count()
{
    // 定义一个vector类型
    DEFINE_VECTOR(swe_vector_stru, vec_a);
    swe_vector_stru st_a;
    unsigned int u_size;

    for(int i=0; i<100; i++)
    {
        st_a.a = i;
        st_a.b = &st_a;
        memset(st_a.c, i, 10*sizeof(int));

        VECTOR_ADD(vec_a, st_a);
    }

    u_size = VECTOR_COUNT(vec_a, find_stru_c_sum_is_greater_than_100);

    printf("find %u elements meets %s", u_size, "find_stru_c_sum_is_greater_than_100");
}

void vector_sample_del_target()
{
    // 定义一个vector类型
    DEFINE_VECTOR(swe_vector_stru, vec_a);
    swe_vector_stru st_a;
    unsigned int u_id, u_size;

    for(int i=0; i<100; i++)
    {
        st_a.a = i;
        st_a.b = &st_a;
        memset(st_a.c, i, 10*sizeof(int));

        VECTOR_ADD(vec_a, st_a);
    }

    // 删除指定的元素
    VECTOR_DEL_TARGET(vec_a,find_stru_c_sum_is_greater_than_100);

    // 再查找应该查找不到了
    u_id = VECTOR_FIND(vec_a, find_stru_c_sum_is_greater_than_100);
    if(u_id == VECTOR_SIZE(vec_a))
    {
        printf("can not find element meets target function using comp func %s\n", "find_stru_c_sum_is_greater_than_100");
    }
    else
    {
        printf("error! find element meets target function, the element index is %u \n", u_id);
    }

    u_size = VECTOR_COUNT(vec_a, find_stru_c_sum_is_greater_than_100);

    printf("find %u elements meets %s", u_size, "find_stru_c_sum_is_greater_than_100");

}


DEFINE_VECTOR_SORT_COMP_FUNC_BEGIN(compare_less_than_stru_a,swe_vector_stru,p1,p2)
    if(p1.a < p2.a)
        return 1;
    else
        return 0;
DEFINE_VECTOR_SORT_COMP_FUNC_END

void vector_sample_sort()
{
    // 定义一个vector类型
    DEFINE_VECTOR(swe_vector_stru, vec_a);
    swe_vector_stru st_a;
    unsigned int u_id;

    for(int i=0; i<10; i++)
    {
        st_a.a = rand();
        st_a.b = &st_a;
        memset(st_a.c, i, 10*sizeof(int));

        VECTOR_ADD(vec_a, st_a);
    }

    for(int i=0; i<10; i++)
        printf("index: %2d   value: %d \n", i, VECTOR_GET(vec_a,i).a);

    VECTOR_SORT(vec_a, compare_less_than_stru_a);

    for(int i=0; i<10; i++)
        printf("index: %2d   value: %d \n", i, VECTOR_GET(vec_a,i).a);

    u_id = VECTOR_MAX(vec_a, compare_less_than_stru_a);

    printf("max id: %d\n", u_id);

    u_id = VECTOR_MIN(vec_a, compare_less_than_stru_a);

    printf("min id: %d\n", u_id);
}

DEFINE_VECTOR_SEARCH_COMP_FUNC_BEGIN(compare_equal_stru, swe_vector_stru, p1, p2)

    if(memcmp(&p1, &p2, sizeof(swe_vector_stru))==0)
        return 1;
    else
        return 0;

DEFINE_VECTOR_SEARCH_COMP_FUNC_END


void vector_sample_search_sequence()
{
    // 定义vector类型
    DEFINE_VECTOR(swe_vector_stru, vec_a);
    DEFINE_VECTOR(swe_vector_stru, vec_b);

    swe_vector_stru st_a;
    unsigned int u_id;

    for(int i=0; i<10; i++)
    {
        st_a.a = rand();
        st_a.b = &st_a;
        memset(st_a.c, i, 10*sizeof(int));

        VECTOR_ADD(vec_a, st_a);
    }

    for(int i=0; i<10; i++)
    {
        st_a.a = rand();
        st_a.b = &st_a;
        memset(st_a.c, i, 10*sizeof(int));

        VECTOR_ADD(vec_a, st_a);
        VECTOR_ADD(vec_b, st_a);
    }

    // vector a中包含vector b
    u_id = VECTOR_SEARCH_SEQUENCE(vec_a, vec_b, compare_equal_stru);

    if(u_id == VECTOR_SIZE(vec_a))
    {
        printf("vec_a does not contain vec b\n");
    }
    else
    {
        printf("vec_a begin at %u that contains vec_b\n", u_id);
    }

    VECTOR_SET(vec_b, 1, st_a);

    // vector a中不包含vector b
    u_id = VECTOR_SEARCH_SEQUENCE(vec_a, vec_b, compare_equal_stru);

    if(u_id == VECTOR_SIZE(vec_a))
    {
        printf("vec_a does not contain vec b\n");
    }
    else
    {
        printf("vec_a begin at %u that contains vec_b\n", u_id);
    }

    VECTOR_CLEAR(vec_a);

    VECTOR_MERGE(vec_a, vec_b);
    VECTOR_MERGE(vec_a, vec_b);
    VECTOR_MERGE(vec_a, vec_b);

    // vector a中包含3段 vector b
    u_id = VECTOR_SEARCH_SEQUENCE_FROM(vec_a, vec_b, VECTOR_SIZE(vec_b),compare_equal_stru);

    if(u_id == VECTOR_SIZE(vec_a))
    {
        printf("vec_a does not contain vec b\n");
    }
    else
    {
        printf("vec_a begin at %u that contains vec_b\n", u_id);
    }


}


typedef struct  
{
    char    name[256];
    int     id;
}person;

DEFINE_VECTOR_SEARCH_COMP_FUNC_BEGIN(compare_equal_person,person,p1,p2)

    if(strcmp(p1.name, p2.name) == 0)
        return 1;
    else
        return 0;

DEFINE_VECTOR_SEARCH_COMP_FUNC_END

void vector_sample_search_any()
{
    DEFINE_VECTOR(person, med);
    DEFINE_VECTOR(person, female);

    person p;

    strcpy(p.name, "xieting");
    VECTOR_ADD(med, p);
    strcpy(p.name, "chenchaoqun");
    VECTOR_ADD(med, p);
    strcpy(p.name, "fuying");
    VECTOR_ADD(med, p);
    strcpy(p.name, "gaozhendong");
    VECTOR_ADD(med, p);
    strcpy(p.name, "guzhengming");
    VECTOR_ADD(med, p);
    strcpy(p.name, "liwenhai");
    VECTOR_ADD(med, p);
    strcpy(p.name, "shaodan");
    VECTOR_ADD(med, p);
    strcpy(p.name, "wangguilin");
    VECTOR_ADD(med, p);


    strcpy(p.name, "xieting");
    VECTOR_ADD(female, p);
    strcpy(p.name, "zhanleilei");
    VECTOR_ADD(female, p);

    unsigned int u_id;

    u_id = VECTOR_SEARCH_ANY_FROM(med, female, 1, compare_equal_person);

    if(u_id == VECTOR_SIZE(med))
    {
        printf("med from 1 does not contain female b\n");
    }
    else
    {
        printf("med have female at %u \n", u_id);
    }
}
```

```
#ifndef _SWE_VECTOR_
#define _SWE_VECTOR_

#include <vector>
#include <algorithm>

using namespace std;

//  vector  是一个顺序存储容器,  可以随时插入、删除元素,  可以通过下标访问元素
//  可以认为是一个带链表功能的数组
//  参见swe_vector_sample.cpp使用方法

/************************************************************************/
/* 接口列表                                                             */
/************************************************************************/
/*

    DEFINE_VECTOR               定义一个vector
    DECLARE_VECTOR              声明一个vector

    - 基本操作 -

    VECTOR_ADD                  向vector中添加一个元素
    VECTOR_GET                  获得vector中一个元素
    VECTOR_SET                  设置vector中一个元素
    VECTOR_MERGE                合并另一个vector
    VECTOR_SIZE                 返回vector中的元素个数
    VECTOR_CLEAR                清空vector
    
    - 查找 -

    VECTOR_FIND                 查找vector中的元素
    VECTOR_FIND_FROM            从某个位置开始查找vector中的元素
    VECTOR_COUNT                统计vector中的满足特定条件的元素
    
    - 删除 -

    VECTOR_DEL                  从vector中删除一个元素
    VECTOR_DEL_TARGET           从vector中的删除满足特定条件的元素
    
    - 排序 -

    VECTOR_SORT                 对vector中的元素进行排序(稳定排序)
    VECTOR_MAX(MIN)             获取vector中的最大(最小)元素的索引号
    
    - 搜索 -

    VECTOR_SEARCH_SEQUENCE      在vector变量1中搜索vector变量2(匹配序列)
    VECTOR_SEARCH_SEQUENCE_FROM 从指定位置开始在vector变量1中搜索vector变量2(匹配序列)
    VECTOR_SEARCH_ANY           在vector变量1中搜索包含在vector变量2中的元素
    VECTOR_SEARCH_ANY_FROM      从指定位置开始在vector变量1中搜索包含在vector变量2中的元素
*/

/************************************************************************/
/* 接口实现                                                             */
/************************************************************************/

/* 功能说明:

    定义一个vector

   参数说明:
    
    TYPE    - vector元素类型
    NAME    - vector变量名称

   附加说明:

    可以是全局变量,也可以是局部变量
*/
#define DEFINE_VECTOR(TYPE,NAME)       vector<TYPE>NAME

/* 功能说明:

    声明一个vector

   参数说明:
    
    TYPE    - vector元素类型
    NAME    - vector变量名称 
*/
#define DECLARE_VECTOR(TYPE,NAME)       extern vector<TYPE>NAME

/* 功能说明:

    向vector中添加一个元素

   参数说明:
    
    NAME    - vector变量名称
    ITEM    - 希望添加的元素

   返回值:
    
    无

   附加说明:

    新添加的元素位于vector的末尾
*/
#define VECTOR_ADD(NAME,ITEM)           NAME.push_back(ITEM)

/* 功能说明:

    获得vector中一个元素

   参数说明:
    
    NAME    - vector变量名称
    INDEX   - 希望访问的元素下标

   返回值:

    vector中的元素

   附加说明:

    下标从0开始, 一直到N-1, N为vector中元素总数.
    不可访问不存在的元素, 当vector为空时, 0位置也不可访问
*/
#define VECTOR_GET(NAME,INDEX)          NAME[(unsigned int)INDEX]


/* 功能说明:

    设置vector中一个元素

   参数说明:
    
    NAME    - vector变量名称
    INDEX   - 希望访问的元素下标
    ITEM    - 希望设置的值

   返回值:
    
    无

   附加说明:

    下标从0开始, 一直到N-1, N为vector中元素总数.
    不可访问不存在的元素, 当vector为空时, 0位置也不可访问
*/
#define VECTOR_SET(NAME,INDEX,ITEM)     NAME[INDEX]=(ITEM)


/* 功能说明:

    合并另一个vector

   参数说明:
    
    NAME1   - vector变量1名称
    NAME2   - vector变量2名称

   返回值:
    
    无

   附加说明:

    将vector变量2中的元素合并到vector变量1中

*/
#define VECTOR_MERGE(NAME1,NAME2)       NAME1.insert(NAME1.end(),NAME2.begin(),NAME2.end())


/* 功能说明:

    返回vector中的元素个数

   参数说明:
    
    NAME    - vector变量名称

   返回值:

    无

   附加说明:

*/
#define VECTOR_SIZE(NAME)               (unsigned int)NAME.size()


/* 功能说明:

    清空vector

   参数说明:
    
    NAME    - vector变量名称

   返回值:

    无

   附加说明:

*/
#define VECTOR_CLEAR(NAME)                  NAME.clear()


/* 功能说明:

    定义查找中使用的回调函数(VECTOR_FIND中的TARGET)

   参数说明:
    
    FUNC_NAME   - 回调函数名称
    TYPE        - 定义vector时使用的类型
    PARA_NAME   - 回调函数参数名称(注: TYPE PARA_NAME 组合为回调函数参数)

   返回值:

    在函数内部实现时, 若PARA1 == 预期条件, 则返回1, 否则返回0

   附加说明:
    
    (1) 需要和DEFINE_VECTOR_FIND_TARGET_FUNC_END配对使用, 参考sample代码
    (2) 若需声明, 则使用DECLARE_VECTOR_FIND_TARGET_FUNC

   实现模板:

   DEFINE_VECTOR_FIND_TARGET_FUNC_BEGIN(xxx_find_is_not_null, yyy_type, p)
    
        if(p.zzz != NULL)
            return 1;
        else
            return 0;

   DEFINE_VECTOR_FIND_TARGET_FUNC_END

*/
#define DEFINE_VECTOR_FIND_TARGET_FUNC_BEGIN(FUNC_NAME,TYPE,PARA_NAME)  \
                                        bool FUNC_NAME(const TYPE& PARA_NAME) {

#define DEFINE_VECTOR_FIND_TARGET_FUNC_END                              \
                                        }

#define DECLARE_VECTOR_FIND_TARGET_FUNC(FUNC_NAME,TYPE,PARA_NAME)  \
                                        extern bool FUNC_NAME(const TYPE& PARA_NAME)


/* 功能说明:

    查找vector中的元素

   参数说明:
    
    NAME    - vector变量名称
    TARGET  - 查找的目标函数(由DEFINE_VECTOR_FIND_TARGET_FUNC定义)

   返回值:

    若找到,     则返回 0 至 N-1 间的一个值(N为vector的size)
    若未找到,   则返回 N

   附加说明:

*/
#define VECTOR_FIND(NAME,TARGET)        (unsigned int)(find_if(NAME.begin(), NAME.end(), TARGET) - NAME.begin())

/* 功能说明:

    从某个位置开始查找vector中的元素

   参数说明:
    
    NAME    - vector变量名称
    START   - 起始查找的位置(0到N-1, N为vector的size)
    TARGET  - 查找的目标函数(由DEFINE_VECTOR_FIND_TARGET_FUNC定义)

   返回值:

    若找到,     则返回 0 至 N-1 间的一个值(N为vector的size)
    若未找到,   则返回 N

   附加说明:

*/
#define VECTOR_FIND_FROM(NAME,START,TARGET)        \
                                        (unsigned int)(find_if(NAME.begin()+(size_t)START, NAME.end(), TARGET) - NAME.begin())

/* 功能说明:

    统计vector中的TARGET条件的元素

   参数说明:
    
    NAME    - vector变量名称
    TARGET  - 统计的目标函数(由DEFINE_VECTOR_FIND_TARGET_FUNC定义)

   返回值:

    符合条件的元素个数

   附加说明:

*/
#define VECTOR_COUNT(NAME,TARGET)       (unsigned int)count_if(NAME.begin(), NAME.end(), TARGET)


/* 功能说明:

    从vector中删除一个元素

   参数说明:
    
    NAME    - vector变量名称
    INDEX   - 希望删除的元素下标

   返回值:

    无

   附加说明:

    下标从0开始, 一直到N-1, N为vector中元素总数
*/
#define VECTOR_DEL(NAME,INDEX)          NAME.erase(NAME.begin()+INDEX)

/* 功能说明:

    从vector中的删除TARGET指定的元素

   参数说明:
    
    NAME    - vector变量名称
    TARGET  - 删除的目标函数(由DEFINE_VECTOR_FIND_TARGET_FUNC定义)

   返回值:

    无

   附加说明:

*/
#define VECTOR_DEL_TARGET(NAME,TARGET)  NAME.erase(remove_if(NAME.begin(), NAME.end(), TARGET),NAME.end())

/* 功能说明:

    定义排序中使用的回调函数(VECTOR_SORT中的COMP)

   参数说明:
    
    FUNC_NAME   - 回调函数名称
    TYPE        - 定义vector时使用的类型
    PARA_NAME1  - 回调函数参数1名称(注: TYPE PARA_NAME1 组合为回调函数参数1)
    PARA_NAME2  - 回调函数参数1名称(注: TYPE PARA_NAME1 组合为回调函数参数1)

   返回值:

    在函数内部实现时, 若PARA1 < PARA2, 则返回1, 否则返回0

   附加说明:
    
    (1) 需要和DEFINE_VECTOR_SORT_COMP_FUNC_END配对使用, 参考sample代码
    (2) 若需声明, 则使用DECLARE_VECTOR_SORT_COMP_FUNC
    (3) 这个函数定义两个参数的大小比较关系, 定义的关系为 <, 即满足 < 关系时, 返回1.

    实现模板:

    DEFINE_VECTOR_SORT_COMP_FUNC_BEGIN(xxx_compare_less_than, yyy_type, p1, p2)

        if(p1.zzz < p2.zzz)
            return 1;
        else
            return 0;
    
    DEFINE_VECTOR_SORT_COMP_FUNC_END


*/
#define DEFINE_VECTOR_SORT_COMP_FUNC_BEGIN(FUNC_NAME,TYPE,PARA_NAME1,PARA_NAME2)    \
                                        bool FUNC_NAME(const TYPE& PARA_NAME1,const TYPE& PARA_NAME2) {

#define DEFINE_VECTOR_SORT_COMP_FUNC_END                                            \
                                        }

#define DECLARE_VECTOR_SORT_COMP_FUNC(FUNC_NAME,TYPE,PARA_NAME1,PARA_NAME2)         \
                                        extern bool FUNC_NAME(const TYPE& PARA_NAME1,const TYPE& PARA_NAME2)

/* 功能说明:

    对vector中的元素进行排序(稳定排序)

   参数说明:
    
    NAME    - vector变量名称
    COMP    - 排序中使用的比较函数(由DEFINE_VECTOR_SORT_COMP_FUNC定义)

   返回值:

    无

   附加说明:

*/
#define VECTOR_SORT(NAME,COMP)          stable_sort(NAME.begin(), NAME.end(), COMP)


/* 功能说明:

    获取vector中的最大(最小)元素的索引号

   参数说明:
    
    NAME    - vector变量名称
    COMP    - 排序中使用的比较函数(由DEFINE_VECTOR_SORT_COMP_FUNC定义)

   返回值:

    元素位于vector中的Index

   附加说明:
    
    COMP 必须描述的是小于关系

*/
#define VECTOR_MAX(NAME,COMP)          (unsigned int)(max_element(NAME.begin(), NAME.end(), COMP) - NAME.begin())
#define VECTOR_MIN(NAME,COMP)          (unsigned int)(min_element(NAME.begin(), NAME.end(), COMP) - NAME.begin())


/* 功能说明:

    定义搜索中使用的回调函数(VECTOR_SEARCH中的COMP)

   参数说明:
    
    FUNC_NAME   - 回调函数名称
    TYPE        - 定义vector时使用的类型
    PARA_NAME1  - 回调函数参数1名称(注: TYPE PARA_NAME1 组合为回调函数参数1)
    PARA_NAME2  - 回调函数参数1名称(注: TYPE PARA_NAME1 组合为回调函数参数1)

   返回值:

    在函数内部实现时, 若PARA1 == PARA2, 则返回1, 否则返回0

   附加说明:
    
    (1) 需要和DEFINE_VECTOR_SEARCH_COMP_FUNC_END配对使用, 参考sample代码
    (2) 若需声明, 则使用DECLARE_VECTOR_SEARCH_COMP_FUNC
    (3) 这个函数定义两个参数的大小比较关系, 定义的关系为 ==, 即满足 == 关系时, 返回1.

    实现模板:

    DEFINE_VECTOR_SEARCH_COMP_FUNC_BEGIN(xxx_compare_less_than, yyy_type, p1, p2)

        if(p1.zzz == p2.zzz)
            return 1;
        else
            return 0;
    
    DEFINE_VECTOR_SEARCH_COMP_FUNC_END


*/
#define DEFINE_VECTOR_SEARCH_COMP_FUNC_BEGIN(FUNC_NAME,TYPE,PARA_NAME1,PARA_NAME2)    \
                                        bool FUNC_NAME(const TYPE& PARA_NAME1,const TYPE& PARA_NAME2) {

#define DEFINE_VECTOR_SEARCH_COMP_FUNC_END                                            \
                                        }

#define DECLARE_VECTOR_SEARCH_COMP_FUNC(FUNC_NAME,TYPE,PARA_NAME1,PARA_NAME2)         \
                                        extern bool FUNC_NAME(const TYPE& PARA_NAME1,const TYPE& PARA_NAME2)

/* 功能说明:

    在vector变量1中搜索vector变量2(匹配序列)

   参数说明:
    
    NAME1    - vector变量1名称
    NAME2    - vector变量2名称
    COMP    - 排序中使用的比较函数(由DEFINE_VECTOR_SEARCH_COMP_FUNC定义)

   返回值:

    若搜索成功, 则返回匹配序列的起始位置,取值范围0至N-1,N为vector变量1的长度
    若搜索失败, 则返回N

   附加说明:
    
    (1) COMP 必须描述的是等于关系
    (2) 这是一个序列搜索, 例如vector 1中的元素为{a, b, c, d, e, f, g},
        若vector 2中的元素是{c, d, e}, 则搜索结果返回值为2, 该值为vector 1中元素c的索引
        若vector 2中的元素是{c, e ,d}, 则搜索结果返回值为7, 该值为vector 1的size

*/
#define VECTOR_SEARCH_SEQUENCE(NAME1,NAME2,COMP) \
                                        (unsigned int)(search(NAME1.begin(), NAME1.end(), NAME2.begin(), NAME2.end(), COMP) - NAME1.begin())


/* 功能说明:

    从指定位置开始在vector变量1中搜索vector变量2(匹配序列)

   参数说明:
    
    NAME1   - vector变量1名称
    NAME2   - vector变量2名称
    START   - 起始搜索的位置(0到N-1, N为vector 1的size)
    COMP    - 排序中使用的比较函数(由DEFINE_VECTOR_SEARCH_COMP_FUNC定义)

   返回值:

    若搜索成功, 则返回匹配序列的起始位置,取值范围0至N-1,N为vector变量1的长度
    若搜索失败, 则返回N

   附加说明:
    
    (1) 参考VECTOR_SEARCH_SEQUENCE

*/
#define VECTOR_SEARCH_SEQUENCE_FROM(NAME1,NAME2,START,COMP) \
                                        (unsigned int)(search(NAME1.begin()+(size_t)START, NAME1.end(), NAME2.begin(), NAME2.end(), COMP) - NAME1.begin())


/* 功能说明:

    在vector变量1中搜索包含在vector变量2中的元素

   参数说明:
    
    NAME1    - vector变量1名称
    NAME2    - vector变量2名称
    COMP    - 排序中使用的比较函数(由DEFINE_VECTOR_SEARCH_COMP_FUNC定义)

   返回值:

    若搜索成功, 则返回匹配序列的起始位置,取值范围0至N-1,N为vector变量1的长度
    若搜索失败, 则返回N

   附加说明:
    
    (1) COMP 必须描述的是等于关系
    (2) 这是一个类似关键字匹配的搜索, 例如vector 1中的元素为{a, b, c, d, e, f, g},
        若vector 2中的元素是{c, f, g}, 则搜索结果返回值为2, 该值为vector 1中元素c的索引
        若vector 2中的元素是{z, f ,g}, 则搜索结果返回值为7, 该值为vector 1的size

*/
#define VECTOR_SEARCH_ANY(NAME1,NAME2,COMP) \
                                        (unsigned int)(find_first_of(NAME1.begin(), NAME1.end(), NAME2.begin(), NAME2.end(), COMP) - NAME1.begin())


/* 功能说明:

    从指定位置开始在vector变量1中搜索包含在vector变量2中的元素

   参数说明:
    
    NAME1   - vector变量1名称
    NAME2   - vector变量2名称
    START   - 起始搜索的位置(0到N-1, N为vector 1的size)
    COMP    - 排序中使用的比较函数(由DEFINE_VECTOR_SEARCH_COMP_FUNC定义)

   返回值:

    若搜索成功, 则返回匹配序列的起始位置,取值范围0至N-1,N为vector变量1的长度
    若搜索失败, 则返回N

   附加说明:
    
    (1) 参考VECTOR_SEARCH_SEQUENCE

*/
#define VECTOR_SEARCH_ANY_FROM(NAME1,NAME2,START,COMP) \
                                        (unsigned int)(find_first_of(NAME1.begin()+(size_t)START, NAME1.end(), NAME2.begin(), NAME2.end(), COMP) - NAME1.begin())




#endif /* _SWE_VECTOR_ */
```

```
// main.cpp

void vector_sample_basic();
void vector_sample_find();
void vector_sample_del_target();
void vector_sample_sort();
void vector_sample_count();
void vector_sample_search_sequence();
void vector_sample_search_any();

int main()
{
    vector_sample_basic();
    vector_sample_find();
    vector_sample_count();
    vector_sample_del_target();
    vector_sample_sort();
    vector_sample_search_sequence();
    vector_sample_search_any();

    return 0;
}
```
