# BYUCTF2023

Tải file [tại đây.](https://github.com/Okoonzz/07.-Rev/tree/File/BYUCTF)

## Chain
Khi mở file này, nhận thấy được rằng được viết và complie trên macbook bởi mã trả về toàn ARM =))

![](https://hackmd.io/_uploads/H1dMaC38n.png)

Đọc qua mã giả trả về thấy được rằng logic bài này khá đơn giản, chỉ việc xor và trả lại kết quả. Nhưng một điều đáng chú ý ở thử thách này đó chính là `(_BYTE *)off_210F4`. Khi đọc qua, nhận thấy rằng đây là một địa chỉ của offset nào đó. Tiến hành kiểm tra thì thấy rằng nó dẫn đến một hàm `sub_105AC` và hàm này có địa chỉ tại `0x105AC`. Khi vào được hàm này kiểm tra thì đây chính là hàm đó.

![](https://hackmd.io/_uploads/BJlRaA3Uh.png)

Ở code C ta có được như `*(ptr + i) = ptr[i]` với ptr chính là con trỏ trỏ đến một địa chỉ nào đó. Thì đoạn code này cũng tương tự, thực chất nó chính là `dword_210F4[dword_21040[i]]`. Cuối cùng, chỉ cần tìm ra được giá trị của "`dword_210F4`" là gì. Vì như đã nói ở trên nó chính là con trỏ trỏ đến một địa chỉ nào đó thì giá trị của "`dword_210F4`" chính là địa chỉ của hàm `sub_105AC`.

Sau khi có được tất cả các dữ kiện cần thiết, đây là script để giải bài này:

```
from pwn import *

key = p64(0xe59f1024e59f0024).hex()
key += p64(0xe1a01fa3e0413000).hex()
key += p64(0xe1b010c1e0811143).hex()
key += p64(0xe59f3010012fff1e).hex()
key += p64(0x012fff1ee3530000).hex()
key += p64(0x000210f8e12fff13).hex()

key_list = [int(key[i:i+2],16) for i in range(0, len(key), 2)]

array =  [0xc2,0x9c,0x65,0x83,0x95,0x66,0xfa,0x15,0x5e,0x58,0x2f,0x23,0xac,0x4f,0xa1,0x4c,0x7d,0x1e,0x69,0x80,0x8c,0x4a,0x26,0x5b,0x5f,0x91,0x30,0xcf,0xc0,0x4d,0x97,0x9b,0xba,0x20,0x77,0x4c,0xf5,0xef,0x97,0x96,0x31,0x30,0x8c,0xe2]

idx = [0x0e,0x03,0x1c,0x13,0x17,0x21,0x12,0x04,0x27,0x09,0x0d,0x22,0x1e,0x15,0x0b,0x24,0x1d,0x0a,0x18,0x2b,0x19,0x00,0x1b,0x2a,0x08,0x1f,0x20,0x25,0x02,0x1a,0x0c,0x29,0x07,0x05,0x11,0x28,0x14,0x16,0x23,0x0f,0x01,0x10,0x2c,0x06,0x26]

print(len(array))
print(len(idx))
print(key_list)
print(len(key_list))

for i in range(44):
    p = key_list[idx[i]]
    c = array[i]
    print(chr(p^c), end='')
print()
```

# GO

![](https://hackmd.io/_uploads/B1tlDkTI3.png)

Ở đoạn này, ta sẽ hiểu được rẳng đầu tiên in một chuỗi có nội dung là "Password: " ra màn hình. Sau đó lấy dữ liệu từ người dùng nhập vào và khi người dùng nhập vào thì dữ liệu sẽ được lưu tại `[rdx]` còn `[rdx+8]` chính là len của dữ liệu nhập vào. Vì đã biết trước đó là rax thường xuyên dùng để làm đối số truyền vào hàm trong golang, nên có thể hiểu đoạn này chính là cho dữ liệu vào MD5 sau đó qua một số lần kiểm tra. Tiếc là mình chỉ có thể dịch được kiểm tra đầu tiên chính là kiểm tra độ dài sau khi băm còn lần kiểm tra thứ hai thì mình chịu. Có thể dùng patch nhưng mình chưa dùng cái này bao giờ. 

![](https://hackmd.io/_uploads/BJSht16Uh.png)

![](https://hackmd.io/_uploads/BkBTYk6Ln.png)

Tiếp tục, tại đây nhận ra rằng nó có hàm "Atoi" và "rand seed" nên khả năng cao là nó lấy dãy số "`27443509682471009008637982892046`" để làm seed và sau đó chuỗi "`A*4D/nt5ayWF_5o~,L:$PKpq`" được lưu vào `var_78`, cuối cùng sạch sẽ các thanh ghi chuẩn bị vào một vòng lặp.

Ở trong vòng lặp này so sánh idx với **0x18**. Sau đó một hàm "rand Intn" được gọi và có đối số là **0x5E**. Điều đáng chú ý rẳng `xor     rbx, rax` mà `rbx` ở đây chính là chuỗi đã được chuyển vào còn `rax` chính là sau kết quả của "rand Int chuyển ra" (đoạn này debug sẽ thấy). 

Qua đó, có thể đoán được rẳng chuỗi `27443509682471009008637982892046` được làm seed cho mỗi lần random trong khoảng **0x5E** sau đó sẽ xor với từng ký tự được chuyển vào `var_78`

> Ở đây, nếu ta phân tích kỹ thì sẽ thấy được thanh `rcx` được thêm vào để tính toán địa chỉ, có thể sẽ gây khó hiểu. Nhưng hãy nhìn lên đoạn di chuyển `var_90` vào `rcx`, nhưng `var_90` này được load từ `rax` mà `rax` ở đoạn code này được xem như là phần tử idx nên tương tự `rsp` được thêm vào để tính toán lấy địa chỉ từng ký tự. Hơn nữa đoạn code còn lấy từng "byte" nên cũng được xem là từng ký tự.

Và cuối cùng đây là script để giải bài này:

```
package main

import(
	"math/rand"
	"fmt"
	"strconv"
)

func main(){
	n, _:= strconv.ParseInt("27443509682471009008637982892046", 10, 64)
	rand.Seed(n)
	for i := 0; i < 0x18; i++{
		fmt.Println(rand.Intn(0x5e))
	}
}

//////////////////////////////////////////////////////////////////////////////

ran = [35, 83, 65, 39, 91, 8, 15, 66, 0, 18, 50, 25, 42, 69, 48, 10, 67, 19, 93, 20, 15, 44, 64, 12]

xorr = "A*4D/nt5ayWF_5o~,L:$PKpq"

flag = ""
for i in range(len(xorr)):
    flag += chr(ran[i] ^ ord(xorr[i]))

print (flag)
```
Vẫn còn 1 bài nữa, nhưng bài này có kiến thức khá mới nên sẽ có trong bài viết về **BYU+DANTE**

=> Vậy qua những bài này, giúp nhận ra được cách hiểu của IDA về arr[sth[]] cũng như biểu diễn nó như thế nào và lấy nó ở đâu là chính xác. Và đôi khi không cần phải rev quá kĩ bởi vì phần chủ yếu để ra được flag lại nằm ở nhánh không cần thiết phải pass điều kiện trước đó.

# CALC
Đây là một bài khá hay ở giải uiu nên mình sẽ viết lại nó.

Trước khi hoàn thành bài này, ta hãy nhìn vào đoạn code sau:

```c++
bool doSTH(float a){
    return a < 0.0;
}

int main(){
    float a = -0.0;
    if (doSTH(a)) puts("Okk!!");
    else puts("Ooops");
}
//output: Ooops
```
Để hiểu rõ hơn về điều này có thể tham khảo [tiêu chuẩn IEE 754.](https://en.wikipedia.org/wiki/IEEE_754#Special_values)

Sau khi đọc được đoạn mã giả trả về từ IDA thì thấy được đoạn while như sau:

![](https://hackmd.io/_uploads/Sy8eE2S93.png)

Đoạn trên tức là chỉ cần tính toán có kết quả `8573.8567` thì sẽ trả về được `correct`. Nhưng khi tiến hành nhập vào nó chỉ trả về fake flag

![](https://hackmd.io/_uploads/SJzTNnBqn.png)

Lúc này, để ý rằng phía dưới vẫn còn một đoạn nữa chưa biết nó là gì 

![](https://hackmd.io/_uploads/ryb-S3S5n.png)

Ở đây, có một vòng lặp được lặp lại 368 lần. Nó thực hiện hàm `calculate` sau đó có thêm một hàm `gauntlet` để kiểm tra. Tiến hành phân tích xem hàm này có chức năng gì.

![](https://hackmd.io/_uploads/B1C_Bnrc2.png)

Nhìn qua thấy được rằng đây là một hàm kiểu `bool` kiểm tra xem có phải là "số âm, không phải số và Inf". Khi vào kiểm tra các hàm này thì chỉ có kiểm tra số âm thì sẽ có kiểu trả về thay đổi còn hai hàm còn lại không hề kiểm tra mà trả về thẳng là `0`.

Như đã đọc bài viết ở trên, bây giờ ta sẽ làm lại hàm `gauntlet` này để khi kiểm tra nó sẽ trả về 1.

*Đây là một script khá hay mình tham khảo được:*

```c++
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

typedef struct {
    long op;
    double a;
    double b;
} calculation;

double do_op(calculation calc) {
    if (calc.op == '%') {
        return fmod(calc.a, calc.b);
    } else if (calc.op == '+') {
        return calc.a + calc.b;
    } else if (calc.op == '-') {
        return calc.a - calc.b;
    } else if (calc.op == '*') {
        return calc.a * calc.b;
    } else if (calc.op == '/') {
        return calc.a / calc.b;
    } else if (calc.op == '^') {
        return pow(calc.a, calc.b);
    } else {
        printf("oops %ld\n", calc.op);
        return -1.0;
    }
}

int will_flip(double input) {
    if (signbit(input)) return 1;
    if (isnan(input)) return 1;
    if (isinf(input)) return 1;
    return 0;
}

int main() {
    unsigned char flag[47];
    *((unsigned long*)&flag[0]) = 0x10eeb90001e1c34b;
    *((unsigned long*)&flag[8]) = 0xcb382178a4f04bee;
    *((unsigned long*)&flag[16]) = 0xe84683ce6b212aea;
    *((unsigned long*)&flag[24]) = 0xa0f5cf092c8ca741;
    *((unsigned long*)&flag[32]) = 0x20a92860082772a1;
    *((unsigned int*)&flag[40]) = 0x35abb366;
    *((unsigned short*)&flag[44]) = 0xe9a4;
    flag[46] = 0;
    
    void* mem = malloc(0x2280);
    FILE* blob = fopen("./A.bin", "r");
    fread(mem, 0x2280, 1, blob);
    fclose(blob);
    int op_count = 0x2280 / sizeof(calculation);
    calculation* ops = (calculation* )mem;
    int flips = 0;
    for (int i = 0; i < op_count; ++i) {
        double ans = do_op(ops[i]);
        int wf = will_flip(ans);
        if (wf) {
            flag[i >> 3] ^= (1 << (7 - i % 8));
        }
        flips += wf;
    }
    printf("flips: %d\n", flips);
    printf("flag: %s\n", flag);
    return 0;
}
```

> Ở đây, tất cả chỉ mô phỏng lại thử thách này, chỉ khác hàm `gauntlet` tức là `will_flip` trong script sẽ được làm lại và khi nó gặp các trường hợp này sẽ trả về 1 hết tất cả. Còn phần `A.bin` chính là phần `calculate` trong vòng lặp for, tức là khi nhập input đúng ban đầu vào thì chương trình sẽ tính toán hàm này ta chỉ việc dump các byte ở đây ra và sẽ so sánh từng trường hợp với hàm `gauntlet` được làm lại và nó sẽ nằm ở `v78`.
