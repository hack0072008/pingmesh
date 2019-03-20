Pingmesh：用于数据中心网络延迟测量和分析的大规模系统
====

概论
-
&emsp;&emsp;我们是否可以在大型数据中心网络中随时获得任意两台服务器之间的网络延迟？收集的延迟数据可用于解决一系列挑战：告知应用程序感知延迟问题是否由网络引起，定义和跟踪网络服务水平协议（SLA）以及自动网络故障排除。
   我们开发了用于大规模数据中心网络延迟测量和分析的Pingmesh系统，以肯定地回答上述问题。 Pingmesh已在Microsoft数据中心运行了四年多，每天收集数十TB的延迟数据。 Pingmesh不仅被网络软件开发人员和工程师广泛使用，还被应用程序和服务开发人员和运营商广泛使用。
    
CCS概念

网络->网络测量、云计算、网络监控、计算机系统架构->云计算

关键字

数据中心网络 网络故障排除 无声数据包丢失


1.介绍
-
&emsp;&emsp;在今天的数据中心，有数十万台服务器。这些服务器通过网络接口卡（NIC），交换机和路由器，电缆和光纤连接，形成大规模的内部和内部数据中心网络。 由于云计算的快速发展，数据中心网络（DCN）的规模正在扩大。 在物理数据中心基础设施之上，构建了各种大规模的分布式服务，例如Search [5]，分布式文件系统[17]和存储[7]，MapReduce [11]。<br>
&emsp;&emsp;这些分布式服务是庞大且不断发展的软件系统，具有许多组件并具有复杂的依赖性。 所有这些服务都是分布式的，许多组件需要通过网络在数据中心内或跨不同的数据中心进行交互。 在这样的大型系统中，软件和硬件故障是常态而非例外。因此，网络团队面临着一些挑战。<br>
&emsp;&emsp;第一个挑战是确定问题是否是网络问题?由于分布式系统的性质，许多故障显示为“网络”问题，例如，某些组件只能间歇性地到达，或者端到端延迟显示在第99百分位处突然增加，每台服务器的网络速度吞吐量从20MB/s降低到5MB/s。我们的经验表明，这些“网络”问题中约有50％不是由网络引起的。 然而，要判断“网络”问题是否确实是由网络故障引起的，这并不容易。<br>
&emsp;&emsp;第二个挑战是定义和跟踪网络服务水平协议（SLA）。许多服务需要网络提供某些性能保证。 例如，搜索查询可能会触及数千台服务器，搜索查询的性能由最慢服务器的最后一次响应确定。这些服务对网络延迟和数据包丢失敏感，他们关心网络SLA。需要针对不同的服务单独测量和跟踪网络SLA，因为它们可能使用不同的服务器集和网络的不同部分。 由于网络中的大量服务和客户，这成为一项具有挑战性的任务。<br>
&emsp;&emsp;第三个挑战是网络故障排除。当网络SLA因各种网络故障而中断时，会发生“实时站点”事件。 实时站点事件是指对客户，合作伙伴或收入产生影响的任何事件。 现场事件需要尽快被检测，缓解和解决。但数据中心网络拥有数十万到数百万台服务器，数十万台交换机以及数百万条电缆和光纤。因此，检测问题所在的位置是一个难题。<br>
&emsp;&emsp;为了应对上述挑战，我们设计并实施了Pingmesh，这是一个用于数据中心网络延迟测量和分析的大型系统。 Pingmesh利用所有服务器启动TCP或HTTP pings,以提供最大延迟测量覆盖率。Pingmesh形成多层次的完整图表。在数据中心内，Pingmesh允许机架内的服务器形成完整的图形，并使用架顶式（ToR）交换机作为虚拟节点，并让它们形成第二个完整的图形。在数据中心之间，Pingmesh通过将每个数据中心视为虚拟节点来形成第三个完整的图形。完整图形和相关ping参数的计算由中央Pingmesh控制器控制。<br>
&emsp;&emsp;测量的延迟数据由数据存储和分析管道收集和存储，汇总和分析。根据等待时间数据，在宏级别（即数据中心级别）和微级别（例如，每服务器和每个机架级别）定义和跟踪网络SLA。 通过将服务和应用程序映射到它们使用的服务器来计算所有服务和应用程序的网络SLA。<br>
&emsp;&emsp;Pingmesh已经在微软的数十个全球分布式数据中心运行了四年。 它每天产生24TB的数据和超过200亿的探测数据。 由于Pingmesh数据的普遍可用性，因为网络变得更容易回答实时站点事件：如果Pingmesh数据不表示网络问题，则实时站点事件不是由网络引起的。<br>
&emsp;&emsp;Pingmesh主要用于网络故障排除，以找出问题所在。 通过可视化和自动模式检测，我们能够回答数据包丢失和/或延迟增加的时间和地点，识别网络中的静默交换机数据包丢失和黑洞。 应用程序开发人员和服务运营商也使用Pingmesh生成的结果，通过考虑网络延迟和数据包丢弃率来更好地选择服务器。<br>
&emsp;&emsp;本文作出了以下贡献：我们通过设计和实施Pingmesh，展示了建立大规模网络延伸测量和分析系统的可行性。 通过让每个服务器参与，我们始终为所有服务器提供延迟数据。 我们展示了Pingmesh通过在宏观和微观范围内定义和跟踪网络SLA来帮助我们更好地理解数据中心网络，并且Pingmesh帮助揭示和定位交换机数据包丢弃，包括数据包黑洞和静默随机数据包丢弃，这在以前很少被了解。<br>

