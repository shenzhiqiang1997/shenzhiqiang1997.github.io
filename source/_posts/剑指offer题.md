---
title: 剑指offer题
date: 2018-03-15 01:17:15
tags:
    - 算法
    - 题目
categories:
    - 算法
---
* 剪绳子
给定一根长度为n的绳子，请把绳子剪成m段（m、n都是整数，n>1并且m>1），每段绳子的长度记为k[0],k[1],…,k[m]。请问k[0]* k[1] * … *k[m]可能的最大乘积是多少？
<!-- more -->
    ```java
    //动态规划
    public int maxProductAfterCutting_soultion1(int length){
        if(length<2)    return 0;
        if(length==2)   return 1;
        if(length==3)   return 2;
    
        int[] products=new products[length+1];
        //存储前3段的长度 分别为绳子为0 1 2 3长时绳子长度 便于计算
        products[0]=0;
        products[1]=1;
        products[2]=2;
        products[3]=3;
    
        //从4开始计算f(4)并存放在[4]处 后面的计算只需要考虑绳子分割成两段的每种情况
        //每一段的最大值f(j)已经计算好并存在了[j]中 只需要把[j]与[i-j]相乘 再把每种分割情况的乘积值进行相比 如果最大则放入[i]即得到f(i)
        for(int i=4;i<length;i++){
            int max=0;
            //最多把一个绳子平分 另一段利用i-j直接得到
            for(int j=1;j<=i/2;j++){
                int product=products[j]*products[i-j];
                if(product>max) max=product;
            }
            //最后把最大值放到[i]中
            products[i]=max;
        }
    
        //最终f(length)就是[length]
        return products[length];
    }
    //贪婪算法
    public int maxProductAfterCutting_soultion2(int length){
        if(length<2)    return 0;
        if(length==2)   return 1;
        if(length==3)   return 2;
    
        //如果length>=5 尽可能多的按3切段
        int numOfThree=length/3;
    
        //如果最后可以剩余4 则该长度为4的部分不能切段成3
        if(length-numOfThree*3==1)
            numOfThree-=1;
    
        //如果剩余4则按2切段
        int numOfTwo=(length-numOfThree*3)/2;
    
        //最后把这些部分相乘
        return (int)Math.pow(3,numOfThree)*(int)Math.pow(2,numOfTwo);
    
    }
    ```
* 打印从1到最大的N位数
输入数字n，按顺序打印出从1最大的n位十进制数。比如输入3，则打印出1、2、3一直到最大的3位数即999。
    ```java
    //方法1 直接处理字符数组 每次加一并输出
    public void printN(int n){
        if(n<=0)
            throw new Excpetion();
    
        char[] string=new char[n];
        
        for(int i=0;i<n;i++)
            string[i]='0';
    
        while(!increase(string))
            print(string);
    }
    
    //方法2 从最低位到最高位递归的安排数字 知道最高位被安排过之后进行输出
    //可把输出的数当做每位从0到9的全排列 只不过不输出最前面的0
    public void printN(int n){
        if(n<=0)
            throw new Excpetion();
    
        char[] string=new char[n];
        
        for(int i=0;i<n;i++)
            string[i]='0';
    
        //为最低位安排数字
        for(int i=0;i<10;i++)
            assignDigit(string,'0'+i,0);
    }
    
    public void assignDigit(char[] string,char digit,int index){
        //为第index位为安排数字
        string[index]=digit;
        
        //如果最后一位已经被安排过了 那么就进行输出 并且返回
        if(index==string.length-1){
            print(string);
            return;
        }
    
        //为下一位安排数字
        for(int i=0;i<10;i++)
            assignDigit(string,'0'+i,index+1);
    }
    
    
    public boolean increase(char[] string){
        int tokenOver=0;
    
        int sum=0;
        for(int i=srting.length-1;i>=0;i--){
            sum=string[i]-'0'+tokenOver;
    
            //最后一位加一
            if(i==string.length-1)
                sum+=1;
    
            //如果加了一之后大于10要产生进位
            if(sum>=10){
                //如果是最高位产生进位 则直接返回true 不能继续加1
                if(i==0){
                    return true;
                }else{
                    //否则减去10之后 之后把数加到相应位置 后再把tokenOver设置为1 作为进位产生的加数
                    string[i]=(sum-10)+'0';
                    tokenOver=1;
                }
            }else{
                //否则不再有进位 直接把结果加到相应位 返回false说明最高位没有进位 还可以继续加1
                string[i]=sum+'0';
                return false;
            }
    
        }
    }
    
    public void print(char[] string){
        boolean beginWithZero=true;
    
        for(int i=string.length-1;i>=0;i--){
            if(beginWithZero==true&&string[i]!='0')
                beginWithZero=false;
    
            if(!beginWithZero)
                System.out.print(string[i]);
        }
    
        System.out.println();
    
    }
    ```
