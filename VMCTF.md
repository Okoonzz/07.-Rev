# VM_Rust (Rồng kỹ thuật)

*Đây là bài của VKU, nhưng mình khá fail khi chưa nhìn ra được chỗ check của bài trong 5 tiếng cũng như những ngày sau đó :(((*

Vì đây là rust, nên mình không vội để vào IDA mà xem thử file này nó đang làm gì ở đây:

![](https://hackmd.io/_uploads/Sk1HN8dx6.png)

Có thể thấy được khá rõ đây là dạng check flag, được rồi bây giờ vào IDA xem nó đang làm gì ở đây. Đây là hàm main của chương trình:

![](https://hackmd.io/_uploads/rJ3F8LOe6.png)

Sau khi nhìn qua hàm được chạy tại `v4` và hàm ở `return`, dễ dàng nhận thấy được, hàm cần tập trung vào phân tích đó chính là hàm `sub_55FD70B55CD0`. Vì rust không phải như golang nhìn symbol là đoán được, mà bắt buộc phải debug mới hiểu được chương trình đang làm gì ở đây.

Chạy qua chương trình một lần thì thấy được rằng đây là một dạng `vm` và các `ins, op, data` được lấy từ file `bytecode`. Bây giờ ta sẽ tiến hành đào sâu vào chương trình.

Đầu tiên, ta sẽ đặt bp tại điểm lấy những byte đầu tiên của `bytecode`:

![](https://hackmd.io/_uploads/BJ6u_Udl6.png)

Chạy qua chương trình vài lần có thể dễ dàng nhận thấy được đây là điểm kết thúc của chương trình:

![](https://hackmd.io/_uploads/By0ndLOla.png)

Ta sẽ đặt bp tại đây để lấy được `input` sau khi có `input` ta sẽ trả lại bp ban đầu để thực hiện tính toán của chương trình. 

Sau khi nhập được input cũng như chạy qua chương trình vài lần mình thấy được rằng đầu tiên nó lấy 4bytes (little) ở `bytecode`, dữ liệu sau khi được lấy thì được lưu tại `v29` nên mình nghĩ `v29` chính là `op`. Ta sẽ viết lại một đoạn code mô tả lại quá trình lấy dữ liệu của `v29`. Đây là đoạn code mình đã viết được:

```python
import struct

with open("./bytecode", "rb") as f:
    by = f.read()

op = []
for idx in range(0, len(by), 4):
    op.append(int(struct.unpack("<I", by[idx:idx+4])[0]))

def controlidx(ins: int, idx: int, tmp: int):
    v27: int
    v28: int
    v27 = 1+idx if ins&0x10000000 != 0 else 0+idx
    v28 = v27+2
    if ins&0x20000000 == 0:
        v28 = v27
    if ins & 0x40000000 != 0:
        v28 = tmp
    return v28

test = []
idx = []
v22 = 0
v66 = 0
v24 = 0
i = 0
for _ in range(len(op)):
    v30 : int
    v31 : int
    v32 : int
    v27 : int
    v29 = op[i]
    test.append(v29)
    idx.append(i)
    v30 = v66
    if v29&0x40000 == 0:
        v24 = v66 + v22
    v31 = v22*v66
    if v29 & 0x80000 == 0:
        v31 = v24
    v32 = v22 ^ v66
    if v29 & 0x100000 == 0:
        v32 = v31
    v24 = 1 if v22==v66 else 0
    if v29 & 0x200000 == 0:
        v24 = v32
    if v29 & 0x4000000 != 0:
        v24 = op[i+1]
    i = controlidx(v29, i, v24)
```

> `test` chính là `v29`, còn `idx` chính là `i`. Mình muốn lưu lại nhưng thứ này vì trong quá trình debug mình thấy những thứ này lấy khá lộn xộn, nên mình sẽ tạm kiểm soát nó lấy như thế nào.

Sau khi viết một mô phỏng đơn giản ta lại tiếp tục debug chương trình. Trong quá trình debug mình sẽ thấy được những thứ như sau:

Đầu tiên chính là đoạn code mà chương trình load `input` vào:

![](https://hackmd.io/_uploads/HJuXsUdep.png)

Trong đoạn code này, ta cũng có thể bắt gặp được len của flag chính là 0x37, nhận ra được điều này cũng khá tốn nhiều thời gian đối với mình. Khi mình đã thử rất nhiều input thì thấy được đoạn code sau:

![](https://hackmd.io/_uploads/rJ6to8deT.png)

Đoạn này để gán `idx` cho mỗi lần load input vào. Dù có nhập `input` dài bao nhiêu thì lúc đầu tiên nó gán cũng là `0x37`, nên vì thế flag chính xác có len là 0x37.

Tiếp đến sau khi load được input vào thì đến đoạn code sau:

![](https://hackmd.io/_uploads/HJ3HlvOxT.png)

Ở đoạn code này mình đã comment khá rõ với hai biến `v17` và `v19`, chỉ còn `v20` mình đã không rõ nó đang làm gì bởi trong quá trình load hết input hay load từng char thì nó vẫn giữ nguyên một kết quả là `0`. Ban đầu mình nghĩ đây là nơi để lưu check, vì nó là `vm` nên có thể check theo bit hay gì đó chỉ cần nó nhảy một số là có thể detect được, nhưng mình đã hoàn toàn sai, tương tự cho các biến ở dưới (`v30` sẽ chứa các giá trị không đổi sau mỗi lần kiểm tra với từng chữ, `v22` giống như biến thông báo của `v17, v18`).

> Làm tới đoạn này đã mất gần 4 tiếng nhưng kết quả thu được vẫn chưa tìm ra được chỗ check

Sau đó mình lại chuyển hướng đến đoạn code mà sau mỗi lần chạy nó không nhảy đến:

![](https://hackmd.io/_uploads/B1m6-PdgT.png)

Đoạn code này, chính là lúc in ra thông báo kết quả, mình thấy cũng chả có gì đáng focus vào, và cuối cùng là đoạn code control idx:

![](https://hackmd.io/_uploads/HkSgMDOla.png)

Sau khi phân tích hết những thứ này mình đã mất hết 5 tiếng và cũng là thời gian cuối cùng hoàn thành làm bài. Nhưng vì deadline còn hạn nên mình đã ăn gian thêm vài ngày và chỉ thu được một tí kết quả cũng không mấy khả quan.

Khi viết disassembler mình đã thửu chạy hết và thấy được các `op` có đoạn data sau là rất nghi ngờ:

![](https://hackmd.io/_uploads/rkitMPuga.png)

Ở những dữ liệu trên, nếu debug nhiều lần thì sẽ thấy rất quen bởi đó là các giá trị của `v29` được load vào, nhưng những data cuối mình không thấy trong quá trình chạy nó được load vào `v29`, có thể đoán được đây chính là data enc. Nhưng nó enc kiểu nào ? Và check ở chỗ nào khi mỗi input được load vào ? Đó vẫn là câu hỏi mình chưa tìm được trong quá trình từ lúc deadline còn cho đến hết.

> Đã rust còn VM :<<

> Tiếp tục bài trên, mình đã tham khảo qua bài của anh Jinn, lúc đầu mình nghĩ mình đã sai, nhưng không mình đã đi đúng, nhưng đúng chỉ một phần, có một số điều cần phải chỉnh lại.

Lúc đầu mình chỉ viết 1 phần của vm nên không nhìn thấy hết được những data mà nó truy cập tới, mình đã chỉnh lại toàn bộ code như sau:

```python
import struct

notep = []

with open("./bytecode", "rb") as f:
    by = f.read()

byteCode = []
for idx in range(0, len(by), 4):
    byteCode.append(int(struct.unpack("<I", by[idx:idx+4])[0]))

pc = 0
op = 1

while pc < 1774:
    op = byteCode[pc]
    if op == 0xFFFFFFFF:
        pc += 1
        continue
    if op & 0x40000 != 0:
        notep.append(f"{hex(pc)}:       {hex(op)}  add val, val1, val2\n")
    if op & 0x80000 != 0:
        notep.append(f"{hex(pc)}:       {hex(op)}  mul val, val1, val2\n")
    if op & 0x20000 != 0:
        notep.append(f"{hex(pc)}:       {hex(op)}  shift value")
    if op & 0x100000 != 0:
        notep.append(f"{hex(pc)}:       {hex(op)}  xor val, val2, val1\n")
    if op & 0x200000 != 0:
        notep.append(f"{hex(pc)}:       {hex(op)}  cmp val2, val1 ; val = rescmp\n")

    if op & 4 != 0:
        notep.append(f"{hex(pc)}:       {hex(op)}        mov v25, v17\n")
    if  op & 0x20 != 0:
        notep.append(f"{hex(pc)}:       {hex(op)}        mov v25, v18\n")
    if op & 0x100 != 0:
        notep.append(f"{hex(pc)}:       {hex(op)}        mov v25, v19\n")

    if ( (op & 0x4000000) != 0 ):
        notep.append(f"{hex(pc)}:       {hex(op)}  mov v24, byteCode[pc+1]\n")
    if ( (op & 0x8000000) != 0 ):
        notep.append(f"{hex(pc)}:       {hex(op)}  mov v24, byteCode[v25]\n")
    if ( (op & 0x2000000) != 0 ):
        notep.append(f"{hex(pc)}:       {hex(op)}  get char input\n")
    if ( (op & 0x1000000) != 0 ):
        notep.append(f"{hex(pc)}:       {hex(op)}  try debug me\n")
    if (op & 0x10000000) != 0:
        pc +=1
    elif (op & 0x20000000) != 0:
        pc +=2
    else:
        pc += 1
    if ( (op & 0x40000000) != 0 ):
        notep.append(f"{hex(pc)}:       {hex(op)}  sth idk\n")

trans = "".join(notep)
with open ("huuh.txt", "w") as f:
    f.write(trans)
```

Đối với những bài `vm` có nhiều cách viết disassembler cho một bài `vm`, có thể vừa print vừa thực hiện thay đổi theo chương trình hoặc có thể print thẳng ra hoặc print những phần quan trọng,... Đối với bài này ta chỉ có duy nhất một cách đó là print thẳng ra, bởi nó có những phần code, mình cũng thực sự không hiểu tác dụng của nó nên việc code lại toàn bộ là không thể.

Ở đoạn code trên có những đoạn mặc dù IDA trả về không có nhưng tại sao vẫn lại thêm vào disassembler ?? 

Ở đây ta có đoạn code như sau:

![](https://hackmd.io/_uploads/HJJ_Gtnep.png)


Đoạn code này tương ứng với đoạn mã giả của IDA sau:

![](https://hackmd.io/_uploads/ByBM8u3xp.png)

Nếu debug nhiều lần thì sẽ thấy được toàn bộ chương trình hai biến `v31, v24` chỉ như `val` và nó cập nhật lại `val`.Giả sử ở đây nó đã thực hiện được `v24 = v67 + v22`, tiếp đến nó sẽ thực hiện phép nhân sau cùng nó check điều kiện nếu thỏa thì phép nhân được chuyển thành kết quả của phép cộng trước đó. Vậy điều này là hoàn toàn tương đương nếu điều kiện check được phủ lại thì `val` là nhân, còn không thì `val` vẫn giữ kết quả của phép cộng. Tương tự cho những trường hợp còn lại thì ta sẽ có được đoạn code trên. *(cách này mình tham khảo từ cách của anh Jinn rồi sau đó viết lại theo cách hiểu, còn lúc đầu mình làm theo cách vừa chạy theo chương trình vừa in)*

Sau khi có được đoạn disassembler, ta có một vài điểm đáng chú ý tới như là `xor` và `cmp`. Sở dĩ **tại sao lại đáng chú ý ?? Đáng nghi ngờ thế tại sao mình lại không nói ở phần trên ??**

Đáng chú ý là bởi vì đây là flag checker và nó là `vm` nên việc lồng ghép phức tạp ở thời gian 5 tiếng là điều khó xảy ra. Và vì check nên thường có thể là xor và toàn bài debug nhiều lần cũng chỉ nhìn thấy mỗi xor là nổi bật nhất nên không thể không nghi ngờ nó, và ở trên mình cũng đã nói có một đoạn dữ liệu giống như `enc` mà mình không biết nó lấy khi nào. Còn tại sao mình không nói ở phần trên là do một phần mình không biết nên nói chỗ nào là hợp lý, một phần vì mình debug không tới đoạn chương trình lấy dữ liệu nào xor, lúc mình chạy nó toàn lấy các giá trị 4bytes chứ không phải 1byte nên mình có nghi ngờ nhưng không đề cập đến.

Sau khi chuyển hết tập trung vào xor thì ta có thể có được một vài điều như sau:

![](https://hackmd.io/_uploads/BykmjOngp.png)

Ta thấy được nó có `pc = 0x306` và `op = byteCode[pc] = 0x10100040`. Vậy bây giờ ta sẽ đặt bp ngay tại buffer này rồi xem xem nó xor cái gì ở đây? Để tìm được đến buffer này, nếu ngồi debug thì có thể tới ~~nhưng đó là cách làm mình cho là ngu nhất :)))~~

Mình đã biết được ta có được công thức sau để tính toán chính xác địa chỉ cần tìm: `imagebase + offset = addrFind`

Vậy ngay lúc đầu tiên mình sẽ có được base của mảng này, sau đó chỉ cần cộng thêm offset ta có được như sau:

![](https://hackmd.io/_uploads/BkqHhOnlT.png)

Nhảy đến địa chỉ vừa tìm được ta có thể thấy heap chứa chính xác giá trị mà disassembler đã hiển thị:

![](https://hackmd.io/_uploads/S1VO3d2x6.png)

Bây giờ sẽ đặt bp ngay tại đây, sau đó lại chạy tiếp chương trình và trace theo xor. Input của mình là `flag{abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVW1}`

![](https://hackmd.io/_uploads/r1E-ad2ep.png)

Debug thêm vài lần sẽ thấy chính là từng char của input đem đi xor. Thế là xong, đây là script toàn bài:

```python
sth = [0x28, 0xec, 0x64, 0x92, 0x6d, 0x60, 0x49, 0x22, 0xaa, 0xa5, 0x8d, 0x34, 0xbd, 0xb1, 0xad, 0xf, 0x70, 0xcc, 0x8d, 0x30, 0x94, 0xe8, 0x9f, 0x33, 0x61, 0xab, 0xdd, 0x9, 0xa1, 0x90, 0x6, 0xa7, 0xdc, 0xd, 0x5d, 0xa6, 0xe6, 0x75, 0xf3, 0xe8, 0xb4, 0xb2, 0xbe, 0xc7, 0xd6, 0xfb, 0x2, 0x1a, 0xfa, 0xe8, 0x21, 0x16, 0x2a, 0xc3, 0x9d, 0x24]

enc = [78, 128, 5, 245, 22, 20, 33, 75, 217, 250, 250, 85, 206, 238, 204, 80, 38, 129, 210, 86, 251, 132, 244, 64, 62, 217, 190, 61, 254, 160, 116, 248, 143, 100, 57, 149, 185, 54, 155, 220, 218, 220, 219, 171, 137, 130, 50, 111, 165, 140, 68, 117, 67, 167, 248, 89, 27, 91, 51, 49, 109, 0, 27, 91, 51, 50, 109, 0, 27, 91, 48, 109, 0, 73, 32, 110, 101, 101, 100, 32, 116, 104, 101, 32, 102, 108, 97, 103, 58, 32, 0, 76, 71, 84, 77, 10, 0, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 97, 98, 99, 100, 101, 102, 102, 108, 97, 103, 123, 0, 89, 65, 65, 65, 89, 33, 33, 33, 0, 84, 114, 121, 32, 65, 103, 97, 105, 110, 0, 116, 114, 121, 32, 100, 101, 98, 117, 103, 103, 105, 110, 103, 32, 109, 101, 10, 0]

flag = ""

for a, b in zip(sth, enc):
    flag+=(chr(a^b))

print(flag)
```
=> Sau bài này, ta có thể thấy được bắt buộc `vm` phải viết được một disassembler hoàn chỉnh, sau đó muốn làm gì tiếp theo thì làm. Hơn thế nữa bài này còn nhận thấy được có thể đặt bp ở heap rồi debug không phải chỉ đặt được ở phần code.

# VM_Rust (Tết đến xuân về)

Vẫn là một bài VM nữa, nhưng bài lần này khá hay và học hỏi được nhiều thứ hơn bài trước.

Về bài ở rồng kỹ thuật thì mọi thứ khá rõ ràng khi có đầy đủ symbol và không cần phải detect quá nhiều, và may mắn khi mấu chốt của bài chỉ nằm ở xor.

Cũng tương tự như những bài VM khác việc tiếp cận một VM đầu tiên và bắt buộc đó là **phải viết được disassembler.** 

Đối với hầu hết những bài VM để viết được một disassembler chúng ta cần phải xác định được những thành phần: 
* **opcode (op)**
* **bytecode**
* **register**
* **mem**
* **instruction pointer (pc)**
* **stack**

Bây giờ sẽ đi vào chương trình chính, ở bài viết này tập trung nhiều vào cách viết disassembler hơn thay vì cách giải quyết thử thách.

Sẽ có những chương trình khi phân tích bằng IDA static thì nó sẽ hiển thị khác khi debug. Đây là toàn bộ đoạn code khi static:

![image](https://hackmd.io/_uploads/Sy_vrgHnp.png)

![image](https://hackmd.io/_uploads/Hy5_Bxr3T.png)

![image](https://hackmd.io/_uploads/S1wFBgH3p.png)

![image](https://hackmd.io/_uploads/HJEqBgS3a.png)

...

![image](https://hackmd.io/_uploads/BJwirer36.png)

Sau khi phân tích cũng như debug đây là đoạn code đã rename lại để dễ nhìn hơn:

![image](https://hackmd.io/_uploads/B1aE_xr2p.png)

![image](https://hackmd.io/_uploads/rytBOeS26.png)

...

![image](https://hackmd.io/_uploads/r1SDuxrhp.png)

> Để có được những rename như vậy thì đơn giản ở đâu `switch` thì chắc chắn nó chính là `op` và `op` đương nhiên sẽ lấy của `bytecode[pc]`. Có một việc mình đã nhầm lẫn mặc dù làm cả 2 bài khi viết một disassembler đó là đã quá chú trọng vào việc giá trị của các `reg` thay đổi như thế nào. Nói một cách dễ hiểu đó là khi viết một disassembler ta chỉ cần chú ý đến việc thay đổi giá trị của `pc` giống như sau lệnh `add, sub,...` thì `pc` sẽ nhảy lên tiếp tục là bao nhiêu chứ không cần quan tâm đến giá trị của các thanh ghi sau khi `add, sub,...` là bao nhiêu. Bởi việc viết disassembler là đang viết lại chương trình nên các phép toán như `add, sub, xor,...` ta chỉ cần cho nó là một chuỗi để hiển thị còn chủ yếu **tập trung vào cách `pc` thay đổi như thế nào qua từng câu lệnh.**

Ở đây, kết hợp với debug ta sẽ thấy được đoạn như sau:

![image](https://hackmd.io/_uploads/H1RWperna.png)

Đối với bài này, chương trình đã set up trước một buffer `s[12306]`, buffer này chứa `reg, input, stack`. 

Khi debug lên thì IDA sẽ hiển thị như sau:

![image](https://hackmd.io/_uploads/rkRY6xB26.png)

Vậy câu hỏi được đặt ra bây giờ **làm sao biết được `v100[v32] += v100[cur_pc];` chính là đang `add` hai `reg` ???**

Để trả lời được câu hỏi này, chúng ta sẽ kết hợp thêm với debug ta sẽ có được case 2 như sau:

![image](https://hackmd.io/_uploads/HJj9H0hn6.png)

Đầu tiên ta sẽ thấy được toán tử `+` đang tương tác với var`v100` và `v100` này chính là một mảng có 9 phần tử, lúc này `v32 = 1` và `cur_pc = 2` nên sẽ là `0x70+2=0x72` và được lưu lại `v100[32]`. Vậy qua đó ta có thể kết luận rằng đây chính là 2 reg đang tương tác với nhau, và tổng cộng toàn bộ chương trình có tất cả 9 registers.

Ta lại tiếp tục nhìn đến đoạn code này:

![image](https://hackmd.io/_uploads/HJbbvRhna.png)

Ở đây đang có `HIWORD(v101) = pc + 3;` và sau khi toán tử `+` tương tác xong thì sẽ có bước set up lại `pc` như sau:

![image](https://hackmd.io/_uploads/ryNEwR2nT.png)

Vậy sau khi add thì thanh `pc += 3`. Tương tự cho case 0,1,2,3 sẽ như sau:

```python
        case 0:
            v9 = code[pc+2]
            v10 = code[curpc]
            notep.append(f"{hex(pc)}:     mov reg{v10}, reg{v9}\n")
            pc += 3
            curpc = pc
        case 1:
            v100 = code[curpc]
            val = (code[pc+3]<<8)|(code[pc+2])
            notep.append(f"{hex(pc)}:     mov reg{v100}, {hex(val)}\n")
            pc += 4
            curpc = pc
        case 2:
            v32 = code[curpc]
            v31 = code[pc+2]
            notep.append(f"{hex(pc)}:     add reg{v32}, reg{v31}\n")
            pc += 3
            curpc = pc
        case 3:
            v15 = code[curpc]
            v4 = code[pc+2]
            notep.append(f"{hex(pc)}:     sub reg{v15}, reg{v4}\n")
            pc += 3
            curpc = pc
```

Như đã nói ở trên thì ta sẽ xem những đoạn code như `v100[v32] += v100[cur_pc]` thì `v100[x]` chính là reg và nó cũng tương tự như đoạn code này:

![image](https://hackmd.io/_uploads/SJxACCCh26.png)

Thì `*(_WORD *)&s[2 * x + 0x3000]` cũng chính là reg. Vậy có thể kết luận rằng buffer ban đầu sẽ chứa register có base là 0x3000.

Tiếp đến ta sẽ có case 4 như sau:

![image](https://hackmd.io/_uploads/Bko2_R2n6.png)

> Có một điều khá thú vị đó là nếu như đặt breakpoint ở asm thì mã giả sẽ bị đổi như trên hình. Còn nếu đặt breakpoint ở mã giả thì mã giả sẽ giữ nguyên như lúc ban đầu. 

Đối với case 4 này thì việc nhìn `v100[v37] = s.m128i_u8[cur_pc];` có thể sẽ khó hình dung được nó đang chính xác là lệnh gì. Thay vào đó ta sẽ nhìn với mã giả khi còn được giữ nguyên như sau:

![image](https://hackmd.io/_uploads/BkFobypnp.png)

Ở đây, sẽ thấy được khá rõ khi ban đầu đã xác định được `*(_WORD *)&s[2 * x + 0x3000]` chính là thanh ghi. Vậy thì khả năng cao đây là một lệnh move. Nhưng lệnh move sẽ bao gồm như `mov reg, reg`; `mov reg, val`; `move reg, mem`; `mov mem, reg`. Vậy thì tương ứng nhất đối với ở đây thì chỉ có `mov reg, mem`. Bởi vì không có dấu hiệu nào là reg và phía bên trái toán tử `=` là một reg nên chỉ còn `mov reg, mem`.

Tương tự cho case 4,5,6,7,8,9,10,11,12,13,14

```python
        case 4:
            v37 = code[curpc]
            v36 = code[pc+2]
            notep.append(f"{hex(pc)}:     mov reg{v37}, BYTE mem[val reg{v36}]\n")
            pc += 3
            curpc = pc
        case 5:
            v42 = code[curpc]
            v41 = code[pc+2]
            notep.append(f"{hex(pc)}:     mov reg{v42}, WORD mem[val reg{v41}]\n")
            pc += 3
            curpc = pc
        case 6:
            v33 = code[pc+2]
            v10 = code[curpc]
            notep.append(f"{hex(pc)}:     mul reg{v10}, reg{v33}\n")
            pc +=3
            curpc = pc
        case 7:
            v51 = code[curpc]
            v50 = code[pc+2]
            notep.append(f"{hex(pc)}:     div reg{v51}, reg{v50}\n")
            pc += 3
            curpc = pc
        case 8:
            v21 = code[curpc]
            v20 = code[pc+2]
            notep.append(f"{hex(pc)}:     mod reg{v21}, reg{v20}\n")
            pc += 3
            curpc = pc
        case 9:
            v49 = code[curpc]
            v48 = code[pc+2]
            notep.append(f"{hex(pc)}:     xor reg{v49}, reg{v48}\n")
            pc += 3
            curpc = pc
        case 10:
            tmp = code[curpc]
            notep.append(f"{hex(pc)}:     not reg{tmp}\n")
            pc += 2
            curpc = pc
        case 11:
            v16 = code[pc+2]
            v17 = code[curpc]
            notep.append(f"{hex(pc)}:     and reg{v16}, reg{v17}\n")
            pc += 3
            curpc = pc
        case 12:
            v39 = code[pc+2]
            v40 = code[curpc]
            notep.append(f"{hex(pc)}:     or reg{v39}, reg{v40}\n")
            pc += 3
            curpc = pc
        case 13:
            v12 = code[pc+2]
            v13 = code[curpc]
            notep.append(f"{hex(pc)}:     shl reg{v13}, reg{v12}\n")
            pc += 3
            curpc = pc
        case 14:
            v29 = code[pc+2]
            v30 = code[curpc]
            notep.append(f"{hex(pc)}:     shr reg{v30}, reg{v29}\n")
            pc += 3
            curpc = pc
```
Tiếp tục đến case 15:

![image](https://hackmd.io/_uploads/Hy82A1636.png)

Như đã nói ở phần trên về move cũng như reg thì đây cũng là một loại move nhưng lại khác biệt một tí đó là bên trái toán từ `=` đó là `*(_WORD *)&s[2 * v11 + 0x1000]`. Nếu như đây là một reg thì cũng không chính xác bởi 9 reg xếp liền nhau và có base 0x3000. Vậy chỉ còn duy nhất là `mov mem, reg`. Nhưng mem lúc này khác hoàn toàn với mem ở trên, vậy thật sự thì move này là lệnh gì ?? 

Này thực chất cũng là một lệnh move vào mem nhưng đang move lên đầu của mem tức là push. Vậy ta sẽ nhìn lại một lần nữa buffer của bài sẽ như sau: `phần đầu tiên sẽ chứa input, base 0x1000 sẽ là đầu của stack, base 0x3000 sẽ là nơi đầu tiên chứa 9 thanh ghi`.

Tương tự case 16 sẽ là pop:

```python
        case 15:
            tmp = code[curpc]
            notep.append(f"{hex(pc)}:     push reg{tmp}\n")
            pc += 2
            curpc = pc
        case 16:
            tmp = code[curpc]
            notep.append(f"{hex(pc)}:     pop reg{tmp}\n")
            pc += 2
            curpc = pc
```
Tiếp đến case 17:

![image](https://hackmd.io/_uploads/Skp2UxT36.png)

Ở đây, thấy được rằng chương trình đang lấy giá trị của 2 reg để compare thì trường hợp này chính là cmp.

```python
        case 17:
            v43 = code[pc+2]
            v45 = code[curpc]
            notep.append(f"{hex(pc)}:     cmp reg{v43}, reg{v45}\n")
            # if (v45 == v43):
            #     v102 = 1
            #     v47 = 3
            # if v45 > v43:
            #     v102 = v47
            pc += 3
            curpc = pc
```
> Tại đây có đoạn code mình đã comment lại bởi vì như đã nói ban đầu, không cần quá tập trung vào những giá trị khác, ta chỉ cần tập trung nhận biết được xem nó là lệnh gì và sau lệnh đó `pc` sẽ nhảy bao nhiêu.

Tiếp theo case 18:

![image](https://hackmd.io/_uploads/ry142gTn6.png)

Thì đây chính là biểu diễn cho một lệnh jump, bởi lẽ chỉ có thanh ghi `pc` tham gia tính toán ở đây. Nhưng vấn đề khá lớn đó là hầu như trên đoạn code trên không cho thấy được thanh ghi `pc` tiếp theo sẽ là bao nhiêu. Bao nhiêu ở đây không phải là nhảy đến địa chỉ nào bởi lẽ đã tính toán được địa chỉ nhảy tới mà hãy nhớ mình đang thực hiện viết một disassembler chứ không phải viết lại flow chương trình nên không rõ `pc` sẽ cộng bao nhiêu. 

Tạm thời sẽ nhìn xuống tiếp case 19:

![image](https://hackmd.io/_uploads/HyYNebT2a.png)

Tương tự như thế đây cũng là một lệnh nhảy nhưng có sự so sánh giá trị trước khi nhảy ở đây: `if ( (v93 & 1) != 0 )`, `v93` này chính là kết quả so sánh của lệnh `cmp` ở case 17, và chú ý đến `else` ở đây nếu so sánh thỏa mãn thì sẽ tính toán `pc` để nhảy tới còn nếu không thì sẽ `pc+3`

Tương tự cho case 20,221, 22:

![image](https://hackmd.io/_uploads/SJF7bW6h6.png)

Vậy ta sẽ có được đoạn code cho những lệnh nhảy như sau:

```python
        case 18:
            tmp = pc
            pc += 2
            curpc = (code[curpc])|((code[pc])<<8)
            notep.append(f"{hex(pc)}:     jmp {hex(curpc)}\n")
            pc = tmp+3
            curpc = pc
        case 19:
            v35 = code[pc+2]
            v38 = code[curpc]
            adr = (v35<<8)|v38
            notep.append(f"{hex(pc)}:     je {hex(adr)}\n")
            # if (v102 & 1) != 0:
            #     curpc = adr
            # else:
            #     curpc = pc+3
            # curpc = adr
            curpc = pc +3
        case 20:
            v35 = code[pc+2]
            v38 = code[curpc]
            adr = (v35<<8)|v38
            notep.append(f"{hex(pc)}:     jne {hex(adr)}\n")
            # if (v102 & 1) == 0:
            #     curpc = adr
            # else:
            #     curpc = pc+3
            # curpc = adr
            curpc = pc+3
        case 21:
            v35 = code[pc+2]
            v38 = code[curpc]
            adr = (v35<<8)|v38
            notep.append(f"{hex(pc)}:     jl {hex(adr)}\n")
            # if (v102 & 2) != 0:
            #     curpc = adr
            # else:
            #     curpc = pc+3
            # curpc = adr
            curpc = pc +3
        case 22:
            v35 = code[pc+2]
            v38 = code[curpc]
            adr = (v35<<8)|v38
            notep.append(f"{hex(pc)}:     jge {hex(adr)}\n")
            # if (v102 & 2) == 0:
            #     curpc = adr
            # else:
            #     curpc = pc+3
            # curpc = adr
            curpc = pc +3
```
> Cũng tương tự như trên đã lưu ý, ban đầu mình đã hơi sus khi quá quan tâm đến những giá trị khác, lúc sau phải comment lại hết và chỉ quan tâm `pc`.

Tương tự cho case 23, 24, 25:

```python
        case 23:
            notep.append(f"{hex(pc)}:     final\n")
        case 24:
            v19 = code[curpc]
            v18 = code[pc+2]
            notep.append(f"{hex(pc)}:     mov mem[val reg{v19}], reg{v18}\n")
            pc+=3
            curpc = pc
        case 25:
            v1 = code[curpc]
            v2 = code[pc+2]
            notep.append(f"{hex(pc)}:    mov WORD [val reg{v1}], reg{v2}\n")
            pc += 3
            curpc = pc
```

Cuối cùng là case 26, 27:

![image](https://hackmd.io/_uploads/HyLmE-T3a.png)

![image](https://hackmd.io/_uploads/rJVVN-a26.png)

Ta thấy có `push` và `pop` thì chắc chắn sẽ có `call` và `ret`. Thật sự thì mình cũng không biết vì sao 26 là call nhưng trước tiên nhìn vào 27, ở đây `pc` đang lấy thanh ghi ở đầu stack thì điều này tương ứng với `ret` bởi khi thoát một hàm thì sẽ lấy địa chỉ đầu stack - địa chỉ đã được push khi call để làm địa chỉ trả về nên 27 là `ret` và còn lại 26 chính là`call`

Toàn bộ disassembler của bài:

```python
code = b'\x01\x00\x01\x00\x01\x01p\x00\x01\x02\x02\x00\x19\x01\x00\x02\x01\x02\x19\x01\x00\x02\x01\x02\x01\x00\x02\x00\x19\x01\x00\x02\x01\x02\x19\x01\x00\x01\x00\xff\xff\x02\x01\x02\x19\x01\x00\x02\x01\x02\x19\x01\x00\x01\x00\xfe\xff\x02\x01\x02\x19\x01\x00\x02\x01\x02\x19\x01\x00\x01\x00\x02\x00\x01\x01\x80\x00\x19\x01\x00\x01\x00\xfe\xff\x02\x01\x02\x19\x01\x00\x01\x00\x01\x00\x02\x01\x02\x19\x01\x00\x01\x00\xff\xff\x02\x01\x02\x19\x01\x00\x01\x00\x02\x00\x02\x01\x02\x19\x01\x00\x01\x00\xfe\xff\x02\x01\x02\x19\x01\x00\x01\x00\x01\x00\x02\x01\x02\x19\x01\x00\x01\x00\xff\xff\x02\x01\x02\x19\x01\x00\x01\x07\x02\x00\x01\x08\x06\x00\x1ah\x01\x01\x03\x00\x00\x01\x01\x00\x00\x02\x01\x03\x04\x01\x01\x00\x02\x01\x01\x04\x04\x00\x0e\x02\x04\x01\x04\x0f\x00\x0b\x01\x04\x1a\xf1\x00\x01\x04\x00\x00\x11\x00\x04\x13\xed\x00\x00\x07\x01\x00\x08\x02\x1a\x8c\x01\x01\x04\x01\x00\x11\x00\x04\x13\xed\x00\x1ah\x01\x01\x01\x01\x00\x02\x03\x01\x12\xa5\x00\x1a\xac\x01\x17\x0f\x03\x0f\x04\x0f\x05\x0f\x06\x01\x03\x07\x00\x01\x04\xff\xff\x11\x01\x04\x16T\x01\x11\x01\x03\x15T\x01\x11\x02\x04\x16T\x01\x11\x02\x03\x15T\x01\x01\x04\x00\x00\x01\x03p\x00\x02\x03\x04\x05\x03\x03\x02\x03\x07\x11\x03\x01\x14C\x01\x01\x03\x80\x00\x02\x03\x04\x05\x03\x03\x02\x03\x08\x11\x03\x02\x13[\x01\x01\x03\x02\x00\x02\x04\x03\x01\x03\x0e\x00\x11\x04\x03\x16\x1d\x01\x01\x00\x00\x00\x12_\x01\x01\x00\x01\x00\x10\x06\x10\x05\x10\x04\x10\x03\x1b\x0f\x01\x0f\x02\x0f\x03\x01\x01\xa0\x00\x02\x01\x08\x04\x03\x01\x01\x02\x01\x00\r\x02\x07\x0c\x03\x02\x18\x01\x03\x10\x03\x10\x02\x10\x01\x1b\x0f\x01\x0f\x02\x01\x01\xa0\x00\x02\x01\x08\x04\x01\x01\x0e\x01\x07\x01\x02\x01\x00\x0b\x01\x02\x00\x00\x01\x10\x02\x10\x01\x1b\x0f\x01\x0f\x02\x0f\x03\x0f\x04\x01\x00\x00\x00\x01\x01\xa0\x00\x01\x02\xa8\x00\x01\x03\x01\x00\x04\x04\x01\x02\x00\x04\x02\x01\x03\x11\x01\x02\x14\xc4\x01\x01\x01\xf8\x07\x00\x02\x00\x01\x00\x00\x00\x11\x01\x02\x14\xfc\x01\x01\x01\x07\x00\x11\x07\x01\x14\xfc\x01\x01\x01\x02\x00\x11\x08\x01\x14\xfc\x01\x01\x00\x01\x00\x10\x04\x10\x03\x10\x02\x10\x01\x1b'

notep = []
pc = 0
curpc = 0
# v47 = 2
# v102 = 0
while pc < len(code):
    pc = curpc
    op = code[curpc]
    curpc += 1
    match op:
        case 0:
            v9 = code[pc+2]
            v10 = code[curpc]
            notep.append(f"{hex(pc)}:     mov reg{v10}, reg{v9}\n")
            pc += 3
            curpc = pc
        case 1:
            v100 = code[curpc]
            val = (code[pc+3]<<8)|(code[pc+2])
            notep.append(f"{hex(pc)}:     mov reg{v100}, {hex(val)}\n")
            pc += 4
            curpc = pc
        case 2:
            v32 = code[curpc]
            v31 = code[pc+2]
            notep.append(f"{hex(pc)}:     add reg{v32}, reg{v31}\n")
            pc += 3
            curpc = pc
        case 3:
            v15 = code[curpc]
            v4 = code[pc+2]
            notep.append(f"{hex(pc)}:     sub reg{v15}, reg{v4}\n")
            pc += 3
            curpc = pc
        case 4:
            v37 = code[curpc]
            v36 = code[pc+2]
            notep.append(f"{hex(pc)}:     mov reg{v37}, BYTE mem[val reg{v36}]\n")
            pc += 3
            curpc = pc
        case 5:
            v42 = code[curpc]
            v41 = code[pc+2]
            notep.append(f"{hex(pc)}:     mov reg{v42}, WORD mem[val reg{v41}]\n")
            pc += 3
            curpc = pc
        case 6:
            v33 = code[pc+2]
            v10 = code[curpc]
            notep.append(f"{hex(pc)}:     mul reg{v10}, reg{v33}\n")
            pc +=3
            curpc = pc
        case 7:
            v51 = code[curpc]
            v50 = code[pc+2]
            notep.append(f"{hex(pc)}:     div reg{v51}, reg{v50}\n")
            pc += 3
            curpc = pc
        case 8:
            v21 = code[curpc]
            v20 = code[pc+2]
            notep.append(f"{hex(pc)}:     mod reg{v21}, reg{v20}\n")
            pc += 3
            curpc = pc
        case 9:
            v49 = code[curpc]
            v48 = code[pc+2]
            notep.append(f"{hex(pc)}:     xor reg{v49}, reg{v48}\n")
            pc += 3
            curpc = pc
        case 10:
            tmp = code[curpc]
            notep.append(f"{hex(pc)}:     not reg{tmp}\n")
            pc += 2
            curpc = pc
        case 11:
            v16 = code[pc+2]
            v17 = code[curpc]
            notep.append(f"{hex(pc)}:     and reg{v16}, reg{v17}\n")
            pc += 3
            curpc = pc
        case 12:
            v39 = code[pc+2]
            v40 = code[curpc]
            notep.append(f"{hex(pc)}:     or reg{v39}, reg{v40}\n")
            pc += 3
            curpc = pc
        case 13:
            v12 = code[pc+2]
            v13 = code[curpc]
            notep.append(f"{hex(pc)}:     shl reg{v13}, reg{v12}\n")
            pc += 3
            curpc = pc
        case 14:
            v29 = code[pc+2]
            v30 = code[curpc]
            notep.append(f"{hex(pc)}:     shr reg{v30}, reg{v29}\n")
            pc += 3
            curpc = pc
        case 15:
            tmp = code[curpc]
            notep.append(f"{hex(pc)}:     push reg{tmp}\n")
            pc += 2
            curpc = pc
        case 16:
            tmp = code[curpc]
            notep.append(f"{hex(pc)}:     pop reg{tmp}\n")
            pc += 2
            curpc = pc
        case 17:
            v43 = code[pc+2]
            v45 = code[curpc]
            notep.append(f"{hex(pc)}:     cmp reg{v43}, reg{v45}\n")
            # if (v45 == v43):
            #     v102 = 1
            #     v47 = 3
            # if v45 > v43:
            #     v102 = v47
            pc += 3
            curpc = pc
        case 18:
            tmp = pc
            pc += 2
            curpc = (code[curpc])|((code[pc])<<8)
            notep.append(f"{hex(pc)}:     jmp {hex(curpc)}\n")
            pc = tmp+3
            curpc = pc
        case 19:
            v35 = code[pc+2]
            v38 = code[curpc]
            adr = (v35<<8)|v38
            notep.append(f"{hex(pc)}:     je {hex(adr)}\n")
            # if (v102 & 1) != 0:
            #     curpc = adr
            # else:
            #     curpc = pc+3
            # curpc = adr
            curpc = pc +3
        case 20:
            v35 = code[pc+2]
            v38 = code[curpc]
            adr = (v35<<8)|v38
            notep.append(f"{hex(pc)}:     jne {hex(adr)}\n")
            # if (v102 & 1) == 0:
            #     curpc = adr
            # else:
            #     curpc = pc+3
            # curpc = adr
            curpc = pc+3
        case 21:
            v35 = code[pc+2]
            v38 = code[curpc]
            adr = (v35<<8)|v38
            notep.append(f"{hex(pc)}:     jl {hex(adr)}\n")
            # if (v102 & 2) != 0:
            #     curpc = adr
            # else:
            #     curpc = pc+3
            # curpc = adr
            curpc = pc +3
        case 22:
            v35 = code[pc+2]
            v38 = code[curpc]
            adr = (v35<<8)|v38
            notep.append(f"{hex(pc)}:     jge {hex(adr)}\n")
            # if (v102 & 2) == 0:
            #     curpc = adr
            # else:
            #     curpc = pc+3
            # curpc = adr
            curpc = pc +3
        case 23:
            notep.append(f"{hex(pc)}:     final\n")
        case 24:
            v19 = code[curpc]
            v18 = code[pc+2]
            notep.append(f"{hex(pc)}:     mov mem[val reg{v19}], reg{v18}\n")
            pc+=3
            curpc = pc
        case 25:
            v1 = code[curpc]
            v2 = code[pc+2]
            notep.append(f"{hex(pc)}:    mov WORD [val reg{v1}], reg{v2}\n")
            pc += 3
            curpc = pc
        case 26:
            v1 = code[curpc]
            v2 = code[pc+2]
            add = v1|(v2<<8)
            # v102 = curpc
            notep.append(f"{hex(pc)}:     call {hex(v1|(v2<<8))}\n")
            # curpc = add
            pc+=3
            curpc = pc
        case 27:
            notep.append(f"{hex(pc)}:     ret\n")
            # curpc = v102+2
            # notep.append(f"{hex(curpc)}\n")
            pc += 1
            curpc = pc


for i in notep:
    print(i)
```

