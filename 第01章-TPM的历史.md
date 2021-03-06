# TPM的历史
可信平台模块（TPM）是现在大多数个人电脑和服务器都有的密码协处理器。单从计算机的硬件配置角度来说，TPM是一个普遍存在的配置。但是因为没有重要的应用使用它们，所以直到今天大多数情况下用户并不知道它们的存在。当然这种情况正在迅速改变。随着最近各种各样的TPM设计得到美国联邦信息处理标准的认证，以及总统顾问委员会推荐美国政府使用TPM来保护国家的计算机，TPM已经成为计算机所有者用来保护密码资产的重要工具。即便如此，现在大多数人对TPM的了解并不足以满足顺利使用的要求，这也催生了这本书的写作。本章将简要介绍各种版本TPM，包含了TPM1.1b到TPM2.0之间的版本。

## 为什么用TPM？
1990年代，计算机领域的人逐渐意识到互联网将改变个人计算机的连接方式，相关的商业也转向互联网。同时这种情况也使得个人电脑的安全性需求大大增加。在个人电脑发展的初期，其设计很少考虑安全需求，所以也没有相关的硬件支持。软件设计的情况也是一样，主要的设计驱动是易用性。

一群计算机工程师为了扭转这种趋势设计了第一代TPM，它可以作为构建个人电脑系统安全的硬件基础。这群工程师后来成为可信计算组织（TCG）的成员。基于成本的考虑，这种安全解决方案的价格必须足够低。最终的设计结果是可以直接和主板连接的TPM芯片。TPM的命令集用来提供所有安全应用所需的功能，我们会在第三章详细描述。除此之外，任何其他不必要的功能被设计成软件实现，从而降低成本。但是这种以低成本为原则的设计也导致TPM的规范描述文档难以阅读。
硬件安全的话题不太容易展开，同时令人费解的文档也没有任何的帮助。姑且就以这种技术的历史为切入点开始介绍。

## TPM 1.1b到1.2的发展历史
第一代大规模部署的TPM是2003年发布的TPM1.1b。这个版本在当时已经具备TPM基本功能，主要包括密钥生成（仅限RSA密钥），存储，安全授权，设备安全状态认证。同时也实现了通过匿名身份密钥来实现用户隐私保护的基础功能，这项功能需要TPM提供相关证书。身份密钥的生成需要被授权。TPM还增加了隐私证书授权中心（CA）用于证明密钥是在一个真实的TPM硬件中产生的，但不需要关心在哪一个TPM中产生。

在TPM内部有一个叫做平台配置寄存器（PCR）动态内存区域用来保存系统启动过程中的测量值。这些PCR加上身份密钥可以用来认证系统的启动过程是否正常（通常是与一些已知的PCR值比较，但不仅仅是比较这么简单）。这也是用于建立系统安全的基石，TCG的主要实现目标之一。

TCG的另外一个预期的附加性目标是让TPM免于物理攻击。尽管这对于TCG来说是可以实现的，但是最终他们基于某些原因（注：复杂度，成本？）的考量，TPM的物理防护设计由厂商实现，厂商可以根据情况进行差异化实现。不过基于TPM的安全技术包含了对软件攻击防护的设计。

IBM个人电脑是第一个使用TPM的产品，TPM以类似智能卡芯片的方式连接到主板（智能卡芯片内部也有类似的安全协处理器，这种方式主流平台使用多年）。HP和DELL随后跟进，到2005年时已经很难发现不带TPM的个人电脑了。

TPM1.1b的一个不足之处是硬件层面的兼容性不够好。因为TPM厂商通常在硬件实现上有一些细微的差异，这导致不同的TPM芯片可能需要不同的设备驱动程序，同时芯片的引脚也没有被标准化。

TCG后来意识到尽管TPM1.1b通过密码来防止密钥被攻击，但是不能阻止攻击者猜测密码并不断的尝试。攻击者通常会使用一个常用的密码字典来不断尝试，这种攻击被称为字典攻击。因此，TPM1.2后就要求能够防止字典攻击。

隐私组织常常会抱怨1.1b版本没有实现私有的CA（注：1.1b中相关证书由TPM提供）。这促使TPM1.2增加一个命令来解决这个问题，这个命令提供了第二种匿名密钥方法——直接匿名认证（DAA），同时还有代理密钥授权和管理功能。

