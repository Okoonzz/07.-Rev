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
