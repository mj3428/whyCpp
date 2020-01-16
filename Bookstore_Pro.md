# 书店程序
```
# include <iostream>
# include "Sales_item.h"

int main()
{
	Sales_item total;
    if (std::cin >> total){
        Sales_item trans;
        while(std::cin >> trans){
            if (total.isbn() == trans.isbn())
                total += trans;
            else{
                std::cout << total << std::endl;
                total  = trans;
            }
        }
        std::cout << total << std::endl;
    }else{
        std::cerr << "No data?!" << std::endl;
        return -1;
    }
    return 0;
	
}
```
与往常一样,首先包含要使用的头文件:来自标准库的iostream和自己定义的 Sales-item.h。在main中,我们定义了一个名为total的变量,
用来保存一个给定的 "ISBN的数据之和。我们首先读取第一条销售记录,存入total中,并检测这次读取操作是否成功。如果读取失败,则意味着没有任
何销售记录,于是直接跳到最外层的else分支,打印一条警告信息,告诉用户没有输入。  
while的循环体是一个单个的if语句,它检查ISBN是否相等。如果相等,使用复合赋值运算符将trans加到total中。如果ISBN不等,我们打印保
存在total中的值,并将其重置为trans的值。在执行完if语句后,返回到while的循环条件,读取下一条销售记录,如此反复,直至所有销售记录都处理完。  
当while语句终止时, total保存着文件中最后一个ISBN的数据。我们在语句块的最后一条语句中打印这最后一个ISBN的total值,至此最外层if语句就结束了。  