在配置有TPM的机器上认证TPM背书密钥（EndorsementKey）需要TPM提供证书，一种部署方法是整机厂商在硬盘上存储证书，但是IT组织在安装软件之前通常会格式化硬盘，从而导致证书被删除。所以这种方法可操作性比较差。为了解决这个问题，TPM1.2的设计增加了一块非易失性内存（NVRAM，通常有2KB大小），同时附带了单向计数器来做内存的访问控制。

为了防止机器损坏或者硬件升级对TPM的影响，TPM1.1b规定了一种密钥在不同TPM之间迁移的方法。密钥迁移需要密钥所有者和TPM所有者的授权，并且TPM假设TPM所有者是IT组织，而用户是密钥的所有者。但是在TPM1.2中用户必须有能力使用TPM所有者的权限来授权消除字典攻击和授权建立NVRAM（意思是说密钥迁移所需要的两种权限就没有办法独立控制了），所以1.2中密钥迁移的授权方式就不适用了（这里的意思应该是user自己可以私自迁移）。因此，1.2设计了一种机制可以让用户可以创建只能由第三方实施迁移的密钥。这种密钥可以被（第三方）认证，因此称为被认证的可以迁移密钥（Certified Migratable Keys）.

签名密钥通常用来签署合同，因此签署过程中加入时间戳信息是非常有用的。TCG曾经考虑在TPM增加时钟，但问题是没有电池的情况下，TPM断电时钟也就不能维持运行了。尽管可以在设计中增加电池听起来也是可行的，可这将会大大增加实现的成本。最后的结果是，TPM被设计成内部的计数器可以同步外部时钟，这样签名操作时就可以使用内部计数器的值。这种内部计时器和外部时钟相结合的方法可以决定合同签署的时间。TPM也可以通过这种设计确定两次签名操作的时间差。

1.2版本的设计保持了应用层编码接口不变，因此可以实现1.1b版本的软件的兼容运行。当然因为设计的差异是客观存在的，保证兼容性的副作用是1.2的设计也变得更加复杂，因为总是要处理设计差异带来的特殊情况。

在TPM普遍应用之前，一些应用通过智能卡这种安全协处理器来存储密钥和认证用户身份。TPM可以很好实现这功能，但是因为TPM是被集成在主板上的，因此它可以有额外的用处，比如说作为设备的身份，这一点我们将会在第三章继续讨论。

自2005年开始，TPM1.2被部署在大多数x86个人电脑上。2008年开始在服务器平台上部署（google intel TXT），最终大多数服务器也配置了TPM。尽管如此，只有硬件是不够的，还需要有配套的软件来使用它们才有意义。为了应用TPM，微软提供了Windows驱动，IBM则贡献了linux驱动。至此，相关的软件开始出现，第四章将会详细介绍。

## TPM由1.2发展到2.0版本
在2000年时，TCG面临哈希算法的选择。当时有两种选择：被广泛应用的MD5和同样广泛应用但不及前者的SHA-1.SHA-1是当时强度最高的商用哈希算法，并且很容易以低成本的方式实现。当时（TPM1.2）选择SHA-1是幸运的，因为不久之后MD5的弱点就被公开了。

在2005年左右，密码学研究者公布了第一个针对SHA-1的重要攻击。然而，当时广泛部署的TPM1.2重度依赖SHA-1算法，并且是硬编码，不能通过配置修改。尽管分析表明这种用于攻击SHA-1的方法在TPM上并不可行，但是密码学界公认的是密码的安全性总是随着时间的流逝而变弱而不是变强。鉴于此，TCG立即着手制定可以灵活配置哈希算法的TPM2.0规范。这就意味着设计规范将不再硬编码SHA-1或者其他任何算法，而是被设计成在不修改规范的情况下可以通过算法标识符使用任意算法。TCG希望这个包含前述修改（以及其他可以使其他密码学算法变灵活的修改）新版本规范成为最后发布的主要版本。

最初，TCG的TPM2.0工作组的原定的强制任务是增加哈希算法的灵活性。但即使是粗略地看过一遍TPM2.0的规范之后就会发现，它相对TPM1.2来说增加的不仅是算法标识符这么简单。那具体的情况到底如何呢？

