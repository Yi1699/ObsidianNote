
#### 堆栈溢出

----
### DSP320C6000的指令列表汇集
　　内联指令 汇编指令 简要描述
```

　　int _abs （int src）;

　　int _labs （__int40_t src）ABS返回src的绝对值

　　int _add2 （int src1， int src2）ADD2把src1的高、低16位和src2的高、低16位分别相加，放入结果的高、低16位

　　ushort & _amem2 （void *ptr）;LDHU

　　STHU从内存中加载一个halfword到dst里，必须2byte对齐（读或存）

　　const ushort & _amem2_const （const void *ptr）;LDHU必须2byte对齐（读）

　　unsigned & _amem4 （void *ptr）;LDW

　　STW必须4byte对齐（读或存）

　　const unsigned & _amem4_const （const void *ptr）;LDW必须4byte对齐（读）

　　double & _amemd8 （void *ptr）;LDW/LDW

　　STW/STW必须8byte对齐（读或存）

　　const double & _amemd8_const （const void *ptr）;LDDW必须8byte对齐（读）

　　unsigned _clr （unsigned src2， unsigned csta，unsigned cstb）;CLR指定了从需要清0的首位和末位

　　unsigned _clrr （unsigned src2， int src1）;CLR将src2中指定位清0，清0的首位和末位由src1的低10位指定

　　__int40_t _dtol （double src）; 将一个double寄存器重新解释成一个__int40_t

　　long long _dtoll （double src）; 将一个double寄存器重新解释成一个long long

　　int _ext （int src2， unsigned csta， unsigned cstb）;EXT从src2里提取csta和cstb指定的区域且符号扩展到32位。提取出的区域先符号左移再右移。

　　int _extr （int src2， int src1）;EXT同上，区别：左右移的位数由src1的低10位指定

　　unsigned _extu （unsigned src2， unsigned csta ， unsigned cstb）;EXTU同上上，区别最后是0扩展到32位。

　　unsigned _extur （unsigned src2， intsrc1）;EXTU同上，区别：左右移的位数由src1的低10位指定例：

　　_ftoi （1.0） == 1065353216U

　　unsigned _ftoi （float src）; 将float的比特位解释成unsigned

　　unsigned _hi （double src）; 返回double寄存器的高位（奇数位）

　　unsigned _hill （long long src）; 返回longlong寄存器的高位（奇数位）

　　double _itod （unsigned src2， unsignedsrc1）; 创建一个新的double寄存器为了解释2个unsigned的值，其中src2是高（奇数）寄存器，src1是低（偶数）寄存器

　　float _itof （unsigned src）; 将unsigned中的比特位解释成float例：

　　_itof （0x3f800000） = 1.0

　　long long _itoll （unsigned src2， unsignedsrc1）; 创建一个新的longlong寄存器为了解释2个unsigned的值，其中src2是高（奇数）寄存器，src1是低（偶数）

　　unsigned _lmbd （unsigned src1， unsignedsrc2）;LMBD搜索src2里面的1或0，1或0是由src1的LSB决定的，返回比特位变化的位数

　　unsigned _lo （double src）; 返回double寄存器的低（奇数）寄存器

　　unsigned _loll （long long src）; 返回longlong寄存器的低（奇数）寄存器

　　double _ltod （__int40_t src）; 把一个__int40_t寄存器解释成一个double寄存器

　　double _lltod （long long src）; 把一个longlong寄存器解释成一个double寄存器

　　int _mpy （int src1， int src2）;MPYSrc1和src2相乘，操作数默认为有符号的

　　int _mpyus （unsigned src1， int src2）;MPYUS无符号src1和有符号src2相乘，S是用来那个是有符号的操作数，当两个操作数都是有符号的或者无符号的

　　int _mpysu （int src1， unsigned src2）;MPYSU同上

　　unsigned _mpyu （unsigned src1，unsigned src2）;MPYU同上上上，默认为无符号

　　int _mpyh （int src1， int src2）;MPYH同上，区别见图示

　　int _mpyhus （unsigned src1， int src2）;MPYHUS

　　int _mpyhsu （int src1， unsigned src2）;MPYHSU

　　unsigned _mpyhu （unsigned src1，unsigned src2）;MPYHU

　　int _mpyhl （int src1， int src2）;MPYHL同上，区别见图示

　　int _mpyhuls （unsigned src1， int src2）;MPYHULS

　　int _mpyhslu （int src1， unsigned src2）;MPYHSLU

　　unsigned _mpyhlu （unsigned src1，unsigned src2）;MPYHLU

　　int _mpylh （int src1， int src2）;MPYLH

　　int _mpyluhs （unsigned src1， int src2）;MPYLUHS

　　int _mpylshu （int src1， unsigned src2）;MPYLSHU

　　unsigned _mpylhu （unsigned src1，unsigned src2）;MPYLHU

　　void _nassert （int src）; 不生成代码，告诉优化器一些事情

　　unsigned _norm （int src）;

　　unsigned _lnorm （__int40_t src）;NORM

　　返回src2的冗余的符号比特位的个数，具体见图示

　　int _sadd （int src1， int src2）;

　　long _lsadd （int src1， __int40_t src2）;SADD将src1和src2相加，且饱和其结果

　　int _sat （__int40_t src2）;SAT将一个40比特的long转换为一个32比特的有符号int，如有需要，对结果进行饱和

　　unsigned _set （unsigned src2， unsignedcsta ， unsigned cstb）;SET将src2中指定的区域置位1，指定的区域由csta和cstb指定

　　unsigned _setr （unit src2， int src1）;SET

　　int _smpy （int src1， int src2）;SMPY把src1的低16位和src2的低16位相乘

　　int _smpyh （int src1， int src2）;SMPYH高16位

　　int _smpyhl （int src1， int src2）;SMPYHL

　　int _smpylh （int src1， int src2）;SMPYLH

　　int _sshl （int src2， unsigned src1）;SSHL以src1操作数将src2左移，并且将结果饱和在32位

　　int _ssub （int src1， int src2）;

　　__int40_t _lssub （int src1， __int40_tsrc2）;SSUB从src1中减去src2，并饱和结果（src1-src2）

　　unsigned _subc （unsigned src1， unsignedsrc2）;SUBC有条件的减和左移（常用于除法）

　　int _sub2 （int src1， int src2）;SUB2把src1的高低16位分别减去src2的高低16位。任何低16位的借位不会影响高16位。

　　int _abs2 （int src）;ABS2计算16位的绝对值

　　int _add4 （int src1， int src2）;ADD4把src1和src2的4对8位数相加。不会进行饱和，进位不会影响其他的8位数

　　long long & _amem8 （void *ptr）;LDDW

　　STDW加载和存储8bytes，指针必须8byte对齐

　　const long long & _amem8_const （const void *ptr）;LDDW加载8bytes，指针必须8byte对齐

　　__float2_t & _amem8_f2（void * ptr）;LDDW

　　STDW加载和存储8bytes，指针必须8byte对齐，必须包含c6x.h

　　const __float2_t & _amem8_f2_const（void * ptr）;LDDW加载8bytes，指针必须8byte对齐，必须包含c6x.h

　　double & _amemd8 （void *ptr）;LDDW

　　STDW

　　const double & _amemd8_const （const void *ptr）;LDDW

　　int _avg2 （int src1， int src2）;AVG2计算每对有符号16位置的平均值

　　unsigned _avgu4 （unsigned， unsigned）;AVGU4计算每对有符号8位数的平均值

　　unsigned _bitc4 （unsigned src）;BITC4统计每个8位的比特位是1的个数，写入结果对应位置

　　unsigned _bitr （unsigned src）;BITR翻转比特位的顺序

　　int _cmpeq2 （int src1， int src2）;CMPEQ2比较每16位的值是否相等，结果放入dst的最低2位

　　int _cmpeq4 （int src1， int src2）;CMPEQ4比较每8位的值是否相等，结果放入dst的最低4位，相等置1，否则为0

　　int _cmpgt2 （int src1， int src2）;CMPGT2每16位有符号比较，src1》src2，置为1；否则置为0。结果放入dst的最低2位

　　unsigned _cmpgtu4 （unsigned src1，unsigned src2）;CMPGTU4每8位无符号比较，src1》src2，置为1；否则置为0。结果放入dst的最低4位

　　unsigned _deal （unsigned src ）;DEAL将src中的比特位的奇数位和偶数位抽出来进行重组，偶数位放在低的16位，奇数位放在高的16位

　　int _dotp2 （int src1， int src2）;

　　__int40_t _ldotp2 （int src1， int src2）;DOTP2

　　DOTP2将src1中的和src2中的16位有符号对进行点积，结果被写成有符号32位int或者符号扩展为64位

　　int _dotpn2 （int src1， int src2）;DOTPN2将src1和src2中的16位有符号数进行点积相减

　　int _dotpnrsu2 （int src1， unsigned src2）;DOTPNRSU2Src1和src2的高16位的点积减去低16位的点积。Src1中的数被当做有符号，src2中的数被当做无符号，再加上2^15，结果再符号右移16位

　　int _dotprsu2 （int src1， unsigned src2）;DOTPRSU2Src1和src2的高16位的点积加上低16位的点积。Src1中的数被当做有符号，src2中的数被当做无符号，再加上2^15，结果再符号右移16位

　　int _dotpsu4 （int src1， unsigned src2）;DOTPSU4将src1和src2的每8位进行相乘再求和，src1的每8位数被当做有符号，src2的每8位数被当做无符号

　　unsigned _dotpu4 （unsigned src1，unsigned src2）;DOTPU4都被当做无符号的

　　int _gmpy4 （int src1， int src2）;GMPY4将src1和src2的4个无符号进行伽罗瓦域的乘法

　　int _max2 （int src1， int src2）;MAX2将src1和src2的2个有符号16位整数比较，取较大值

　　int _min2 （int src1， int src2）;MIN2将src1和src2的2个有符号16位整数比较，取较小值

　　unsigned _maxu4 （unsigned src1，unsigned src2）;MAXU4将src1和src2的4个无符号8位整数比较，取较大值

　　unsigned _minu4 （unsigned src1，unsigned src2）;MINU4将src1和src2的4个无符号8位整数比较，取较小值

　　ushort & _mem2 （void * ptr）;LDB/LDB

　　STB/STB加载和存储2byte，不需要对齐

　　const ushort & _mem2_const （const void * ptr）;LDB/LDB加载2byte，不需要对齐

　　unsigned & _mem4 （void * ptr）;LDNW

　　STNW加载和存储4byte，不需要对齐

　　const unsigned & _mem4_const （const void * ptr）;LDNW加载4byte，不需要对齐

　　long long & _mem8 （void * ptr）;LDNDW

　　STNDW加载和存储8byte，不需要对齐

　　const long long & _mem8_const （const void * ptr）;LDNDW加载8byte，不需要对齐

　　double & _memd8 （void * ptr）;LDNDW

　　STNDW加载和存储8byte，不需要对齐

　　const double & _memd8_const （const void * ptr）;LDNDW加载8byte，不需要对齐

　　long long _mpy2ll （int src1， int src2）;MPY2将src1和src2中的2个有符号16位分别相乘，将2个32位的结果写入longlong中

　　long long _mpyhill （int src1， int src2）;MPYHI将src1中高16位作为1个有符号16位乘以src2的有符号32位，结果写入longlong的低48位

　　long long _mpylill （int src1， int src2）;MPYLI将src1中低16位作为1个有符号16位乘以src2的有符号32位，结果写入longlong的低48位

　　int _mpyhir （int src1， int src2）;MPYHIR将src1的高16位作为一个16位有符号乘以src2的有符号32位。乘积利用round模式通过加2^14转成32位，最后再右移15位

　　int _mpylir （int src1， int src2）;MPYLIR将src1的低16位作为一个16位有符号乘以src2的有符号32位。乘积利用round模式通过加2^14转成32位，最后再右移15位

　　long long _mpysu4ll （int src1， unsignedsrc2）;MPYSU4将src1的4个8位有符号乘src2的4个8位无符号，得到4个16位有符号，组成一个64位

　　long long _mpyu4ll （unsigned src1，unsigned src2）;MPYU4将src1和src2的4个无符号8位相乘，得到4个无符号16位组成一个64位的数

　　int _mvd （int src2 ）;MVD将src2的数据移入返回值中，利用了乘法流水线（延迟）

　　unsigned _pack2 （unsigned src1，unsigned src2）;PACK2

　　unsigned _packh2 （unsigned src1，unsigned src2）;PACKH2

　　unsigned _packh4 （unsigned src1，unsigned src2）;PACKH4

　　unsigned _packl4 （unsigned src1，unsigned src2）;PACKL4

　　unsigned _packhl2 （unsigned src1，unsigned src2）;PACKHL2

　　unsigned _packlh2 （unsigned src1，unsigned src2）;PACKLH2

　　unsigned _rotl （unsigned src1， unsignedsrc2）;ROTL按照src1的最低5位的数去左移src2的32位，src1中剩下的高的5-31位被忽略

　　int _sadd2 （int src1， int src2）;SADD2将src1和src2中的2个16位有符号数相加，生成2个16有符号数并且是饱和过的。

　　int _saddus2 （unsigned src1， int src2）;SADDUS2将src1中的2个无符号16位数和src中的2个16位有符号数相加，得到2个无符号16位数

　　unsigned _saddu4 （unsigned src1，unsigned src2）;SADDU4将src1和src2中的4个无符号8位数相加

　　unsigned _shfl （unsigned src2）;SHFL将src2的高16和低16位进行交织

　　unsigned _shlmb （unsigned src1，unsigned src2）;SHLMB将src2左移1byte，然后将src1的最高位充入src2左移后多出来的位置

　　unsigned _shrmb （unsigned src1，unsigned src2）;SHRMB将src2右移1byte，然后将src1的最低位充入src2右移后多出来的位置

　　int _shr2 （int src1， unsigned src2）;SHR2将src2的2个16位有符号数分别右移，右移的位数由src1的低5位决定，多出的位置由符号位扩展

　　unsigned shru2 （unsigned src1， unsignedsrc2）;SHRU2将src2的2个16位无符号数分别右移，右移的位数由src1的低5位决定，多出的位置由0扩展

　　long long _smpy2ll （int src1， int src2）;SMPY2将src1和src2中的2个有符号16位数相乘，然后左移1位，再进行饱和。

　　int _spack2 （int src1， int src2）;SPACK2将src1和src2中的1个有符号32位数进行饱和到有符号16位，然后把src1的饱和结果放入dst的高16位，src2的饱和结果放入dst的低16位

　　unsigned _spacku4 （int src1 ， int src2）;SPACKU4将src1和src2中的4个有符号16位数饱和成无符号8位数，

　　int _sshvl （int src2， int src1）;SSHVL将src2中的有符号32位数左移或右移，移位的数量由src1指定的比特数确定。

　　src1在［-31，31］之间，如果src1为正，src2则左移；如果src1为负，src2右移|src1|且符号位扩展

　　int _sshvr （int src2， int src1）;SSHVR将src2中的有符号32位数左移或右移，移位的数量由src1指定的比特数确定。

　　src1在［-31，31］之间，如果src1为正，src2则右移且是符号扩展；如果src1为负，src2左移|src1|

　　int _sub4 （int src1， int src2）;SUB4将src1和src2中的4个8位数相减，不进行饱和

　　int _subabs4 （int src1， int src2）;SUBABS4将src1和src2中的4个无符号8位相减求绝对值

　　unsigned _swap4 （unsigned src）;SWAP4将src的4个8位无符号数按图示换位置

　　unsigned _unpkhu4 （unsigned src）;UNPKHU4扩展0

　　unsigned _unpklu4 （unsigned src）;UNPKLU4扩0

　　unsigned _xpnd2 （unsigned src）;XPND2按src的最低2位进行扩展，bit1扩展高16位，bit0扩展低16位

　　unsigned _xpnd4 （unsigned src）;XPND4按src的最低4位进行扩展

　　long long _addsub （int src1， int src2）;ADDSUB平行做2步：

　　1、src2+src1-》dst_o

　　2、src1-src2-》dst_e

　　long long _addsub2 （int src1， int src2）;ADDSUB216位有符号

　　ADD2:src2的高、低16位+src1的高、低16位-》dst_o

　　SUB2： src1的高、低16位-src2的高、低16位-》dst_e

　　long long _cmpy （unsigned src1，unsigned src2）;CMPY有符号16位

　　Src1和src2的高16位的点积-src1和src2的低16位点积-》dst_o

　　饱和（src1和src2的高16位的点积+src1和src2的低16位点积）-》dst_e

　　unsigned _cmpyr （unsigned src1，unsigned src2）;CMPYR

　　unsigned _cmpyr1 （unsigned src1，unsigned src2 ）;CMPYR1

　　long long _ddotp4 （unsigned src1，unsigned src2）;DDOTP4没有饱和

　　long long _ddotph2 （long long src1，unsigned src2）;DDOTPH2

　　long long _ddotpl2 （long long src1，unsigned src2）;DDOTPL2

　　unsigned _ddotph2r （long long src1，unsigned src2）;DDOTPH2R

　　unsigned _ddotpl2r （long long src1，unsigned src2）;DDOTPL2R

　　long long _dmv （int src1， int src2）;DMV将两个寄存器移入一个寄存器一次性的

　　long long _dpack2 （unsigned src1，unsigned src2）;DPACK2

　　long long _dpackx2 （unsigned src1，unsigned src2）;DPACKX2

　　__float2_t _fmdv_f2（float src1， floatsrc2）DMV

　　unsigned _gmpy （unsigned src1，unsigned src2）;GMPY伽罗瓦域上的乘法

　　long long _mpy2ir （int src1， int src2）;MPY2IR进行16位乘32位。

　　将src1的高16位和低16位当做有符号16位；将src2的值当做有符号32位。

　　乘积通过加上2^14round到32位，然后结果右移15位。

　　2个结果的低32位写入dst_o:dst_e

　　int _mpy32 （int src1， int src2）;MPY32进行32位乘32位。都是有符号的，64位结果中的低32位写入dst

　　long long _mpy32ll （int src1， int src2）;MPY3232位有符号数×32位有符号数，有符号的64位结果被写入dst

　　long long _mpy32su （int src1， int src2）;MPY32SUsrc1有符号32位×src2无符号32位=dst有符号64位

　　long long _mpy32us （unsigned src1， intsrc2）;MPY32USsrc1无符号32位×src2有符号32位=dst有符号64位

　　long long _mpy32u （unsigned src1，unsigned src2）;MPY32Usrc1无符号32位×src2无符号32位=dst无符号64位

　　int _rpack2 （int src1， int src2）;RPACK2

　　long long _saddsub （unsigned src1，unsigned src2）;SADDSUB并行进行：

　　1、饱和（src1+src2）-》dst_o

　　2、饱和（src1-src2）-》dst_e

　　long long _saddsub2 （unsigned src1，unsigned src2）;SADDSUB2并行进行SADD2和SSUB2指令

　　long long _shfl3 （unsigned src1， unsignedsrc2）;SHFL3如图，生成一个longlong

　　int _smpy32 （int src1， int src2）;SMPY3232位有符号×32位有符号，64位的结果左移1位然后饱和，然后将之后的结果的高32位写入dst

　　int _ssub2 （unsigned src1， unsignedsrc2）;SSUB2Src1中的2个16位有符号-src2中的2个有符号16位，结果进行饱和

　　unsigned _xormpy （unsigned src1，unsigned src2）;XORMPY加瓦罗域乘法

　　int _dpint （double src）;DPINT将double转成int（round）

　　__int40_t _f2tol（__float2_t src）; 将一个__float2_t解释成一个__int40

　　__float2_t _f2toll（__float2_t src）; 将一个__float2_t解释成一个longlong

　　double _fabs （double src）;ABSDP将src的绝对值放入dst。

　　float _fabsf （float src）;ABSSP

　　__float2_t _lltof2（long long src）; 将一个longlong解释成一个__float2_t

　　__float2_t _ltof2（__int40_t src）; 将一个__int40解释成一个__float2_t

　　__float2_t & _mem8_f2（void * ptr）;LDNDW

　　STNDW从内存里加载一个64位值

　　const __float2_t & _mem8_f2_const（void * ptr）;LDNDW

　　STNDW

　　long long _mpyidll （int src1， int src2）;MPYIDSrc1×src2-》dst

　　double_mpysp2dp （float src1， float src2）;MPYSP2DPSrc1×src2-》dst

　　double_mpyspdp （float src1， doublesrc2）;MPYSPDPSrc1×src2-》dst

　　double _rcpdp （double src）;RCPDP64位double倒数近似值放入dst

　　float _rcpsp （float src）;RCPSP32位float的倒数近似值

　　double _rsqrdp （double src）;RSQRDP64位double的平方根倒数近似值

　　float _rsqrsp （float src）;RSQRSP32位float的平方根倒数近似值

　　int _spint （float）;SPINTFloat转为int

　　ADDDP2个double相加

　　ADDSP2个float相加

　　AND位与

　　ANDN与后取反

　　MPYSP2个float相乘

　　OR位或

　　SUBDP2个double相减

　　SUBSP2和float相减

　　XOR异或

　　__x128_t _ccmatmpy （long long src1，__x128_t src2）;CMATMPY

　　long long _ccmatmpyr1 （long long src1，__x128_t src2）;CCMATMPYR1

　　long long _ccmpy32r1 （long long src1，long long src2）;CCMPY32R1

　　__x128_t _cmatmpy （long long src1，__x128_t src2）;CMATMPY

　　long long _cmatmpyr1 （long long src1，__x128_t src2）;CMATMPYR1

　　long long _cmpy32r1 （long long src1，long long src2）;CMPY32R1

　　__x128_t _cmpysp （__float2_t src1，__float2_t src2）;CMPYSP

　　double _complex_conjugate_mpysp （double src1， double src2）;CMPYSP

　　DSUBSP

　　double _complex_mpysp （double src1，double src2）;CMPYSP

　　DADDSP

　　int _crot90 （int src）;CROT90复数的90度旋转

　　int _crot270 （int src）;CROT270复数的270度旋转

　　long long _dadd （long long src1， long longsrc2）;DADDSrc1的2个32位有符号数+src2的2个32位有符号数

　　long long _dadd2 （long long src1， long long src2）;DADD24路有符号16位相加

　　__float2_t _daddsp （__float2_t src1，__float2_t src2）;DADDSP

　　long long _dadd_c （scst5 immediate src1，long long src2）;DADD2路float加法

　　long long _dapys2 （long long src1， long long src2）;DAPYS2

　　long long _davg2 （long long src1， long long src2）;DAVG2有符号16位

　　long long _davgnr2 （long long src1， long long src2）;DAVGNR2有符号16位，无round模式

　　long long _davgnru4 （long long src1，long long src2）;DAVGNRU4无符号8位，无round模式

　　long long _davgu4 （long long src1， long long src2）;DAVGU4无符号8位

　　long long _dccmpyr1 （long long src1，long long src2）;DCCMPYR1

　　unsigned _dcmpeq2 （long long src1， long long src2）;DCMPEQ216位比较，相等返回1，不等返回0

　　unsigned _dcmpeq4 （long long src1， long long src2）;DCMPEQ48位比较，相等返回1，不等返回0

　　unsigned _dcmpgt2 （long long src1， long long src2）;DCMPGT216位比较，src1》src-》1，否则返回0

　　unsigned _dcmpgtu4 （long long src1，long long src2）;DCMPGTU48位比较，src1》src-》1，否则返回0

　　__x128_t _dccmpy （long long src1， long long src2）;DCCMPY

　　__x128_t _dcmpy （long long src1， long long src2）;DCMPY

　　long long _dcmpyr1 （long long src1， long long src2）;DCMPYR1

　　long long _dcrot90 （long long src）;DCROT90

　　long long _dcrot270 （long long src）;DCROT270

　　long long _ddotp4h （__x128_t src1，__x128_t src2 ）;DDOTP4H执行2个dotp4h，都是有符号的

　　long long _ddotpsu4h （__x128_t src1，__x128_t src2 ）;DDOTPSU4H执行2个dotpsu4h，一个有符号，一个无符号

　　__float2_t _dinthsp （int src）;DINTHSPSrc中的16位有符号数转成单精度浮点放入dst_e和dst_o中

　　__float2_t _dinthspu （unsigned src）;DINTHSPUSrc中的16位无符号数转成单精度浮点放入dst_e和dst_o中

　　__float2_t _dintsp（long long src）;DINTSPSrc中的有符号32位转成单精度浮点，放入dst_e和dst_o中

　　__float2_t _dintspu（long long src）;DINTSPUSrc中的无符号32位转成单精度浮点，放入dst_e和dst_o中

　　long long _dmax2 （long long src1， long long src2）;DMAX2对src1和src2中的16位有符号数比大小，将大的放入dst中

　　long long _dmaxu4 （long long src1， long long src2）;DMAXU4对src1和src2中的8位有符号数比大小，将大的放入dst中

　　long long _dmin2 （long long src1， long long src2）;DMIN2对src1和src2中的16位有符号数比大小，将小的放入dst中

　　long long _dminu4 （long long src1， long long src2）;DMINU4对src1和src2中的8位有符号数比大小，将小的放入dst中

　　__x128_t _dmpy2 （long long src1， long long src2）;DMPY2将src1和src2中的16位有符号数相乘，得到32位有符号数放入128位寄存器中

　　__float2_t _dmpysp （__float2_t src1，__float2_t src2）;DMPYSP

　　__x128_t _dmpysu4 （long long src1，long long src2）;DMPYSU4将src1中的8位有符号数乘以src2中的无符号8位，等到有符号16位

　　__x128_t _dmpyu2 （long long src1， long long src2）;DMPYU216位无符号数相乘，得到32位数放入128位寄存器中

　　__x128_t _dmpyu4 （long long src1， long long src2）;DMPYU48位无符号数相乘，得到有符号16位结果

　　long long _dmvd （long long src1，unsigned src2 ）;DMVD将2个寄存器移入一个寄存器中。依次进行2次移动，当处理很多的double word时很有用。减轻寄存器压力

　　int _dotp4h （long long src1， long longsrc2 ）;DOTP4H进行两个系列的16位值的点积

　　long long _dotp4hll （long long src1， long long src2 ）;DOTP4H返回值不同

　　int _dotpsu4h （long long src1， long longsrc2）;DOTPSU4HSrc1中被当做有符号16位，src2被当做无符号16位，得到32位结果

　　long long _dotspu4hll （long long src1，long long src2）;DOTPSU4HSrc1中被当做有符号16位，src2被当做无符号16位，得到64位结果

　　long long _dpackh2 （long long src1， long long src2）;DPACKH2

　　long long _dpackh4 （long long src1， long long src2）;DPACKH4并行执行2个PACKH4

　　long long _dpacklh2 （long long src1， long long src2）;DPACKLH2

　　long long _dpacklh4 （unsigned src1，unsigned src2）;DPACKLH4并行执行PACKH4和PACKL4

　　long long _dpackl2 （long long src1， long long src2）;DPACKL2

　　long long _dpackl4 （long long src1， long long src2）;DPACKL4并行执行2个PACKL4

　　long long _dsadd （long long src1， long long src2）;DSADD将src1中的2个有符号32位数加上src2中的2个有符号32位数，结果进行饱和

　　long long _dsadd2 （long long src1， long long src2）;DSADD2结果饱和到［-2^15 2^15］

　　long long _dshl （long long src1， unsignedsrc2）;DSHL将longlong中的2个32位左移，用0补位（有符号32位）

　　long long _dshl2 （long long src1，unsigned src2）;DSHL2将longlong中的4个16位左移，用0补位（有符号16位）

　　long long _dshr （long long src1， unsignedsrc2）;DSHR右移，符号位补位（有符号32位）

　　long long _dshr2 （long long src1，unsigned src2）;DSHR2右移，符号位补位（有符号16位）

　　long long _dshru （long long src1，unsigned src2）;DSHRU右移，0补位（无符号32位）

　　long long _dshru2 （long long src1，unsigned src2）;DSHRU2右移，0补位（无符号16位）

　　__x128_t _dsmpy2 （long long src1， long long src2）;DSMPY2见图示

　　long long _dspacku4 （long long src1， long long src2）;DSPACKU4并行进行2个SPACK4

　　long long _dspint （__float2_t src）;DSPINT将src中的2个单精度数转成2个整型

　　unsigned _dspinth （__float2_t src）;DSPINTH将src_e和src_o的两个单精度浮点数转陈高个有符号的16位整数

　　long long _dssub （long long src1， long long src2）;DSSUB将src1中的2个32位有符号数减src2中的2个32位有符号数，得到的结果进行饱和［-2^31 （2^31）-1］

　　long long _dssub2 （long long src1， long long src2）;DSSUB24个16位有符号数相减，结果进行饱和［-2^15 （2^15）-1］

　　long long _dsub （long long src1， long longsrc2）;DSUB不饱和

　　long long _dsub2 （long long src1， long long src2）;DSUB2不饱和

　　__float2_t _dsubsp （__float2_t src1，__float2_t src2）;DSUBSP32位单精度数相减

　　long long _dxpnd2 （unsigned src）;DXPND2

　　long long _dxpnd4 （unsigned src）;DXPND4

　　__float2_t _fdmvd_f2（float src1， floatsrc2）;DMVD见MVD

　　int _land （int src1， int src2）;LAND逻辑与

　　int _landn （int src1， int src2）;LANDN

　　int _lor （int src1， int src2）;LOR逻辑或

　　void _mfence（）;MFENCE延迟取指令流水线一直到内存系统的busy标志降低

　　double_mpysp2dp （float src1， float src2）;MPYSP2DP将2个float相乘得到1个double结果

　　double_mpyspdp （float src1， doublesrc2）;MPYSPDP1个float×1个double得到1个double

　　long long _mpyu2 （unsigned src1，unsigned src2 ）;MPYU22个无符号16位数×2个无符号16位数得到2个无符号32位数

　　__x128_t _qmpy32 （__x128_t src1，__x128_t src2）;QMPY324路：32位有符号×32位有符号，结果的低32位放入dst

　　__x128_t _qmpysp （__x128_t src1，__x128_t src2）;QMPYSP

　　__x128_t _qsmpy32r1 （__x128_t src1，__x128_t src2）;QSMPY32R14路：有符号32位×有符号32位，得到32位。和QMOY32的区别是饱和round

　　unsigned _shl2 （unsigned src1， unsignedsrc2）;SHL22个有符号16位，左移。Src2的低4位是移动的位数。结果也是当做有符号16位

　　long long _unpkbu4 （unsigned src）;UNPKBU4将无符号8位扩成无符号16位

　　long long _unpkh2 （unsigned src）;UNPKH2有符号16位符号扩展

　　long long _unpkhu2 （unsigned src）;UNPKHU2无符号16位进行0扩展

　　long long _xorll_c （scst5 immediate src1，long long src2）;XOR逻辑异或
```