* 在O(1)时间内删除链表节点
给定单向链表的头指针和一个节点指针 定义一个函数在O(1)时间内删除该节点
    ```java
    class ListNode{
    int value;
    ListNode next;
    }
    
    public void deleteNode(ListNode head,ListNode toBeDeleted){
        if(head==null||toBeDeleted==null)
            return;
    
        //如果不是尾节点
        if(toBeDeleted.next!=null){
            ListNode toBeDeletedNext=toBeDeleted.next;
            toBeDeleted.value=toBeDeletedNext.value;
            toBeDeleted.next=toBeDeletedNext.next;
            toBeDeletedNext=null;
        //如果是尾节点且是头结点
        }else if(head==toBeDeleted){
            head==null;
            toBeDeleted==null;
        //如果是尾节点但不是头结点 
        }else{
            ListNode node=head;
            
            //顺序地找到尾节点前一个节点
            while(node.next!=toBeDeleted){
                node=node.next;
            }
    
            node.next=null;
            toBeDeleted=null;
        }
    }
    ```
* 数字序列中某一位的数字
数字按照0123456789101112131415161718192021…的顺序排列。第5位（从0开始计数）为5，第13位为1，第19位为4…… 求任意第n位对应的数字
    ```java
    //方法1：累加位数和 直到超过所给下标找到所在数字 再在所在数据中寻找
    public class Solution{
        public int digitAtIndex(int index){
            //判断边界条件
            if (index<0)
                return  -1;
    
            int digitSum=0;
            for(int i=0;i<Integer.MAX_VALUE;i++){
                int digitLen=String.valueOf(i).length();
                digitSum+=digitLen;
                //如果位数累加和大于所给下标则当前数字就是下标所在的数字
                if(digitSum>index){
                    //进一步寻找下标位置的单个数字
                    //此时digitSum即当前数字的最后一个位置 使其回到第一个位置
                    int beginIndex=digitSum-digitLen;
                    //则index-digitSum就是在所找数字当前数字内部的位置
                    int preciseIndex=index-beginIndex;
                    //返回所找到的最终单个数字
                    return Integer.valueOf(String.valueOf(i).charAt(preciseIndex)+"");
                }
            }
            //如果失败返回-1
            return -1;
        }
    }
    
    //方法2：不断跳过前面位数段的数字 直到寻找到下标数字所在的位数段与位数段开头 
    //再在相应位数段里寻找具体下标的单个数字
    public class Solution{
        public int digitAtIndex(int index){
            //判断边界条件
            if (index<0)
                return  -1;
    
            //首先去寻找位数段 从位数为1开始寻找
            int digits=1;
    
            while(true){
                int digitsNumber=getNumOfDigits(digits);
    
                //直到定位到相应位数段 从相应位数段的第一个数字向后寻找下标数字
                if (index<digitsNumber*digits) {
                    return digitAtIndex(index,digits);
                }else {
                    //不断减少index 直到定位到相应位数段
                    index-=digitsNumber*digits;
                    digits++;
                }
    
            }
    
            //如果失败返回-1
    
        }
    
        //从相应数字段开头去查找下标数字
        public int digitAtIndex(int index,int digits){
            //获取到相应位数段的第一个数字
            int beginNumber=getBeginNumberOfDigits(digits);
            //下标所在的数字为第一个数字加上以位数段为开头的下标/位数
            int indexNumber=beginNumber+index/digits;
            //再根据余数来具体确定下标所在的单个数字
            int preciseIndex=index%digits;
    
            //返回最终找到的下标所在的单个数字
            return Integer.valueOf(String.valueOf(indexNumber).charAt(preciseIndex)+"");
        }
    
        public int getBeginNumberOfDigits(int digits){
            //下标为1比较特殊 返回0
            if (digits==1)
                return 0;
            else
                //否则为100···
                return power10(digits-1);
        }
    
        //获取相应位数的所有数字个数
        public int getNumOfDigits(int digits){
            //如果是1位数 比较特殊 是10个数字
            if (digits==1)
                return 10;
                //如果是其他位数 则从10···开始到99···为9*10的digist-1次方
            else
                return 9*power10(digits-1);
        }
    
        public int power10(int n){
            int result=1;
            for (int i=0;i<n ;i++ ) {
                result*=10;
            }
    
            return result;
        }
    }
    ```