TPM1.1b规范包含了精心设计过的数据结构，这些结构被序列化（数据结构变成可以传输到TPM的字节流）后足够紧凑到可以被2048位RSA密钥一次加密。当然这意味着只能使用2048位（256字节）的密钥加密。同时出于成本和实现对大量数据加密时带来的问题的考虑，这个版本的设计没有要求实现对称加密算法。1.1b版本中，RSA是唯一要求实现的非对称加密算法，而且处于性能的考虑，必须在一次操作内完成数据加密。

既然SHA-1在当时看来注定要被替代，TPM2.0的规范必须支持大于SHA-1的20字节的哈希算法，所以比较明确的是现有的数据结构将会变大。同时一个RSA密钥将不能在一次操作内加密所有的数据。因为RSA的操作非常慢，使用多次操作加密数据的方式并不现实。而增加RSA密钥的长度也不可取，因为首先相关算法在工业界并没有被大规模应用，其次这也将带来诸如增加成本，密钥数据结构变化以及拖慢速度等问题。最后，TPM工作组采用了业界通用做法，即非对称密钥加密对称密钥，对称密钥加密数据。因为对称算法相对非对称算法来说要快很多，所以很适合加密大量数据。这样一来，对称密钥的引入祛除了数据结构大小的限制。同时这也使得规范设计者以不同于1.2的方式实现一些功能。

有一种说法是最好设计是首先完成一个设计，然后认识到不足并不断地完善。TPM2.0的设计就是这样产生的。但是，现在还没有办法确认软件实现上也是如此。（截至到2018的现在，TPM2.0相关的软件也处于重度开发中，并不确认1.2中的一些功能没有被忽略）

## TPM2.0规范的开发历程
TPM2.0的规范历经了多年缓慢而长足的进步，这期间相关功能争论和增删不断。来自Intel的David Grawrock是当时规范委员会的主席。在他的带领下，工作组选定了主要的功能和基本的特性，同时完成顶层设计。此时，委员会决定修改所有的数据结构来实现算法的独立性，这被称作算法的灵活性。所有的认证技术由最初称做通用授权（generalized authorization）的技术变成统一的增强认证（enhanced authorization）技术。这一转变在减少实现成本和规范理解难度的同时增加了授权的灵活性。新的规范中，所有的对象（object）和实体（entity）使用统一的认证技术。因为新的规范中在增加算法灵活度的同时，也允许用户精确的控制算法相关的参数来决定密钥和用于密钥保护的密钥，这样使得整个TPM系统的安全性可能根据用户配置被评估出来（有潜在的安全隐患？），这个问题也带来了一些争议和讨论。

当David Grawrock因为在intel的角色变化离开委员会主席的位置时，来自微软的David Wooten成为了全日制编辑（主编），同时HP接管了主席的位置。因为此时规范被要求设计成是可编译的，这促使David Wooten在编写规范的同时开发了一个TPM仿真器。规范可编译可以大大减少含糊不清的描述，因为一旦有疑问就可以通过将规范编译成仿真器（这里理解为，由规范的设计文档直接生成模拟器所需的配置文件，头文件等信息，用于验证规范的实际运行情况）。另外，通用的认证结构由波兰表示法（TI机器）转变为反向波兰表示法（HP机器），这使得规范的实现更容易（但是阅读难度增加）。委员会还决定增加多个密钥组织架构来满足多种用户角色的需求。

Wooten废寝忘食的工作和他强大的领导力使规范的开发颇有成效。HP的Graeme Proudler在主席的位子上退休以后，来自约翰霍普金斯大学应用物理实验室的David Challener以及AMD的Julian Hammersly接任成为联合主席。同样来自Kenneth Goldman在Wooten完成第一次规范发布后成为规范主编，Kenneth Goldman曾经在1.2版本的规范开发中担任主编多年。

多年来，越来越多的成员加入并开始理解规范。其中，Will Arthur和Kenneth Goldman更是逐行深入理解规范。他们提交了很多bug和可读性的修改，从而进一步增加了规范的一致性和可读性。即便有了以上成员的努力修改，理解规范仍然有一定的难度，这也是写这本书的动力。

## 总结
TPM规范已经被完善过两次。第一次是由1.1b到1.2，主要涉及到相关新旧功能的合并，功能逐渐增加的进化过程也导致最终的规范非常复杂。在第二代也就是TPM2.0规范开发中，设计者以解决SHA-1的弱点为契机，从底层重新设计了一套更加集成化和统一的架构。下一章将介绍贯穿本书的密码学概念。理解这些概念对于你理解TPM2.0是非常有必要的。
