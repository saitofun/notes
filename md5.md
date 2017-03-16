# md5

## summary

* 512bits (group);
* 512bits = 16 *32 (sub_group);
* output: 4 * 32 (sub_group);
* output: 128bits hash value;

## procedure

### padding

* condition: N * 512 + 448 (N = Natural Number)
* step1 : padding a 1 bit and several 0 bit, until meet the condition.
* step2 : padding 64 bits to indicate the lenght of previous information.
* output : the total length after md5 encoding is `N * 512 + 448 + 64`

### initialization

> initial 128 bits numbers are
`
> A = 0x01234567
> B = 0x89ABCDEF
> C = 0xFEDCBA98
> D = 0x76543210
`

### data processing

#### pre-defination
> F(x, y, z) = (x & y) | ((~x) & z)
> G(x, y, z) = (x & y) | ((~z) & y)
> H(x, y, z) = x ^ y ^ z
> I(x, y, z) = y ^ (x | (~z))

> Mj : the j(rd) sub-group (0 ~ 15)
> Ti : a constant value, `4294967296 * abs(sin(i))`

> FF(a, b, c, d, Mj, s, Ti) --> a = b + ((a + F(b, c, d) + Mj + Ti) << s)
> GG(a, b, c, d, Mj, s, Ti) --> a = b + ((a + G(b, c, d) + Mj + Ti) << s)
> HH(a, b, c, d, Mj, s, Ti) --> a = b + ((a + H(b, c, d) + Mj + Ti) << s)
> II(a, b, c, d, Mj, s, Ti) --> a = b + ((a + I(b, c, d) + Mj + Ti) << s)

`note: << operation is circularly left shift, not left shift`

#### round 1
FF(a, b, c, d, M0 ,  7, 0xd76aa478)
FF(d, a, b, c, M1 , 12, 0xe8c7b756)
FF(c, d, a, b, M2 , 17, 0x242070db)
FF(b, c, d, a, M3 , 22, 0xc1bdceee)
FF(a, b, c, d, M4 ,  7, 0xf57c0faf)
FF(d, a, b, c, M5 , 12, 0x4787c62a)
FF(c, d, a, b, M6 , 17, 0xa8304613)
FF(b, c, d, a, M7 , 22, 0xfd469501)
FF(a, b, c, d, M8 ,  7, 0x698098d8)
FF(d, a, b, c, M9 , 12, 0x8b44f7af)
FF(c, d, a, b, M10, 17, 0xffff5bb1)
FF(b, c, d, a, M11, 22, 0x895cd7be)
FF(a, b, c, d, M12,  7, 0x6b901122)
FF(d, a, b, c, M13, 12, 0xfd987193)
FF(c, d, a, b, M14, 17, 0xa679438e)
FF(b, c, d, a, M15, 22, 0x49b40821)

#### round 2
GG(a, b, c, d, M1 ,  5, 0xf61e2562)
GG(d, a, b, c, M6 ,  9, 0xc040b340)
GG(c, d, a, b, M11, 14, 0x265e5a51)
GG(b, c, d, a, M0 , 20, 0xe9b6c7aa)
GG(a, b, c, d, M5 ,  5, 0xd62f105d)
GG(d, a, b, c, M10,  9, 0x02441453)
GG(c, d, a, b, M15, 14, 0xd8a1e681)
GG(b, c, d, a, M4 , 20, 0xe7d3fbc8)
GG(a, b, c, d, M9 ,  5, 0x21e1cde6)
GG(d, a, b, c, M14,  9, 0xc33707d6)
GG(c, d, a, b, M3 , 14, 0xf4d50d87)
GG(b, c, d, a, M8 , 20, 0x455a14ed)
GG(a, b, c, d, M13,  5, 0xa9e3e905)
GG(d, a, b, c, M2 ,  9, 0xfcefa3f8)
GG(c, d, a, b, M7 , 14, 0x676f02d9)
GG(b, c, d, a, M12, 20, 0x8d2a4c8a)

#### round 3
HH(a, b, c, d, M5 ,  4, 0xfffa3942)
HH(d, a, b, c, M8 , 11, 0x8771f681)
HH(c, d, a, b, M11, 16, 0x6d9d6122)
HH(b, c, d, a, M14, 23, 0xfde5380c)
HH(a, b, c, d, M1 ,  4, 0xa4beea44)
HH(d, a, b, c, M4 , 11, 0x4bdecfa9)
HH(c, d, a, b, M7 , 16, 0xf6bb4b60)
HH(b, c, d, a, M10, 23, 0xbebfbc70)
HH(a, b, c, d, M13,  4, 0x289b7ec6)
HH(d, a, b, c, M0 , 11, 0xeaa127fa)
HH(c, d, a, b, M3 , 16, 0xd4ef3085)
HH(b, c, d, a, M6 , 23, 0x04881d05)
HH(a, b, c, d, M9 ,  4, 0xd9d4d039)
HH(d, a, b, c, M12, 11, 0xe6db99e5)
HH(c, d, a, b, M15, 16, 0x1fa27cf8)
HH(b, c, d, a, M2 , 23, 0xc4ac5665)

#### round 4
II(a, b, c, d, M0 ,  6, 0xf4292244)
II(d, a, b, c, M7 , 10, 0x432aff97)
II(c, d, a, b, M14, 15, 0xab9423a7)
II(b, c, d, a, M5 , 21, 0xfc93a039)
II(a, b, c, d, M12,  6, 0x655b59c3)
II(d, a, b, c, M3 , 10, 0x8f0ccc92)
II(c, d, a, b, M10, 15, 0xffeff47d)
II(b, c, d, a, M1 , 21, 0x85845dd1)
II(a, b, c, d, M8 ,  6, 0x6fa87e4f)
II(d, a, b, c, M15, 10, 0xfe2ce6e0)
II(c, d, a, b, M6 , 15, 0xa3014314)
II(b, c, d, a, M13, 21, 0x4e0811a1)
II(a, b, c, d, M4 ,  6, 0xf7537e82)
II(d, a, b, c, M11, 10, 0xbd3af235)
II(c, d, a, b, M2 , 15, 0x2ad7d2bb)
II(b, c, d, a, M9 , 21, 0xeb86d391)

### output

a += A;
b += B;
c += C;
d += D;
output : abcd(joining-up)