* 把数字翻译成字符串
给定一个数字，按照如下规则翻译成字符串：0翻译成“a”，1翻译成“b”...25翻译成“z”。一个数字有多种翻译可能，例如12258一共有5种，分别是bccfi，bwfi，bczi，mcfi，mzi。实现一个函数，用来计算一个数字有多少种不同的翻译方法。
    ```java
    public class Solution{
        public int getTranslationCount(int number){
            //判断边界条件
            if (number<0)
                return 0;
    
            //将数字转化为字符数组 便于提取单个数字
            char[] numChars=String.valueOf(number).toCharArray();
            //数字的长度
            int len=numChars.length;
    
            //用于存放以下标为i的数字开头的数字的翻译数目
            int[] counts=new int[len];
    
    
            //从最后一个数字开始 向前统计以第i个数字开始的数字的翻译数目 并存储在对应的counts[i]中
            //递归公式f(i)=f(i+1)+g(i,i+1)*f(i+2); 当下标i的数字与后部分组合称为第一部分 以及 下标i和i+1的数字与后部分组合称为第二部分
            for (int i=len-1;i>=0;i--) {
                //用于记录以当前第i个数字为开头的数字的翻译数目
                int count=0;
    
                //首先统计第i个数字开头的第一部分的翻译数目
                //如果是最后一个数字 第一部分翻译数目为1 其他为以第i+1数字开头的翻译数目counts[i+1]
                if (i==len-1)
                    count=1;
                else
                    count=counts[i+1];
    
                //接来下统计第i个数字开头的第二部分的翻译数目
                //如果第i个数字不为最后一个数字 则说明第i个和第i+1个数字有可能被组合翻译
                if (i!=len-1) {
                    //继续查看第i个和第i+1个数字组合后是否在10-25之间
                    //如果是 说明这两个数字可以被组合翻译 需要加上第二部分的翻译数目
                    //否则说明这两个数字不可以被组合翻译 不存在第二部分的翻译
    
                    //得到第i个数字和第i+1个数字
                    char digit1=numChars[i];
                    char digit2=numChars[i+1];
                    //将这两个数字组合 看是否在10-25之间
                    int digit=Integer.valueOf(""+digit1+digit2);
    
                    //如果组合数字在10-25之间 则继续查看是否其后还有数字
                    if (digit>=10&&digit<=25) {
                        //如果其后还有数字 则需要将后一部分的翻译数目作为第二部分的翻译数目加到总翻译数目上
                        if(i<len-2)
                            count+=counts[i+2];
                        else
                            //否则第二部分就只有第i个数字和第i+1个数字组合起来翻译这一种 将其加到总翻译数目上
                            count+=1;
                    }
                }
    
                //第一部分和第二部分加完后就得到了以第i个数字开头的翻译数目 将其记录下来以供递推使用
                counts[i]=count;
            }
    
            //最后整个数字的翻译数目即以第0个数字开头的翻译数目 将其返回
            return counts[0];
        }
    }
    ```
