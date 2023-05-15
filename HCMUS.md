# HCMUS2023
## Crypto + RE

Tải file [tại đây.](https://github.com/Okoonzz/07.-Rev/blob/File/main.zip)

Khi lần đầu tiếp xúc với dạng bài như này, mình đã rất lúng túng trong việc xác định các hàm. Nên đã có các anh hint cho mình nó là gì và thế là đã giải được.

![](https://hackmd.io/_uploads/S1bkVrT4n.png)

Đọc qua code của đề bài, thấy được rằng để thực hiện file này cần phải tạo riêng một file với nội dung là "flag.txt" bởi nếu không thực hiện thì khi chạy bình thường sẽ không trả được bất kì kết quả nào.

Phân tích các dòng code tiếp theo, thì ta sẽ tiến hành thực hiện nhập vào 2 input và sẽ có một hàm đầu tiên kiểm tra các input này. Phân tích xem hàm này kiểm tra cái gì.

![](https://hackmd.io/_uploads/SJtgSHp4n.png)

Ở đây, thấy được rằng 2 input của ta tiếp tục cho qua một hàm "s". Sau đó sẽ có 2 mảng đó là v8[8] và v9[10], tiếp đến là một vòng for để kiểm tra xem hàm "s" đã làm một điều gì đó với input và lưu kết quả tại biến `v6` có bằng với hai mảng `v8` và `v9` cho trước hay không. Nếu bằng thì tiếp tục chương trình, nếu không thì sẽ thoát. Công việc tiếp theo đó là phải xác định được hàm "s" đã làm gì với input.

![](https://hackmd.io/_uploads/HyjM8BTVh.png)

![](https://hackmd.io/_uploads/HkxVISpN3.png)

Ở đây, nếu ai chưa từng tiếp xúc hay chỉ học sơ qua hash thì sẽ không hề nhận biết được hàm này đang làm gì. Bởi lẽ, trong hàm này toàn là dịch bit phải rồi dịch bit trái rất nhiều thứ ở đây. Chắc chắn rằng nếu ngồi dịch ngược hàm này là một điều rất khó. 

Thử cop một số trong các mảng kia dán lên google thì đây là một kết quả.

![](https://hackmd.io/_uploads/r1A6UBaV3.png)

Để hiểu rõ hơn cách hoạt động thì mình xin giới thiệu một trang web về cách một sha256 hoạt động như thế nào: https://sha256algorithm.com/. Ở trang web này mô phỏng rất cụ thể và vô cùng dễ dàng thấy được các phép toán giống hệt như trong source code mà ta đang phân tích. 

Sau khi đã hiểu rõ được cách hoạt động của hash, ta thử kiểm tra từng số của các mảng này bởi lẽ trong các phiên bản sha thì có một vài phiên bản khá giống nhau chỉ khác mỗi các số "Magic". Sau khi kiểm tra từng số thì đây là sha-224.

Sau khi xác định xong việc còn lại là chỉ cần tìm được `v6`, `v7` là gì bằng cách chuyển các số ở `v8`, `v9` về hex và chỉ việc viết lại chúng liền nhau.

Bởi hash có một tính chất đó là chỉ có một chiều và không thể tìm chiều ngược lại. Tức là khi có được dữ liệu cần hash thì qua sha224 hay bất kì sha nào chỉ ra được duy nhất một số và không thể nào từ số đó mà tìm lại được dữ liệu trước khi hash. Vì thế điều cần làm tiếp theo đó là phải [**crack hash**.](https://crackstation.net/). Sau khi crack hash xong ta sẽ có được hai input (recis, cannibalization).

Sau khi có được hai input, tiếp tục phân tích hai hàm "S" và "M" bởi lẽ hai input lần lượt qua hai hàm này. Để phân tích hai hàm này, tương tự như đã phân tích ở hàm "s" thì dễ thấy được hai hàm này lần lượt là SHA-256 và MD5.

Bởi chương trình sau khi tiếp tục cho input qua sha256 và md5 lại tiếp đến một hàm "enc". Tiến hành phân tích hàm này thì dễ dàng nhận ra rằng ở đây mã hóa sử dụng AES-256. Việc cuối cùng là giải mã file đề đã đưa ra bằng cách dung AES-256

```
from Crypto.Cipher import AES
from hashlib import *
from pwn import *

f = open("./flag.txt.enc", 'rb')
prg = f.read()

input1 = 'recis'
input2 = 'cannibalization'

key = bytes.fromhex(sha256(input1.encode()).hexdigest())

iv = bytes.fromhex(md5(input2.encode()).hexdigest())

text = prg

cipher = AES.new(key, AES.MODE_CBC, iv)

plaintext = cipher.decrypt(text)

print (plaintext)
```

=> Tóm lại ở đây giúp ta nhớ được cách hoạt động của sha cũng như quen với các số Magic. Thường thì dịch phải rồi dịch trái rồi xor rất nhiều thì phải nghĩ đến ngay hash và có rất nhiều số lưu dưới dạng mảng như thế thì phải nghĩ đến sha. Việc đọc rồi phân tích kỹ thì vô cùng khó.

## RE

Tải file [tại đây.](https://github.com/Okoonzz/07.-Rev/blob/File/server.zip)

Thực sự thì bài này khá khó với mình, mãi sau khi xong giải và source code được up lên và tham khảo từ rất nhiều anh lớn mình mới có thể giải quyết được bài này. Và wu này viết lại nhằm nói kĩ hơn các cách nhận biết cũng như cách debug còn cơ bản nó hoàn toàn dựa vào cách giải của anh Jinn.

Đầu tiên, đoạn mã từ hàm main của bài này được viết bằng golang - đây là một ngôn ngữ thiên hướng lập trình hướng đối tượng, và golang là một ngôn ngữ rất khó debug nếu không biết cách tiếp cận.

Đây là source code của bài:

```
package main
import (
	"bufio"
	"crypto/hmac"
	"crypto/sha256"
	"crypto/tls"
	"encoding/binary"
	"encoding/hex"
	"fmt"
	"io"
	"os"
	"time"
)
var address = "0.0.0.0:9000"
var key = "Bj7tSK6L4E8tmVebTzH0O0ylb1dTcdpahryyGi2of3q3TLXJxeNYdeUFveFehbOWqrjFQAxV4EF9Rb4c"
var flag = "HCMUS-CTF{1_us3_t1mest4Mp_W1tH_k3y_T0_4uTHEnT1c4t3d_dATA}"
func main() {
	cer, err := getRandomTLS(2048)
	if err != nil {
		fmt.Println("ERROR", err)
		os.Exit(1)
	}
	tlsconfig := &tls.Config{Certificates: []tls.Certificate{cer}}
	ln, err := tls.Listen("tcp", address, tlsconfig)
	if err != nil {
		fmt.Println("ERROR", err)
		os.Exit(1)
	}
	fmt.Println("Send me data and receive the flag")
	fmt.Printf("[#] Start listening on %s", address)
	for {
		conn, err := ln.Accept()
		if err != nil {
			fmt.Println("ERROR", err)
			continue
		}
		Addr := conn.RemoteAddr().String() //Lây địa chỉ IP của client
		fmt.Printf("[%s] Receive Connection\n", Addr) // In ra đã nhận IP này 
		resp := bufio.NewReader(conn) // Đọc dữ liệu từ client, giống như input
		conn.SetReadDeadline(time.Now().Add(time.Second * 1)) // Set deadline receive data
		hashbuf, _ := io.ReadAll(resp) // Đọc toàn bộ dữ liệu từ client 
		fmt.Printf("[%s] Receive hash %s \n", Addr, string(hashbuf))
		unix := time.Now().Add(-7 * time.Hour).Unix() // Tính giờ
		sendFlag := false
		for i := unix - 30; i < unix+30; i++ {
			int64buf := make([]byte, 8)
			binary.LittleEndian.PutUint64(int64buf, uint64(i))
			mac := hmac.New(sha256.New, []byte(key))
			mac.Write(int64buf)
			sha := hex.EncodeToString(mac.Sum(nil))
			//fmt.Println(sha)
			if string(sha) == string(hashbuf) {
				conn.Write([]byte(flag))
				sendFlag = true
				break
			}
		}
		if !sendFlag {
			conn.Write([]byte("No flag for you."))
		}
		conn.Close()
	}
}
```

Cơ bản đoạn code trên với nội dung rằng sau khi kết nối đến server nó sẽ in ra các hash và truyền cho nó dữ liệu sau khi nhận thì sẽ kiểm tra với chuỗi đã được mã hóa bằng sha256 nếu như đúng thì sẽ in ra được flag.

Đó là góc nhìn từ phía người ra đề, bâu giờ nhìn nó theo cách người nhận đề.

Khi chuyển sang code C đọc thực sự là rất khó hình dung và không thể nào hiểu được, vì thế nên chuyển sang góc nhìn bằng ASM.

Khi chạy chương trình như sau và đây là thứ ta nhận được:

![](https://hackmd.io/_uploads/HkQHHygBn.png)

Ở đây, trả về một chuỗi bao gồm địa chỉ cũng như "Receive hash". Tiến hành tìm chuỗi này ở trong code thử xem nằm ngay đoạn nào. Và đây là kết quả:

![](https://hackmd.io/_uploads/SktqSJxH3.png)

Để hiểu rõ hơn về cách golang biểu diễn như thế nào trong quá trình chạy có thể tham khảo [tại đây.](https://medium.com/@nishanmaharjan17/reversing-golang-binaries-part-1-c273b2ca5333)

> Nôm na rằng trước khi gọi gì đó thì sẽ có rất nhiều hàm được gọi để chuẩn bị cũng như các hàm để check lỗi. Và ta thấy được có các lea trước khi call thì đây cũng chỉ là struct không cần bận tâm quá nhiều.

Sau khi đọc xong bài tham khảo về golang, ta sẽ tiến hành đến mục tiêu của bài này *0x00000005E4CBB*

![](https://hackmd.io/_uploads/H1ASQgeSn.png)

Cụ thể ở đoạn này, ta đã biết đối với golang bắt đầu một hàm truyền vào đối số thường là rax rất khác so với C hay C++ nên nó di chuyển vào hàm, tiếp đến mã hóa main_key bằng một lệnh call và tên hàm đó là crypto_hmac_New với main_key được cho sẵn. Sau đó được lưu lại ở rbx (lưu 8 bytes) thông qua `mov rbx, [rsp+360h+int64buf.array]`. Vì đây không stripped nên đọc khá rõ ràng tiến hành tra các từ như "hmac hash python" trên mạng sẽ ra cú pháp cũng như cách sử dụng nó. *(tham khảo thêm [tại đây](https://stackoverflow.com/questions/38133665/python-encoded-message-with-hmac-sha256))*. Kiểm tra main_key và đây chính là nó:

![](https://hackmd.io/_uploads/SJvcSglBn.png)

Sau khi thực hiện băm xong thì tiến hành in ra và so sánh.

![](https://hackmd.io/_uploads/ByReUllS2.png)

Sở dĩ khi phân tích, mình đã bỏ qua nhánh này:

![](https://hackmd.io/_uploads/BJ5NLxeHh.png)

Bởi lẽ nếu ta nhìn thêm xuống dưới thì tất cả các đoạn này đều không dẫn đến một nơi nào để tiếp tục tiến hành phân tích cả nên ta có thể phân tích theo nhánh còn lại. Và một điều làm ta phải đi theo nhánh bên trái là bởi bên nhánh này dẫ đến flag nên ta cứ mạnh dạn đi theo nhánh này mà không cần suy nghĩ.

Công việc phân tích coi như đã xong, bây giờ ta đã có được luồng của chương trình đó là nó sẽ mang main_key đi hash sau đó lưu ở "**int64buf.array**" và sau đó mang đi so sánh với nhập vào nếu đúng thì sinh ra flag. Bây giờ phải xem được giá trị của "int64buf.array" sau mỗi lần hash là gì.

Tiến hành đặt các breakpoint tại đây để xem được kết quả mong muốn:

![](https://hackmd.io/_uploads/ryeqsxeH3.png)

![](https://hackmd.io/_uploads/rkZsjxgBn.png)

> Vì đây là golang không thể nào dùng debug như thường lệ. Bắt buộc phải dùng attach rồi sinh ra thêm tiến trình mới (F9) rồi mới tiến hành debug. Trong quá trình chạy, nếu như gặp lỗi cứ nhấn yes để pass -> F8 -> F9 tuyệt đối không được để thoát chương trình. Lý dó vì sao lại có F8, F9 bởi khi call thì golang sẽ nhảy vào hàm runtime... đây là một đặc tính của loại ngôn ngữ này nên phải làm như thế để pass được.

Trong quá trình debug phát hiện ra rằng format của int64buf có dạng như sau:
```
byte1 + byte2 + b'bd' + 0000
```
**Lưu ý:** b'bd' sẽ khác nhau ở mỗi lần chạy debug cũng như mỗi lần mở lại file. Nên chỉ bắt đúng 1 lần.

Việc còn lại của mình đó là tìm kiếm byte1 và byte2 ở đây là gì. Đây là đoạn scipt dùng bruteforce để tìm kiếm 2 byte đó:

```
import hmac
import hashlib
from pwn import *
sus = """a5c341ee782c538d9c666ef20861d6a4b44d24f73c094c8c00a3c942e39b7763"""
main_key = b'Bj7tSK6L4E8tmVebTzH0O0ylb1dTcdpahryyGi2of3q3TLXJxeNYdeUFveFehbOWqrjFQAxV4EF9Rb4c'
for i in range(255):
    for j in range(255):
        array = bytes([i,j]) +b'bd' + b'\x00'*4
        signature = hmac.new(main_key,msg=array,digestmod=hashlib.sha256).digest()
        if signature.hex() in sus:
            print(i,j,bytes([i,j]))
            break
```
sus ở đây được lấy random các hash nhận được từ server. Cụ thể đoạn code này sẽ tìm tất cả các trường hợp vì cần tìm 2 byte thiếu nên dùng 2 vòng lặp và sẽ có một "signature", nếu như "signature" này có trong sus được sinh ra từ đề bài thì sẽ hiển thị ra. Và điều đặc biệt khi cop tất cả các hash nhận được lúc nào tìm ra được (i, j) thì j luôn là một số không đổi.

Sau khi sinh ra được (i, j) việc cuối cùng gửi dữ liệu này lên và đợi in ra được flag. Ở đây đoạn script ở trên tuy sinh ra được j là 31 nhưng khi chọn để gửi lên server ta sẽ chọn lớn hơn giá trị này để nó có thể bắt được hết tất cả trường hợp trong phạm vi của j. Nếu ta để ý kỹ thì các hash sẽ tăng theo j lần lượt từ 31, 32, 33, 34,... nên ta sẽ bắt trước một số.

```
import hmac
import hashlib
from pwn import *
array = bytes([100,32]) +b'bd' + b'\x00'*4
main_key = b'Bj7tSK6L4E8tmVebTzH0O0ylb1dTcdpahryyGi2of3q3TLXJxeNYdeUFveFehbOWqrjFQAxV4EF9Rb4c'
signature = hmac.new(main_key,msg=array,digestmod=hashlib.sha256).digest()
while True:
    r = remote("0.0.0.0", 9000, ssl=True)
    r.send(signature.hex().encode())
    rec = r.recv(1024)
    if b"{" in rec:
        print(rec)
        break
    r.close()
```
Và đây là kết quả.

![](https://hackmd.io/_uploads/H1q4_WxHn.png)

=>Tóm lại ở bài này ta biết sự quan trọng của debug cũng như cách đọc và cách quan sát nhạy bén các đầu ra của các hàm.
