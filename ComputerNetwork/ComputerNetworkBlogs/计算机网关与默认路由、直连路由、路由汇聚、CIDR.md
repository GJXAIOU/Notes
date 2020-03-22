# 计算机网关与默认路由、直连路由、路由汇聚、CIDR

## 1、名词概念解释： 
①**静态路由**（Static Routing）：即由**网络管理员/用户静态指定**，不会随时间、流量、拓扑结构等因素变化而变化的路由路径。

②**动态路由**（Dynamic Routing）：由**路由器自动学习**，受时间、网络流量、拓扑结构等因素变化而变化的路由路径。

③**直连路由**（Connected Route）：因传输介质直连（直接连接）而产生的路由路径，直连不需要再设置路由，但设置也不会出错，如下图所示： 

![link](https://img-blog.csdn.net/20180819230612815?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 
R1和192.168.10.0、192.168.20.0两个网段直接连接，因此连线接通之后就直接存在直连路由，但是R1和192.168.30.0网段没有直接连接，则R3到192.168.30.0网段不存在直连路由。其他路由器类似，该拓扑中每个路由器都有两条直连路由并配置了两条静态路由如下： 

![link](https://img-blog.csdn.net/20180819231431587?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
④**路由汇聚**（Routing aggregation）：多条路由路径合成一条路由路径（形态：合成的一条路由路径，其子网掩码1的位数必定不大于原有路由路径子网掩码1的位数）。对于上图，我们可以看到对于R1，无论是去192.168.30.0/24网段还是去192.168.40.0/24网段，其出口地址均为192.168.20.2。我们可以修改为ip route 192.168.0.0 16 192.168.20.2，即合并成一条，将网络位第三段的20或30整合到0便可实现路由聚合减少路由表项数目。对于R3路由器也是同理，但是R2不行。如下所示： 

![link](https://img-blog.csdn.net/20180819232308944?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
其实也可以按下所示进行聚合，我们可以看到R3的聚合和R1的聚合不相同，R1将两条C类聚合为一个B类，而R3将两条C类聚合为一个A类，但是都达到了聚合的目的，所以都是可以的： 
![link](https://img-blog.csdn.net/20180819232318476?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

⑤**无类域间路由**（Classless Interdomain Routing，CIDR）：严格的路由汇聚，或者说是路由汇聚的最优方式/精确汇聚（无类域间路由路由属于但不等于路由汇聚的范围）。因为上图中，其路由聚合有多种方式，而CIDR是比较严格的路由聚合，对于其聚合是跨越了A、B、C类地址分类的聚合方式，如对于R1，192.168.30.0/24、192.168.40.0/24是只能进行路由聚合而不能进行CIDR的。下图可以进行CIDR： 

![link](https://img-blog.csdn.net/20180819233419199?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
对于R1，其需要设置静态路由的两个网段为192.168.2.0/24和192.168.3.0/24，其第三段为2（1 0）和3（1 1），则将原有子网掩码左移一位使得192.168.2.0/24和192.168.3.0/24变成同一个网段192.168.2.0/23，且该网段的/24子网只存在192.168.2.0/24和192.168.3.0/24两个，不存在其他，这就是CIDR。CIDR其实是“超网”的逆过程即子网划分的逆过程子网合并。而之前的R1的两条路由192.168.30.0/24和192.168.40.0/24其要合并成一个大网段，就是将30（00 01 1110）和40（00 10 1000）现有的子网网络位化为主机位，子网掩码至少需要左移6位才可以达到子网合并，这其中共有64个子网，而当前只有2个，因此不能“精确汇聚”，即不能CIDR。

⑥**默认路由**（Default Routing）：路由器设置的不存在路由路径的所有报文均采用默认路由进行转发，对于192.168.30.0/24和192.168.40.0/24就是ip route 0.0.0.0 0.0.0.0 192.168.20.2，即最大范围的路由汇聚，包含了所有子网。 
⑦网关（Gateway）：网关其实是对于PC/Server等非网络设备这个概念会更加明显，网关实质就是局域网的默认路由的下一跳地址。

关于有多条路由设置的问题，优先级如下： 
直连路由 > 最长掩码匹配 > 路由汇聚 > CIDR > 默认路由 
（因为有最长掩码优先匹配原则，因此默认路由必定是最后才会使用的路由方式）。

## 2、计算机设置网关与默认路由： 
上面说网关实质就是局域网的默认路由的下一跳地址，我们可以使用route print命令在windows上查看，第一条就是网关代表的默认路由（下面有一部分192.168.10.0、192.168.20.0、192.168.80.0等的路由是VMware的路由记录）： 
![link](https://img-blog.csdn.net/2018081923493640?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如果我们在adapter属性设置中把IP设置为静态并不设置网关，但是通过自己设置路由的方式来代替在adapter中设置网关的目的。如下所示： 
![link](https://img-blog.csdn.net/2018081923591541?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
可以ping一下百度，可以ping通： 

![link](https://img-blog.csdn.net/20180819235948354?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
也就是说，有时候可能PC/Server需要不设置网关，改用设置路由的方式来达到一定的功能，如下所示： 

![link](https://img-blog.csdn.net/2018082000100316?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如果`PC_A`要和Internet与`PC_B`通信，那么应该将网关设为同在局域网内的R1的接口（192.168.10.1）还是R2的接口（192.168.10.2）。假设将PC_A的网关设置为192.168.10.1，则访问Internet直接由R1转发就出去了，但是如果与`PC_B`通信，则`PC_A`一看是目标地址192.168.20.10和自己不在用一个网段就会把数据包丢给R1网关，R1则根据路由表返回来丢给R2，R2再丢给`PC_B`。那为什么不把`PC_A`的网关直接设置成R2的192.168.10.2，其实访问`PC_B`少一次路由，但访问Internet又多一次路由。

有没有什么办法让`PC_A`访问Internet直接由R1转发，而与`PC_B`通信直接发给R2而不经R1转发给R2？既然问出来这个问题，那当然是有的了，就是采用给`PC_A`设置网关为R1的192.168.10.1，但是也增加一条路由到`PC_B`：route ADD 192.168.20.0 MASK 255.255.255.0 192.168.10.2，则`PC_A`遇到发给`PC_B`（192.168.20.0网段）的数据则直接根据最长掩码匹配选择设置的路由将数据交给R2，而非192.168.20.0网段的数据就采用默认路由，即设置的网关来直接转发到Internet中去。[link](https://img-blog.csdn.net/20180819230612815?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0Fwb2xsb25fa3Jq/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