* 礼物的最大价值
    在一个m*n的棋盘的每一个格都放有一个礼物，每个礼物都有一定价值（大于0）。从左上角开始拿礼物，每次向右或向下移动一格，直到右下角结束。给定一个棋盘，求拿到礼物的最大价值。例如，对于如下棋盘
    ```java
    1    10   3    8
    12   2    9    6
    5    7    4    11
    3    7    16   5
    ```
    礼物的最大价值为1+12+5+7+7+16+5=53。
    ```java
    public class Solution{
        public int getMaxValueSolution(int[][] values){
            //判空
            if (values==null||values.length==0||values[0].length==0)
                return 0;
    
            //得到数组的行数和列数
            int rows=values.length;
            int cols=values[0].length;
    
            //用于存储上一行第j到cols-1个格子的礼物最大值
            //以及该行第0到第j-1个格子的礼物最大值
            int[] maxValues=new int[cols];
    
            //遍历每个格子的礼物值
            for (int i=0;i<rows ;i++) {
                for (int j=0;j<cols;j++) {
                    //存放(i,j)格子上方格子礼物最大值
                    int up=0;
                    //存放(i,j)格子左方格子礼物最大值
                    int left=0;
    
                    //如果(i,j)格子有上方格子 则得到上方格子礼物最大值
                    if (i>0)
                        up=maxValues[j];
    
                    //如果(i,j)格子有左方格子 则得到左方格子礼物最大值
                    if (j>0)
                        left=maxValues[j-1];
    
                    //通过f(i,j)=max(f(i-1,j),f(i,j-1))+gift(i,j)
                    //来计算坐标(i,j)格子的最大礼物值
                    maxValues[j]=Math.max(up,left)+values[i][j];
                }
            }
    
            //最右下方的格子为最后一个遍历的格子 它的礼物最大值存储在maxValues[cols-1]
            return maxValues[cols-1];
        }
    }
    ```
* 最长不含重复字符的子字符串
输入一个字符串，找出字符串中最长的不含重复字符的子字符串，计算该子字符串的长度。假设字符串中的字符为“a~z”，例如 arabcacfr ，最长的字符串为 rabc 和 acfr ，长度为 4 。
    ```java
    public class Solution{
        public int longestSubstringWithoutDuplication(String str){
            //判空
            if (str==null||str.length()==0)
                return 0;
    
            //用于存放每个字符最后出现时的下标位置
            int positions[]=new int[26];
            //因为一开始没有字符出现过 所以将每个字符最后出现时的下标位置初始化为-1
            for (int i=0;i<positions.length;i++)
                positions[i]=-1;
    
            //用于存放不包含重复字符的子字符串的长度中的最大值
            int maxLen=0;
            //用于存放f(i) 即遍历到的字符时以其为结尾的不含重复字符的子字符串的长度
            int currentLen=0;
    
            //将字符串转化为字符数组便于遍历
            char[] chars=str.toCharArray();
    
            //从第一个字符开始遍历每个字符
            for (int i=0;i<chars.length;i++) {
                //取出当前第i个字符
                char ch=chars[i];
    
                //如果当前字符之前尚未出现过 则f(i)=f(i-1)+1
                if (positions[ch-'a']<0)
                    currentLen++;
                    //如果当前字符之前出现过
                else{
                    //计算当前字符的下标i和上次出现的位置的距离
                    int d=i-positions[ch-'a'];
    
                    //如果距离d小于等于f(i-1) 则说明上次出现的位置位于以第i-1个字符结尾的不包含重复字符的最长子字符串中
                    if (d<=currentLen)
                        //则f(i)=d 即从上次出现的位置后的一个字符开始到当前字符的长度
                        currentLen=d;
                    else
                        //否则说明上次出现出现的位置位于第i-1个字符结尾的不包含重复字符的最长字符串之前
                        //则f(i)=f(i-1)+1
                        currentLen++;
                }
    
                //计算完后记录当前字符最后一次出现的位置
                positions[ch-'a']=i;
    
                //此时已经得到了f(i) 即以第i个字符结尾的不包含重复字符的字符串的最长长度
                //与历史最大长度进行比较 如果大于它则其值
                if (currentLen>maxLen)
                    maxLen=currentLen;
            }
    
            //当所有字符遍历完后 直接返回历史最大值即可
            return maxLen;
        }
    }
    ```
