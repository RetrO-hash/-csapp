## Datalab

### 第一题

bitXor

only use ~ and | to make x&y

用~和| 来实现与运算、

比较简单离散学过了

```
int bitXor(int x, int y) { 
	  return ~(~(x&(~y))&(~((~x)&y)));
}
```

给了一个example样例

测一下

![image-20210207165558618](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207165558618.png)

-----------

## 第二题

tmin

返回最小的int值，也就是让他溢出1<<31 这样就是最小的int值了。

```
int tmin(void) {
	return 1<<31;
}
```

![image-20210207170655684](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207170655684.png)

-------------

## 第三题

判断x是否是 最大的int值

主要想法是溢出后*2为0 ，但同时满足的还有0

所以考虑排除0即可

```
int isTmax(int x) {
  int t=x+1+x+1;
  //printf("%d",!t);
  //printf("%d\n",q);
  int result=(!t)&(~(!(x+1)));
  return result;
}
```

![image-20210207174012116](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207174012116.png)

---------

## 第四题

```
int allOddBits(int x) {
	int format=0xAAAA;
	int qq=format+(format<<16);
  return !((x&qq)^qq);
}
```

为了判断是否只有奇数位上面是1，

那么就先搞个format 然后&一下 就只会比 format 的1个数还少而且1也都是奇数位的

然后再^下就知道是否和原来的format一样， 一样的话就代表只有奇数位有1 其他无。否则就是其他位也有。

--------------

## 第五题

```
int negate(int x) {
  int q=~x+1;
  return q;
}
```

取负数

不多说了

![image-20210207181126266](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207181126266.png)

-------

## 第六题

判断是否x在区间内

先找出Bit之间的规律。

然后找到了00111001 00110000

之间所有数

```
int isAsciiDigit(int x) {
	int a=3;
	int ss=(x>>6);
	int tt=(x>>4)&1;
	int ee=(x>>5)&1;
	int q=tt&ee;
	//printf("%d\n",q);
	int aa=(x>>3)&1;
	int bb=(x>>2)&1;
	int cc=(x>>1)&1;
	return (!ss)&q&((!aa)|aa&(!bb&(!cc)));
}
```

一个大判断.

![image-20210207182708851](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207182708851.png)

---------

## 第七题

实现三元运算符

经过查 发现任何数字&(-1)都是本身

那么通过这点可以构造

让一边为0另一边为自己然后或就可以了

```
int conditional(int x, int y, int z) {
	int mask = ~0 + !x;
	//printf("%d\n",!x);
	//printf("%d",mask); 
	return ((mask&y)|(~mask&z));
}
```

![image-20210207185728641](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207185728641.png)

------

## 第八题

可以想到x-y<0 这一点

来判断 大小

但是还要考虑溢出问题。。

调了很久。。。

通过>>31来判断符号

```
int isLessOrEqual(int x, int y) {
   int q=(~x+y+1)>>31;
   int a=(x>>31)&!(y>>31);
   //printf("%d\n",a);
   int b=!(x>>31)&(y>>31);
   return a|((!q)&(!b));
}
```

只有 x不是负数y是负数 或者 y>x 并且 x不是正数 y不是负数情况下返回1。（好像写反了） x>>31 为1 是负数

![image-20210207192041235](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207192041235.png)

--------------

## 第九题

实现！ 

非0则1 是0则0

```
int logicalNeg(int x) {
	return ~(~((x|~x+1)>>31))+1;
}
```

搞了一个贼奇怪的嵌套。。。

先是找了个办法让其有办法使得非0数和0区别出来即

~x+1|x后 >>31 必为 1 

而0却是 0

![image-20210207194540580](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207194540580.png)

------

## 第十题

查有多少位

u1s1我想枚举。

反正操作数肯定够

这里写下学的二分吧。。

```
int howManyBits(int x) {
	int sum=0;
	x=x^(x>>31);
	//printf("%d",x);
	sum+=(!!(x>>16)<<4);
	x=x>>((!!(x>>16)<<4));
//printf("%d",x);
	sum+=(!!(x>>8)<<3);
	x=x>>((!!(x>>8)<<3));
	//	printf("%d",x);
	sum+=(!!(x>>4)<<2);
	x=x>>((!!(x>>4)<<2));
	//	printf("%d",x);
	sum+=(!!(x>>2)<<1);
	x=x>>((!!(x>>2)<<1));
	//	printf("%d",x);
	sum+=(!!(x>>1));
	x=x>>(!!(x>>1));
	sum+=x; 
	return sum+1;
}
```

![image-20210207201430474](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207201430474.png)

-------

## 第十一题

浮点数运算

也就是×2

```
unsigned floatScale2(unsigned uf)
{
	int exp=(uf&0x7f800000)>>23;
	int sign=uf&(1<<31);
	if(!exp)
	return uf<<1|sign;
	if(exp==255)	
	return uf;
	exp++;
	if(exp==255)
	return 0x7f800000|sign;
	return (exp<<23)|(uf&0x807fffff);
}
```

分析下float如何存储

以及254 的时候是可以



![img](file:///C:\Users\Retr0\Documents\Tencent Files\2297571095\Image\C2C\Image1\0EB3AE29FD89CA3D5014C1C42F8388D2.jpg)

所以当exp为0的时候直接×2即可。若为255 则是NAN或者∞返回原值。

![20200911122948583](C:\Users\Retr0\Desktop\20200911122948583.png)

然后最后还原回去。就可以了。

![image-20210207234648579](C:\Users\Retr0\AppData\Roaming\Typora\typora-user-images\image-20210207234648579.png)

--------------

## 第十二题 （咕咕下）之后研究



-------------

## 第十三题

```
unsigned floatPower2(int x) {
	int exp=x+127;
	if(exp>=255) return (0xFF<<23);
	else if (exp<=0) return 0;
	else return exp<<23;
}
```

2.0^x。也就是直接+127 如果溢出还是覆盖1111 1111

如果小于等于0 就直接返回0

否则返回正常得exp。