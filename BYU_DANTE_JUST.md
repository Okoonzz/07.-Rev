# BYU_DANTE_JUST

Đây là 3 giải khác nhau, nhưng về độ khó cũng như kiến thức mới khá hay nên mình đã gộp chúng lại với nhau để phân tích cho có mạch :>>

Tải file [tại đây.](https://github.com/Okoonzz/07.-Rev/tree/File/BYUCTF)

## MANGO(JUST)

Đối với bài này tuy thuật toán của nó rất dễ nhưng cách nhận ra nó và biết nó mới là một vấn đề :))).

![](https://hackmd.io/_uploads/Sy__5KTP2.png)

Ở đoạn ASM này có thể nhìn qua thấy được rằng sẽ in ra terminal dòng chữ `Type plain text that will be converted `, tiếp đến nhập "flag" vào. Chú ýe kĩ sau đoạn `fmt_Fscanln` có những mov như `mov     rax, [rdx]` và `mov     rbx, [rdx+8]` chính là dữ liệu nhập vào là độ dài của chuỗi được nhập vào.

Tiếp đến là hàm chuyển đổi từng ký tự sang OCT (này là do mình đã chuyển đổi lại chứ trong mặc định thì phải debug mới thấy rõ ràng được). Kiểm tra hàm này chuyển đổi như thế nào. Khi vào hàm này và debug thứ đạp vào mắt mình đầu tiên đó chính là đoạn code này.

![](https://hackmd.io/_uploads/HyAy15pvn.png)

Ở đây có một đoạn lấy từng byte của chuỗi nhập vào lưu vào edx sau đó `xor rdx, 37h` tức là lấy từng byte của nó xor với 0x37 sau đó có một đoạn chuyển đổi định dạng để in ra. Trong Golang thì đoạn định dạng chuỗi này có thể được thấy như sau:

```go
fmt.Sprintf("%03o", int(plain[i])^0x37)
```
> Có nghĩa là chuỗi sau khi được xor với 0x37 thì in ra với độ dài 3 chữ số nếu không đủ thì sẽ thêm những số 0 phía trước và "o" ở đây tức là dạng bát phân.

Tiếp đến là hàm `main_gen_table` thì hàm này như sau:

![](https://hackmd.io/_uploads/SJWf75aw3.png)

![](https://hackmd.io/_uploads/B1u7X9Tv2.png)

![](https://hackmd.io/_uploads/BklmLmcpP2.png)

Điều đầu tiên nhìn vào hàm này nhận vào 1 đối số có tên là `__int64 str_len` nhưng trước khi gọi hàm có một đoạn `mov     rax, rbx` nên có thể hiểu rằng lấy len của chuỗi vừa chuyển đổi ở hàm chuyển đổi OCT.

Bên trong hàm này có một đoạn `crypto_rand_Prime` và trước đó là `mov     ecx, 14h` thì đoạn này chính là random ra một số nguyên tố bất kì có độ dài là 20bit. Tiếp đến là `Cannot get prime` có thể không quan tâm mấy nội dung ở đây. Và hai nhánh này đều đi xuống một đoạn code, ở đoạn này điều đang chú ý nhất đó là `math_rand__ptr_Rand_Seed`. Ở đây, nhận thấy được một seed. Đoán được rằng seed này lấy từ số nguyên tố vừa tạo ra bởi seed thì cần một số để làm seed nhưng trước đó không hề thấy bất cứ chỗ nào ngoại trừ đoạn lấy số nguyên tố, nên có thể đoán được rằng lấy SNT vừa tạo ra để làm seed. (Có thể nói đoạn này tương tự bài ["GO"](https://github.com/Okoonzz/07.-Rev/blob/main/BYUCTF.md) ở BYU)

Ở dưới nhận thấy rằng có một vòng lặp và lặp đến khi nào bằng với độ dài của chuỗi vừa chuyển đổi thì dừng lại. Và trong lặp này có đoạn code sau

![](https://hackmd.io/_uploads/HJiJA5awn.png)

Ở đây thấy được rằng hàm random này giống như `rand.Int(str_len)`

> Qua đó có thể hiểu được hàm này đó là tạo một mảng unit32 có độ dài bằng độ dài của chuỗi vừa chuyển đổi, sau đó random một số nguyên tố (20 bit) để làm seed. Tiếp đến là random trong khoảng từ 0 đến str_len để làm vị trí di chuyển.

Tiếp đến là một hàm `main_shuffle` nhận vào hai đối số đó là `string plain` và `_slice_uint32 translate` tức là một chuỗi và và chuyển đổi. 

![](https://hackmd.io/_uploads/BkcSiqTwh.png)

![](https://hackmd.io/_uploads/B1NLjqawh.png)

![](https://hackmd.io/_uploads/SknIoqTPh.png)

![](https://hackmd.io/_uploads/Hk4Doc6vn.png)

Hàm này nếu chưa đọc quen ASM thì vô cùng khó nhìn và không hình dung được thực sự nó đang làm gì. Đây là đoạn debug cụ thể

![](https://hackmd.io/_uploads/rJNMMopwh.png)

Đây là đoạn translate trả về từ `gen_table`

![](https://hackmd.io/_uploads/Hkqvzo6wh.png)

Đây là đoạn chuỗi đã được conv sang OCT với dữ liệu nhập vào là `12345`

Đầu tiên khi vào hàm shuffle này bắt gặp được đoạn so sánh với 0x80

![](https://hackmd.io/_uploads/H1ETzi6Dh.png)

Mình đoán rằng nó xem xem chuỗi này có từ nào không phải là printable hay không.

![](https://hackmd.io/_uploads/Hy6xQs6Pn.png)

Tiếp đến nó sẽ lấy chỉ số idx trong chuỗi vừa chuyển đổi, chạy từ đầu đến cuối chuỗi 

![](https://hackmd.io/_uploads/By44Xjawn.png)

Sau đó đoạn này, sẽ lấy từ đầu đến cuối mảng vừa sinh ra được ở hàm `gen_table`

![](https://hackmd.io/_uploads/BkKaQs6v3.png)

Đoạn này, được hiểu rằng nó sẽ lấy ở chuỗi ban đầu tới giá trị thứ idx của mảng vừa tìm được.
> Ban đầu bao gồm 0 0 6 0 0 5 0 0 4 0 0 3 0 0 2 thì ở đây vì lúc đầu thấy được rằng phần tử đầu tiên của mảng sinh ra từ `gen_table` là 8 nên ở đây chuyển số `4` vào `r9d`. Chạy thêm vài lần sẽ thấy.

Tiếp đến kiểm tra đoạn return 

![](https://hackmd.io/_uploads/H1MFIjTvn.png)

Dễ dàng thấy hai số đã được chuyển đổi vào lần lượt 4 và 0. Vậy qua đó có thể hiểu được hàm shuffle này như sau:

```go
for i := range plain {
	out = append(out, plain[translate[i]])
}
```
> Đoạn code trên ta có thể phân tích tĩnh như sau. Nhận thấy được rằng `movzx   r10d, byte ptr [r8+rbx]` thì `r8` ở đây như mảng sau khi chuyển OCT còn `rbx` như idx sau đó di chuyển từng byte để kiểm tra xem có phải chuỗi hợp lệ hay không. Còn ` mov     r11d, [rdi+rbx*4]` thì `rdi` đóng vai trò như mảng của hàm `gen_table` và `rbx*4` tức lấy 4 byte hoặc coi nó như idx cũng không sai sau đó di chuyển vào `r11d`. Sau cùng `movzx   r9d, byte ptr [r8+r11]` lấy `r8+r11` tức là thay đổi như trên đoạn code.

Cuối cùng là hàm `main_mangoify`. Thì hàm này mình không bận tâm lắm khi nhìn qua là có thể dễ thấy nó chuyển đổi chuỗi vừa encrypt thành các ký tự nhất định.

Sau đây là đoạn script để giải bài này:

```go
package main

import (
	"fmt"
	"math"
	"math/rand"
	"os"
	"strconv"
	"strings"
)

func decode(s []byte, chunkSize int) string {
	decoded := []byte{}
	for num := 0; num < len(s); num += 3 {
		chunk := string(s[num:(num + 3)])
		letter, _ := strconv.ParseInt(chunk, 8, 32)
		decoded = append(decoded, byte(letter^0x37))
	}

	return string(decoded)
}

func rev_shuffle(enc string, translate []uint32) []byte {
	// rev_mapping := map[uint32]uint32{}
	// out := make([]byte, 0, len(enc))
	// for i, v := range translate {
	// 	rev_mapping[v] = uint32(i)
	// }

	// for i := range translate {
	// 	out = append(out, enc[rev_mapping[uint32(i)]])
	// }
	// return out

	tmp := make([]uint32, len(enc))
	for idx := range translate {
		tmp[idx] = translate[idx]
	}

	// fmt.Println(tmp) // In ra mảng tạm

	// Giải mã dựa trên thứ tự xuất hiện đã lưu
	tmpDec := make([]byte, len(enc))
	for idx := range translate {
		tmpDec[tmp[idx]] = enc[idx]
	}

	return tmpDec
}

func gen_table(str_len int, candidate uint32) []uint32 {

	idx := make([]uint32, 0, str_len)
	uniq := map[uint32]bool{}

	rand.Seed(int64(candidate))

	for {
		if len(uniq) == str_len {
			break
		}

		r := uint32(rand.Intn(str_len))

		if exists, ok := uniq[r]; !ok || !exists {
			idx = append(idx, r)
		}

		uniq[r] = true
	}

	return idx
}

func get_primes(start, end uint32) []uint32 {
	primes := []uint32{}
	for num := start; num < end; num++ {
		isPrime := true
		for i := uint32(2); i <= uint32(math.Sqrt(float64(num))); i++ {
			if num%i == 0 {
				isPrime = false
				break
			}
		}
		if isPrime {
			primes = append(primes, num)
		}
	}
	return primes
}

func rev_mangoify(encoded string) string {
	encoded = strings.ReplaceAll(encoded, "O", "1")
	parts := strings.Split(encoded, "o")
	decoded := []byte{}

	for _, p := range parts {
		c, _ := strconv.ParseInt(p, 2, 8)
		decoded = append(decoded, byte(c))
	}

	return string(decoded)
}

func solve(encoded string, start, stop uint32) {
	fmt.Println("Generating primes...")
	primes := get_primes(start, stop)

	fmt.Println("Solving...")
	for _, prime := range primes {
		translate := gen_table(len(encoded), prime)
		rev := rev_shuffle(encoded, translate)
		// fmt.Print((rev))
		decoded := decode(rev, 3)
		if strings.HasPrefix(decoded, "justCTF") {
			fmt.Printf("Seed was %d\n", prime)
			fmt.Println(decoded)
		}
	}
}

func main() {
	data, err := os.ReadFile("output.txt")
	if err != nil {
		fmt.Print(err)
		return
	}

	range_start, range_end := (1 << 19), (1 << 20)
	encoded := rev_mangoify(strings.TrimSpace(string(data)))
	solve(encoded, uint32(range_start), uint32(range_end))
}
```
> Đoạn script trên có hai cách để giải mã đoạn mã hóa. Một là cách mình đã comment lại và cách 2 là cách do chính mình viết ra. Cụ thể lưu lại thứ tự xuất hiện rồi tiến hành giải mã nó.

## RUSTSAFETY (DANTE)

Giải này chỉ có 1 bài rev và viết bằng rust. Về thuật toán cũng không có gì phức tạp, tại rust và golang là một thứ gì đó rất khoai trong rev =))))

![](https://hackmd.io/_uploads/rJ2YofRPh.png)

Đây là hàm main, vào xem hàm `sub_56357ABCB2D0` là cái gì thôi nào.

![](https://hackmd.io/_uploads/B1ToxmAwh.png)

Ở đây có một điều đặc biệt đó chính là bắt gặp chuỗi `MY_FAV_POET`. **Chuỗi này tức là xác định biến môi trường**. Lúc này mình đã thử `export MY_FAV_POET=everything` thì nhập vào code sẽ hiển thị `You have bad taste`. Nên mình đã đoán nó sẽ là `DANTE`.

![](https://hackmd.io/_uploads/SkcMGgkdn.png)

Ở đây sau khi pass được việc đầu tiên, thấy được rằng các nhánh đều xuống `loc_55836DCAD433` nên mình sẽ chỉ tập trung vào chỗ này.

`sub_561D50DAFE30` đối với hàm này đơn giản thấy được rằng nó sẽ bỏ đi ký tự đầu tiên của chuỗi nhập vào để lấy đúng len của nó. Ví dụ nhập `12345` thì độ dài của nó sẽ là 6 (lấy luôn cả ký tự "enter") khi qua hàm này sẽ bỏ đi 1 đưa về độ dài chuỗi ban đầu là 5.

Tiếp tục đến `cs:off_561D50DFAB28` đây là hàm chính của chương trình. 

![](https://hackmd.io/_uploads/SJMY9zxdn.png)

Quan sát ở đây có thêm một hàm nhận vào 3 đối số 

![](https://hackmd.io/_uploads/ry_jcGx_h.png)

Vào trong hàm này thấy rất nhiều biểu thức cũng như `goto` và `return`. Tuy chỉ nhiều và làm rối nhưng thực chất ở đây ta chỉ cần chú ý đoạn code sau:

![](https://hackmd.io/_uploads/ByteoMed3.png)

![](https://hackmd.io/_uploads/SkPWifld3.png)

![](https://hackmd.io/_uploads/BJbwizl_h.png)

Cụ thể các đoạn này kiểm tra đầu vào của code có phải khác các ký tự `+`, `-` hay không và kiểm tra len của code nhập vào phải nhỏ hơn 8 và từ 0 đến 9. Và mục đích chính của đoạn code này đó là chuyển từng ký tự của chuỗi nhập vào về thành số nguyên. Và kết quả trả về có 2 dạng. Nếu đúng theo điều kiện thì sẽ là kết quả của `v6 = input << 32` nếu sai thì sẽ trả về `256 << 1`.

> Có những đoạn else chắc chắn không bao giờ nhảy tới được bởi điều kiện ràng buộc là `a3` nên ta có thể dễ dàng bỏ qua được những đoạn này.

Sau khi kết thúc đoạn này ta sẽ chú ý đến đoạn tiếp theo

![](https://hackmd.io/_uploads/r1WMcQgd3.png)

Đoạn này lấy kết quả trả ra từ hàm và so sánh với `2A00000000h`. Từ đó ta sẽ có được đoạn script để giải quyết như sau:

```python
def solvee():
    
    x = BitVec('x', 64)  

    constraints = And((x << 32) & 0x0FFFFFFFF00000000 == 0x2A00000000)

    solver = Solver()
    solver.add(constraints)

    if solver.check() == sat:
        model = solver.model()
        solution = model[x].as_long()
        return solution

    return None

result = solvee()
if result is not None:
    print(f"number: {result}")
else:
    print("sorry :(( .")
    
# number: 42
```
Nếu làm đủ nhiều sẽ nhận thấy được rằng các dạng bài như thế này kiểu gì cũng sẽ liên quan đến bỏ thêm một tệp gì đó vào đường dẫn nào đó. Nên chúng ta sẽ phân tích tiếp tục. 

![](https://hackmd.io/_uploads/Byp-smxd3.png)

Ở đây khi vào `xmmword_55C7377F4000` thấy được một dãy số `6E702E65746E6144333C492F706D742Fh`. Bỏ vào [CyberChef](https://gchq.github.io/CyberChef/) hoặc ta có thể dùng đoạn script như sau để dịch được nó:

```python
string = "676E702E65746E6144333C492F706D742F"
reversed_string = bytes.fromhex(string).decode()[::-1]
print(reversed_string)

#/tmp/I<3Dante.png
```
Như thế là xong hết tất cả.

## SASSIE (BYU)

Dạng bài này cũng tương tự, chắc chắn sẽ có đặt thêm một thứ gì đó ở một path nào đó. Thế nên hãy bắt đầu phân tích xem thử có gì nào.

![](https://hackmd.io/_uploads/HJqb65eOh.png)

Đập vào mắt mình đó là một hàm xor khá đơn giản. Mình thử viết lại hàm này thử xem nó là gì.

```python
offset = [0xe8, 0x7d, 0x61, 0xa8, 0x6e]

arr = [0xc7, 0x0d, 0x13]

for i in range(5):
    offset[i] ^= arr[i%len(arr)]

print("".join(chr(i) for i in offset))
#output: /proc
```

Tiếp tục đến đoạn tiếp theo lại có hàm tương tự và tương tự như script trên ta có được output như sau `/tmp/tmpt2nxegs1` thì đây chắc chắn là path và tên file cần thêm vào để đúng đường dẫn. 

![](https://hackmd.io/_uploads/Hys0foldh.png)

Ở đoạn này, dễ dàng thấy được rằng nó sẽ mở file này và kiểm tra xem nội dung trong file có giống "thứ gì đó được set sẵn" hay không. Nếu không giống thì sẽ trả về "Wrong".

![](https://hackmd.io/_uploads/rkMkSigd2.png)

Đoạn code này rất dễ nhìn, nó vẫn sẽ thực hiện phép xor bình thường và có một dữ liệu đã được set sẵn (Đoạn này các WU trước mình đã nói rất kĩ nên không nói thêm). Đây là script để giải được đoạn này 

```python
offset_3 = [0x1d, 0xe9, 0x89, 0x72, 0x19, 0x06, 0x58, 0x4d, 0x75, 0xee, 0xf9, 0x52, 0xf4, 0x5d, 0xa4, 0x15]

arr_3 = [0x81, 0x36, 0x3d, 0x4b, 0xde]

for i in range(15):
    offset_3[i] ^= arr_3[i%len(arr_3)]

result = ''.join(hex(i)[2:] for i in offset_3)
print(result)

#res -> MD5: lol
```
Sau khi đáp ứng được tất cả, đoạn quan trọng nhất của bài này chính là ở đây

![](https://hackmd.io/_uploads/SJSnBogdh.png)

Ở đây có một đoạn shellcode được đặt ở đây. Tiến hành debug và xem thử nó có gì ở đây.

![](https://hackmd.io/_uploads/BJPkpjeOn.png)

Đến được đây thì đoạn ASM trên khá đơn giản. Nó chỉ load những dữ liệu đã được set sẵn và xor với một list những số cũng đã được set sẵn. Ta có thể chạy tiếp để xem được flag cuối cùng hoặc đây là script tham khảo:

```python
flag = [0x24, 0x41, 0x64, 0xb8, 0xa5, 0xc5, 0x3d, 0x4b, 0x7e, 0xa9, 0xa3, 0xda, 0x19, 0x5e, 0x21, 0xa9, 0x8e, 0xc1, 0x75, 0x09, 0x7f, 0xbc, 0x8e, 0x96, 0x72, 0x0d, 0x24, 0xea, 0xe2, 0xde]

final = [0x46, 0x38, 0x11, 0xdb, 0xd1, 0xa3]
for i in range(30):
    flag[i] ^= final[i%(len(final))]

print("".join(chr(i) for i in flag))
```

=> Các thử thách trên có lẽ thích nhất vẫn là GO nó vô cùng hay. Và cũng như những kiến thức mới về env, shellcode và cách mã giả trả về với ngôn ngữ Rust khá hay, đòi hỏi debug phải tinh tế.