* 0~n-1中缺失的数字
一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0 ~ n-1之内。在范围0~n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。
    ```java
    public class Solution{
        public int getMissingNumber(int array[]){
            //判空
            if(array==null||array.length==0)
                return -1;
    
            //存放划分的起始和结束下标
            int begin=0;
            int end=array.length-1;
    
            //当还可以继续划分的时候 不断循环
            while(begin<=end){
                int mid=(begin+end)/2;
                //查看中间数是否和下标相等
                if (array[mid]==mid)
                    //如果相等则继续向后查找第一个和下标不等的数字
                    begin=mid+1;
                else {
                    //否则不等
                    //继续查看上一个数字是否存在并且是否和下标相等
                    if((mid>0&&array[mid-1]==(mid-1))||mid==0)
                        //如果前一个数字不存在或者前一个数字和下标相等
                        //则当前数字就是要找的第一个和下标不等的数字
                        //返回当前数字的下标 即缺失的数字
                        return mid;
                    else
                        //否则前一个数字仍然和下标不等 继续向前寻找第一个和下标不等的数字
                        end=mid-1;
                }
            }
    
            //最后如果没有找到则说明输入的测试用例不满足题干 返回-1
            return -1;
        }
    }
    ```
* 数组中数值和下标相等的元素
假设一个单调递增的数组里的每个元素都是整数并且是唯一的。请编程实现一个函数，找出数组中任意一个数值等于其下标的元素。例如，在数组{-3,-1,1,3,5}中，数字3和它的下标相等。
    ```java
    public class Solution{
        public int getNumberSameAsIndex(int[] array){
            //判空
            if (array==null||array.length==0) {
                return -1;
            }
    
            //指向二分查找的开始与结束
            int begin=0;
            int end=array.length-1;
    
            //当范围可以继续缩小时 继续查找
            while (begin<=end) {
                //得到中间数的下标
                int mid=(begin+end)/2;
                //如果中间数就等于其下标 则该数就是所找的数字 返回
                if (array[mid]==mid)
                    return array[mid];
                else if(array[mid]<mid)
                    //如果中间数小于其下标 
                    //由于数组是从小到大排序的 且没有重复数字
                    //所以前面的数字一定会比其自身的下标要小
                    //需要继续向后进行查找
                    begin=mid+1;
                else if (array[mid]>mid)
                    //如果中位数大于其下标
                    //由于数组是从小到大排序的 且没有重复数字
                    //所以后面的数字一定会比其自身的下标要大
                    //需要继续向前进行查找
                    end=mid-1;
            }
    
            //最后如果没有查找到则返回-1 因为不会有数字和下标-1相等
            return -1;
        }
    }
    ```