2.背景
-
2.1 数据中心网络

&emsp;&emsp;数据中心网络以高速连接服务器，并提供高服务器到服务器带宽。今天的大型数据中心网络是由商品以太网交换机和路由器构成的。
![Image text](https://github.com/akliy/pingmesh/blob/master/pingmesh-image/Figure%201-Date%20center%20network%20structure)<br>
&emsp;&emsp;图1显示了典型的数据中心网络结构。 网络有两部分：内部数据中心（Intra-DC）网络和内部数据中心（Inter-DC）网络。 DC内网络通常是几层Clos网络架构，类似于[1,12,2]中描述的网络。 在第一层，数十个服务器（例如40台）使用10GbE或40GbE以太网NIC连接到架顶式（ToR）交换机并形成一个Pod节点。 然后将数十个ToR交换机（例如，20台）连接到第二层节点（汇聚）交换机（例如，2-8）。 这些服务器和ToR和Leaf交换机组成一个Podset。 然后，多个Podset连接到第三层Spine交换机（数十到数百）。使用现有的以太网交换机，DC内部网络可以连接数万个或更多具有高网络容量的服务器。<br>
&emsp;&emsp;一个优秀的数据中心是多个Leaf和Spine交换机提供具有冗余的多路径网络。 ECMP（等价多路径）用于对所有路径上的流量进行负载均衡。 ECMP使用TCP / UDP五元组的哈希值进行下一跳选择。 因此，即使连接的五元组已知，TCP连接的确切路径在服务器端也是未知的。 因此，找到有故障的Spine交换机并不容易（因为路径无法确定？）。<br>
&emsp;&emsp;DC间网络用于互连DC内网络并将DC间网络连接到因特网。 DC间网络使用高速长途光纤连接不同地理位置的数据中心网络。 软件定义网络（SWAN [13]，B4 [16]）被进一步引入，以实现更好的广域网络流量工程。<br>
&emsp;&emsp;我们的数据中心网络是一个庞大，复杂的分布式系统。 它由数百个服务器，数万个交换机和路由器以及数百万个电缆和光纤组成。 它由我们自行开发的数据中心管理软件堆栈Au-topilot [20]管理，交换机和NIC运行由不同交换机和NIC提供商提供的软件和固件。 在网络上运行的应用程序可能会引入复杂的流量模式。<br>