* 数组中唯一只出现一次的数字
在一个数组中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字
    ```java
    public class Solution{
        public int findNumberAppearingOnce(int array[]){
            //判空
            if(array==null||array.length==0)
                return -1;
    
            //用于存放整型二进制每位累加和
            int[] bitSum=new int[32];
    
            //遍历数组中每个数字 将每个数字的二进制的每位累加
            for (int i=0;i<array.length;i++) {
                int num=array[i];
                //用于判断一位数是否为1
                int bitMask=1;
    
                //按照逻辑按位从低到高在累加数组中从后往前排放
                for (int j=31;j>=0;j--) {
                    //判断当前位是否为1 如果为1当前位累加1 否则不累加即等价于加0
                    if ((num&bitMask)!=0)
                        bitSum[j]+=1;
    
                    //左移一位 继续判断高位是否为1
                    bitMask<<=1;
                }
            }
    
    
            //用于存放生成的只出现一次的数字
            int onceNum=0;
            //累加完之后遍历所有数二进制累加结果的每一位 生成出现一次的数字
            //从高位开始遍历 每确定只出现一次的数字的一位 就向左移 即从高位开始确定每一位
            for (int i=0;i<32;i++) {
                //左移一位 给下一位生成的数字留出空位
                onceNum<<=1;
    
                //判断当前位是否能被3整除
                if ((bitSum[i]%3)!=0)
                    //如果能不能被3整除则说明只出现一次的数字的该位为1
                    onceNum+=1;
                //否则能被3整除 说明出现一次的数字的该位为0 不做加操作
    
            }
    
            //最后从高位到低位 依次确定了只出现一次数字的每一位之后 返回结果
            return onceNum;
        }
    }
    ```
* 队列的最大值
请定义一个队列并实现函数max得到队列里的最大值，要求函数max、push_back()和pop_front的时间复杂度都是O(1)
    ```java
    public class QueueWithMax{
        //记录当前数字的下标 初始下标为0
        private int currentIndex;
        //数据队列
        private Queue<IndexedData> dataQueue=new LinkedList<>();
        //存放当前队列中可能的最大值的下标及其数值
        private Deque<IndexedData> deqeue=new LinkedList<>();
    
        //入队操作
        public void enqueue(int num){
            //不断查看双端队列的尾部是否小于等于当前数字 如果是则移除 它们已经不可能成为当前队列的最大值
            while(deqeue.size()!=0&&deqeue.peekLast().data<=num)
                deque.pollLast();
    
            //这之后将当前数字及其下标放入尾部 其可能成为当前队列最大值 但是双端队列的最小值
            IndexedData IndexedData=new IndexedData(currentIndex++,num);
            deque.offer(IndexedData);
            //并将当前数字加入数据队列
            dataQueue.offer(IndexedData);
        }
    
        //出队操作
        public int enqueue(){
            //如果当前队列为空 则抛出异常
            if(dataQueue.size()==0)
                throw new Exception("队列为空");
    
            //查看出队的数字是否就是双端队列的头部 即当前队列的最大值
            if(deque.peek().index==dataQueue.peek().index)
                //因为当前队列最大值出队 所以双端队列头部也要出队
                deque.poll();
    
            //数据队列出队
            dataQueue.poll();
        }
    
        //返回当前队列的最大值
        public int max(){
            //如果当前队列为空 则抛出异常
            if(dataQueue.size()==0)
                throw new Exception("队列为空");
    
            //否则返回最大值即可
            return deque.peek().data;
        }
    }
    //用于存放数字及其下标的类
    class IndexedData{
        public int data;
        public int index;
        public IndexedData(int index,int data){
            this.data=data;
            this.index=index;
        }
    }
    ```
* n个骰子的点数
把n个骰子仍在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。
    ```java
    //方法一 采用递归的方案 存在重复计算
    public class Solution{
        //一个骰子的最大点数
        private int maxValue=6;
        //n为骰子的个数
        public void printProbability(int n){
            //判断边界条件
            if(n<1)
                return ;
    
            //计算n个骰子的和的最大值和最小值
            int minSum=n,maxSum=maxValue*n;
    
            //用数组存放每种和出现的次数 共有maxSum-minSum+1种和
            int[] timesOfSum=new int[maxSum-minSum+1];
    
            //统计每种和出现的次数
            countTimesOfSum(n,timesOfSum);
    
            //n个筛子的和总共有maxValue的n次方个情况
            int totalConditions=(int)Math.pow(maxValue,n);
    
            //输出每种和的概率
            for(int i=minSum;i<=maxSum;i++)
                System.out.println("The p of "+i+" is "+((double)timesOfSum[i-n]/(double)totalConditions));
        }
    
        //统计各种和出现的次数
        private void countTimesOfSum(int n,int[] timesOfSum){
            //先给第一个骰子安排数字
            for (int i=1;i<=maxValue;i++)
                assignNumToDice(n,1,i,timesOfSum);
        }
    
        //将数字安排给第i个骰子 并记录在该种安排下各个骰子的和
        //直到最后一个骰子被安排好数字时 将这种和的次数加1
        private void assignNumToDice(int n,int i,int sum,int[] timesOfSum){
            //如果当前是给最后一个骰子安排了数字
            //则在这种情况下的和为sum 将这种和的次数加1 并返回
            if(i==n){
                timesOfSum[sum-n]++;
                return;
            }
    
            //否则还有骰子没有安排完 继续安排
            for(int j=1;j<=maxValue;j++)
                //因为给下一个骰子安排了数字 所以累加和加上安排的数字
                assignNumToDice(n,i+1,sum+j,timesOfSum);
        }
    }
    
    //方法二 采用循环方案 用两个数组轮流生成和的次数 减少了重复计算
    public class Solution{
        //一个骰子的最大点数
        private int maxValue=6;
        //n为骰子的个数
        public void printProbability(int n){
            //判断边界条件
            if(n<1)
                return ;
    
            //用于存放和的最大值
            int maxSum=n*maxValue;
    
            //两个数组 用于轮流存放给一个骰子分配了数字之后的各种和的次数
            int[][] timesOfSum=new int[2][maxSum+1];
    
            //首先采用第一个数组
            int flag=0;
    
            //首先统计第一个骰子分配点数之后各种和的次数
            for(int i=1;i<=maxValue;i++)
                timesOfSum[flag][i]=1;
    
            //从第二个骰子分配点数 轮流用两个数组来存放在分配完第i个骰子的点数后各种和的次数
            for(int i=2;i<=n;i++){
                //当为第i个骰子分配点数时 其和不可能为0到i-1 所以将相应和的次数清零
                for(int j=0;j<i;j++)
                    timesOfSum[1-flag][j]=0;
    
                //当为第i个骰子分配了点数之后 其和可能为从i到i*maxValue
                //将根据上一个数组存储的次数来生成这些和的次数
                for(int j=i;j<=i*maxValue;j++){
                    //由于这是新一轮生成次数 需要先将上上轮存放的次数清零
                    timesOfSum[1-flag][j]=0;
    
                    //再将上个数组中目标和-1/-2/-3/...-6位置的次数累加
                    //即在当前骰子分配后和为n的次数就是将上一个骰子分配点数时的n-1/n-2/.../n-6的次数累加起来
                    //就是当前骰子分配后相应和的次数
                    for(int k=1;k<=maxValue&&j-k>=1;k++)
                        timesOfSum[1-flag][j]+=timesOfSum[flag][j-k];
    
                }
                //下一轮采用另个一个数组来存放新生成的次数
                flag=1-flag;
            }
    
            //计算所有和的情况 总共有骰子点数的n次方种情况
            int totalConditions=(int)Math.pow(maxValue,n);
    
            //输出每种和的概率
            for(int i=n;i<=maxSum;i++)
                System.out.println("The p of "+i+" is "+((double)timesOfSum[flag][i]/(double)totalConditions));
        }
    
    }
    ```
* 股票的最大利润
假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？例如，一只股票在某些时间节点的价格为{9,11,8,5,7,12,16,14}。如果我们能在价格为5的时候买入并在价格为16时卖出，则能收获最大的利润11。
    ```java
    public class Solution{
        public int maxDiff(int[] numbers){
            //判空与判断边界条件
            //求利润最少要两个价格
            if(numbers==null||numbers.length<2)
                return -1;
    
            //存放之前价格的最小值
            int min=numbers[0];
            //存放最大利润
            int maxDiff=numbers[1]-min;
    
            //遍历之后的每个数字
            for(int i=2;i<numbers.length;i++){
                //如果当前数字的前一个数字比最小价格要小 则更新最小价格
                if(numbers[i-1]<min)
                    min=numbers[i-1];
    
                //如果当前数字与最小价格的差 即利润最大 则更新最大利润
                if((numbers[i]-min)>maxDiff)
                    maxDiff=numbers[i]-min;
            }
    
            //最后返回最大利润
            return maxDiff;
        }
    }
    ```
* 树中两个节点的最低公共祖先
输入两个树节点 求它们的最低公共祖先
    ```java
    import org.junit.Test;
    
    import java.util.LinkedList;
    import java.util.Iterator;
    
    class TreeNode{
        public char value;
        public TreeNode[] children;
    
        @Override
        public String toString() {
            return "TreeNode{" +
                    "value=" + value +
                    '}';
        }
    }
    
    public class Solution{
        public TreeNode getLastCommonParent(TreeNode root,TreeNode node1,TreeNode node2){
            //判空
            if(root==null||node1==null||node2==null)
                return null;
    
            //存放根节点开始到某个节点的路径
            LinkedList<TreeNode> list1=new LinkedList<>();
            LinkedList<TreeNode> list2=new LinkedList<>();
            //寻找从根节点到两个节点的路径
            boolean found1=getPath(root,node1,list1);
            boolean found2=getPath(root,node2,list2);
    
            //存放两个链表的最深公共节点
            TreeNode commonNode=null;
            //如果找到从根节点到两个节点的路径后 再找到两个路径链表从后到前的第一个公共节点
    
            if(found1&&found2){
                commonNode=getCommonNode(list1,list2);
            }
    
            //最后返回公共节点
            return commonNode;
        }
    
        private boolean getPath(TreeNode root,TreeNode node,LinkedList list){
            //用前序遍历来寻找路径
    
            //存放当前从当前节点是否找到了路径
            boolean found=false;
            //如果当前节点不为则继续遍历
            if (root!=null) {
                list.offer(root);
                //如果当前节点就是要找的节点 则找到路径
                if(root==node)
                    found=true;
                else if(root.children!=null){
                    //否则如果存在子节点继续递归地在当前节点的儿子中寻找路径
                    for(TreeNode child:root.children){
                        found=getPath(child,node,list);
                        //直到在儿子节点中找到路径才返回
                        if(found==true)
                            break;
                    }
                }
    
                //如果从当前节点为起点没有找到节点则将当前节点从路径中移出
                if(found==false)
                    list.pollLast();
    
            }
    
            //最后返回从当前节点是否找到路径
            return found;
        }
    
        private TreeNode getCommonNode(LinkedList list1,LinkedList list2){
            //遍历两个链表 寻找到最深的公共节点
            TreeNode commonNode=null;
    
            Iterator<TreeNode> i1=list1.iterator();
            Iterator<TreeNode> i2=list2.iterator();
    
            //两个指针在链表上同时移动 记录最后一个公共节点
            while(i1.hasNext()&&i2.hasNext()){
                TreeNode node1=i1.next();
                TreeNode node2=i2.next();
    
                if(node1==node2)
                    commonNode=node1;
            }
    
            return commonNode;
        }
        @Test
        public void test(){
            TreeNode root=new TreeNode();
            root.value='A';
            TreeNode node1=new TreeNode();
            node1.value='B';
            TreeNode node2=new TreeNode();
            node2.value='C';
            root.children=new TreeNode[]{node1,node2};
    
            TreeNode node3=new TreeNode();
            node3.value='D';
            TreeNode node4=new TreeNode();
            node4.value='E';
            node1.children=new TreeNode[]{node3,node4};
    
            TreeNode node5=new TreeNode();
            node5.value='F';
            TreeNode node6=new TreeNode();
            node6.value='G';
            node3.children=new TreeNode[]{node5,node6};
    
            TreeNode node7=new TreeNode();
            node7.value='H';
            TreeNode node8=new TreeNode();
            node8.value='I';
            TreeNode node9=new TreeNode();
            node9.value='J';
            node4.children=new TreeNode[]{node7,node8,node9};
    
            System.out.println(getLastCommonParent(root, node5, node7));
        }
    }
    ```