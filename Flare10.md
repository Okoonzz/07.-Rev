# FLARE-ON 10

## chall3

Đầu tiên mình sẽ nhận được 1 file có mã giả như thế này

![image](https://hackmd.io/_uploads/SJAa0npET.png)

Ở bài này, chương trình sẽ tiến hành kiểm tra argv để xem có hợp lệ hay không, và argv được chia làm nhiều phần cách nhau bởi `/`. Bây giờ sẽ đi vào phân tích từng phần. Và ta sẽ quy ước argv là `c` cho dễ phân tích.

### Part1

Như ở trên ta sẽ dễ dàng nhận thấy được `c[2] + 4*c[1] == 295`, nếu không bằng thì sẽ thoát chương trình. Tiếp đến `c[5] == R`. Sau đó xuống ở dưới đoạn cấp phát vùng nhớ sẽ có được như sau:

![image](https://hackmd.io/_uploads/B1DA1aaNT.png)

`v11` ở đây chính là `c[6]` và cũng là quyền để cấp phát vùng nhớ. Tham khảo thêm [ở đây.](https://learn.microsoft.com/en-us/windows/win32/Memory/memory-protection-constants) và đó cũng chính là `@`.

=>Qua đó payload của chúng ta có cho đến hiện tại bao gồm:
`09C52R@aaaaaaa/`.

Tiếp đến nó sẽ load shellcode vào và gọi shellcode đó.

![image](https://hackmd.io/_uploads/H1iVW66Np.png)

Đây chính là đoạn shellcode:

![image](https://hackmd.io/_uploads/rkXGGTaVp.png)

Nhìn qua thì có vẻ khá bình thường với phép xor ngay tại đây:

![image](https://hackmd.io/_uploads/SkRHzTTEp.png)

Nhưng hãy nhìn kỹ hơn một xíu, ta sẽ thấy một điều khá lạ ở ngay đoạn này:

![image](https://hackmd.io/_uploads/HyUdM6a4T.png)

Tất cả dữ liệu mang đi tính toán xor đều được lấy từ `rbp` chỉ có duy nhất `0x56` thì được load vào tại `rcx`. Nhìn sang phần hex ta sẽ thấy dữ liệu được load lên stack theo như sau: `C6 45 F0 16,  C6 45 F1 17, ..., C6 41 F4 56`. Tất cả đều xuất phát từ `C6 45 ...` nhưng chỉ có duy nhất ở dữ liệu cuối cùng là `41`. Câu hỏi bây giờ đặt ra là `41` này ở đâu mà ra ?

Thì đó chính là ở argv của chúng ta. Bây giờ ta sẽ thay chúng bằng `0x45 (E)` và so sánh với kết quả của xor thì đây chính là argv hoàn chỉnh của phần đầu tiên

=> `09C52R@brUc3E/123AAAAA/`

### Part2

Sau khi xong phần đầu sẽ tiếp đến kiểm tra phần tiếp theo bằng cách xác định `/`.

![image](https://hackmd.io/_uploads/BywOITpNa.png)

Sở dĩ vì sao trong argv giả để nhập vào phần tiếp lại có số `123` là bởi vì ở đây có hàm `strtol`

![image](https://hackmd.io/_uploads/Sk7JDaTE6.png)

Ở đoạn code này:

![image](https://hackmd.io/_uploads/By_x_ppNp.png)

Đầu tiên nó sẽ tính toán giá trị của `k` dựa theo những số ở argv của mình. Ví dụ đầu vào nhập vào là `123` thì sẽ được chuyển thành `112233`. Và xuống đoạn dưới thì chương trình sẽ lấy ngày hiện tại + `0x1f`. Ví dụ ngày 24 thì sẽ là `24+0x1f = 0x37`. Và sau khi hoàn thành xong thì sẽ sinh ra được một file với tên cũng là tên được lấy từ argv như sau:

![image](https://hackmd.io/_uploads/SkrVFT646.png)

Tiếp đến chương trình lại load một đoạn shellcode khá dài:

![image](https://hackmd.io/_uploads/r14IYapV6.png)

Đây chính là chương trình bên trong shellcode đó:

![image](https://hackmd.io/_uploads/rJ00KT64p.png)

#### Part3

Với lần `call` đầu tiên nhằm để xác định phần tiếp theo, tức là tuy đang kiểm tra phần 2 nhưng chương trình phải đảm bảo argv đã chứa được 3 phần bằng cách so sánh với `edx`:

![image](https://hackmd.io/_uploads/r14ScT64p.png)

Tiếp tục đến `call` tiếp theo đó là `strtol`

![image](https://hackmd.io/_uploads/Byb2c6aNa.png)

Tiếp đến `strlen`:

![image](https://hackmd.io/_uploads/HJ-xjaTVp.png)

Sau đó, chương trình sẽ có đoạn cmp ở đây. Mình cũng thực sự không hiểu nó đang làm gì nhưng đại khái sẽ có được argv như thế này: `09C52R@brUc3E/123AAAAA/2EE/` (mình phải thử rất nhiều lần)

Sau đó chương trình sẽ cho ngừng một lúc:

![image](https://hackmd.io/_uploads/SklhT6pVp.png)

#### Part4

Tiếp đến, chương trình tiếp tục kiểm tra xem argv đã có phần thứ 4 hay chưa:

![image](https://hackmd.io/_uploads/HJ6M06TVa.png)

Sau đó đây chính là phần thứ 4, nó sẽ so sánh với `pizza` nhưng chia argv ra so sánh chứ không phải so sánh tuần tự:

![image](https://hackmd.io/_uploads/Skzo066Na.png)

Ta sẽ có được argv ở hiện tại như sau: `09C52R@brUc3E/123AAAAA/2EE/3pizza/AAAAAAAAAAA/`

Tiếp đến chương trình so sánh với 0x11333377:

![image](https://hackmd.io/_uploads/rJ7vZATVT.png)

Câu hỏi đặt ra bây giờ đó là chương trình so sánh cái gì với số đó ? Nếu ta nhìn kỹ dữ liệu chương trình đem so sánh chính là những byte đầu tiên của file mà chương trình sinh ra ở phần trước đã nói:

![image](https://hackmd.io/_uploads/ryrhWA6Ea.png)

Vậy làm sao để thay đổi được nội dung đó để so sánh luôn đúng ? Ta nhìn lại ở đoạn trên 1 xíu có đoạn tính toán `k` và sau đó sinh ra được một số `112233`. Đó cũng chính là cách thay đổi dữ liệu để chính xác. Vậy bây giờ dữ liệu cần thay đổi đó là `1337` thay vì `123` 

Sau đó ta sẽ bắt gặp được đoạn xor tại đây:

![image](https://hackmd.io/_uploads/ryqsXC64p.png)

Nhưng chương trình đã khá rối rồi nên việc xem nó thực hiện thuật toán gì là điều không nên làm. Thay vào đó ta chỉ cần xem kết quả rồi xem so sánh với cái gì ở đây.

Việc so sánh ở ngay tại đây:

![image](https://hackmd.io/_uploads/HkLQEA6ET.png)

Việc so sánh này cũng khá thú vị khi không dùng cmp hay xor mà thay vào đó lại dùng sub. Và kết quả được mang đi so sánh cũng chính là kết quả xor `pr.ost`. 

Vậy argv được cập nhật `09C52R@brUc3E/1337pr.ost/2EE/3pizza/AAAAAAAAAAA/`

### Part5

Tiếp đến đoạn được cho là khoai sắn nhất ở bài:

![image](https://hackmd.io/_uploads/SkBxIC6Ep.png)

Ta sẽ nhìn một xíu ở đoạn `shellcode2` kiểu hàm rất giống `GetProcessAddress`, tìm hiểu thêm [tại đây.](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getprocaddress). Vậy "Beep" chính là tên hàm còn `v5` khả năng cao chính là nơi handle module. 

Ta nhìn ngược lại phía trên có rất nhiều phép gán ở đây:

![image](https://hackmd.io/_uploads/rkOu8AT46.png)

Từ đây ta có thể dễ dàng biết được những index của argv. Câu hỏi bây giờ được đặt ra là nó gán để làm gì? Liên quan gì đến shellcode ?

Sau nhiều lần debug với nhiều input khác nhau thì sẽ phát hiện được đó là những phép gán đó để làm cho shellcode hoạt động đúng mục đích của nó.

Đầu tiên để handle đây chính là đoạn shellcode để làm điều đó:

![image](https://hackmd.io/_uploads/HkqBuCa46.png)

Sau khi xác định được những index của argv cần nằm ở đâu thì ta sẽ điền được những vị trí cần thiết. Đoạn này đã có bài phân tích riêng nên sẽ pass qua đoạn này và chỉ cung cấp kết quả:

```
c[12] = 0x65 = e
c[5]  = 0x60 = `
c[8]  = 0x2e = .
c[9]  = 0x43 = C
c[7]  = 0x52 = R
c[6]  = 0x30 = 0
```

Tiếp đến là đoạn lặp tìm hàm trong module cũng tương tự và đây là kết quả tương ứng:

```
c[1]  = 0x4d = M
c[11] = 0x5a = Z
c[4]  = 0x45 = E
c[2]  = 0x75 = u (byte này cần phải guess)
c[3]  = 0x24 = $
c[0]  = [10] = 0x41 = A
```
Vậy argv sẽ được cập nhật ```09C52R@brUc3E/1337pr.ost/2EE/3pizza/AMu$E`0R.CAZe/ZZZZZZZZZZZZZ/```

### Part6, Part7, Part8

Ba đoạn cuối khá dễ dàng với part6 thì chỉ việc map với dữ liệu đã được set sẵn, còn part7 là so sánh với `ob5cUr3`. Và part8 so sánh với `fin`. Trong 2 đoạn cuối có đoạn crc32 để làm ra một file html. Và có một đoạn ở part4 là `3`. Đoạn này có thể chỉnh hoặc patch cho qua tùy ý. Sẽ có một đoạn so sánh trước khi qua part8 đó cũng chính là đoạn so sánh này. Tức là byte đầu tiên của phần 4 phải là ký tự tương ứng sao cho `0x1f+Day=byte1`. Hoặc có thể patch để qua đoạn này tiến vào part7. Nếu lựa chọn phương án patch thì `com` trong flag sẽ không hiện và phải guess nó là `com`. Còn nếu đúng hết thì sẽ có luôn flag.

=> ```09C52R@brUc3E/1337pr.ost/2EE/<0x1f+day>pizza/AMu$E`0R.CAZe/YPXEKCZXYIGMNOXNMXPYCXGXN/ob5cUr3/fin/```


## chall4

Ở chall4 này đa số sẽ thiên về static và guess khá nhiều, tuy nhiên việc guess sẽ phần nào đó dễ đoán hơn chall3.

Đây là GUI của file khi nhấn chạy file thử thách:

![image](https://hackmd.io/_uploads/r1JVNKVBa.png)

Khi nhấn vào "Launch ..." thì nó sẽ tự động tắt hộp thoại đi và không có việc gì xảy ra nếu như máy của mình chưa có được đủ file cũng như các đường dẫn mà thử thách yêu cầu. 

Cùng nhau nhìn qua mã giả của IDA trả về, và sẽ tập trung vào hàm `sub_402AF0`:

![image](https://hackmd.io/_uploads/HJHYEFNBp.png)

Trong hàm này có rất nhiều hàm gọi API của Win, nhưng chỉ có phần code này là đang giống với những gì mà ta đã thao tác ở lúc đầu nhất:

![image](https://hackmd.io/_uploads/S1egSFEr6.png)

Ở đây đoạn code sẽ show hộp thoại sau đó có một hàm khá lạ (`sub_402150`) và cuối cùng thoát tiến trình. Ta sẽ xem thử hàm này đang thực sư làm gì ở đây:

![image](https://hackmd.io/_uploads/HyyTIt4Hp.png)

Đập vào mắt đầu tiên đó chính là một path chứa folder với tên `Sauerbraten` và có thêm một file `sauerbraten.exe`. Thì chỉ cần cop tên folder này và tra xem thử nó là gì [ở đây](http://sauerbraten.org/)

Sau khi biết nó là một chương trình trò chơi thì ta sẽ tiến hành cài đặt, khi cài đặt xong ta sẽ tiến hành debug chương trình để xem thử thách đang làm gì với path khá lạ này.

Qua đó ta sẽ thấy được sau khi nhấn vào "Launch..." thì chương trình sẽ tạo ra những file như sau:

![image](https://hackmd.io/_uploads/ry5pX1SHT.png)

Và chúng được lưu tại `%APPDATA%\\BananaBot`, hơn nữa trước khi viết nội dung vào file thì chương trình có thêm một bước mã hóa với `AES-ECB` với key là `yummyvitamincjoy`.

Ta sẽ chỉ tập trung tất cả vào `aimbot.dll`

Đây chính là main của dll

![image](https://hackmd.io/_uploads/rJQcbxSST.png)

Như đã comment ở đoạn mã giả ta sẽ chỉ tập trung vào `sub_62F43070`. Đây chính là bên trong hàm:

![image](https://hackmd.io/_uploads/HkF6beBS6.png)

> Đoạn mã giả đã được mình đặt lại tên nên nó sẽ không được như ban đầu.

Ở đây ta có thể lựa chọn phân tích theo 2 hướng đó là debug hoặc static. Vì mình đã làm qua rồi nên mình thấy việc debug cũng không mấy tác dụng gì lắm ngoài việc có thể check một vài thứ. Nên bài này mình sẽ hoàn toàn static.

> Nếu muốn debug thì phải setting như sau:
> ![image](https://hackmd.io/_uploads/rko8zgBrT.png)
> Sở dĩ phải set như thế bởi vì để chương trình nhảy vào Main cũng như chạy được file dll.

Sau khi lấy thời gian thực, kiểm tra cũng như tạo folder thì đến hàm đầu tiên của chương trình:

![image](https://hackmd.io/_uploads/r1hsXgrrT.png)

Ta sẽ thấy được có một hàm `get_num` và trong hàm này chứa anti-debug:

![image](https://hackmd.io/_uploads/BJDk4lBS6.png)

Thấy thêm một hàm `sub_62F42020` khá lạ, khi vào trong hàm này có rất nhiều chức năng khá phức tạp, nên ta sẽ tạm back trở lại và xem tiếp hàm tiếp theo:

![image](https://hackmd.io/_uploads/rJ9YVeSra.png)

Ở đây ta thấy có một hàm mình đã rename lại với tên `xor2`, nhưng giả sử bây giờ ta sẽ không biết hàm này là gì. Thì hàm này lấy kết quả của `get_num` để thực hiện điều gì đó ở đây. Và đây chính là đoạn code trong hàm đó:

![image](https://hackmd.io/_uploads/ByVgSgrHa.png)

Đoạn code này đang thực hiện 1 phép xor, với dữ liệu được lấy từ `off_62FE4020[1]` và `a3`(kết quả của `get_num`). Nhưng sau đó kết quả sau khi xor thì làm gì ?? Phải giải quyết được câu hỏi này thì mới có thể lấy giá trị sau khi xor tìm lại được số mà ta đang đi tìm. Đê trả lời câu hỏi này ta lại nhìn xuống đoạn code tiếp theo:

![image](https://hackmd.io/_uploads/ryqWA48S6.png)

Ở đây, ta thấy được một đoạn code tương tự như ở trên nhưng ở trên thì sau khi xor kết quả được lấy để làm đối số cho API có tên `InternetOpenA`, còn ở dưới thì sau khi xor được kết quả thì kết quả được lấy để làm đối số cho API với tên `InternetOpenUrlA`. Có thể tham khảo thêm [tại đây](https://learn.microsoft.com/en-us/windows/win32/api/wininet/nf-wininet-internetopenurla).

Sau khi biết được `v5` là `lpszUrl` thì bây giờ việc cần làm là bf để tìm được giá trị của `get_num`. Và đây chính là đoạn code thực hiện:

```python
import struct

data = b'\xc1\x8c\xed\x14\x93\xd7\xb6U\x9b\xcf\xb7T\x87\xc8\xb7U\x93\xcd\xaeW\x9b\xc0\xb6V\x86\x8b\xec\t\xc4\x99\xeb\x1d\xa9\xfb\x9ag'
pck = []

for i in range(0, len(data), 4):
    tmp = b''
    pck.append(struct.unpack('<I', data[i:i+4])[0])

for num in range(0, 0xffffffff, 1):
    tmp = b''
    for pk in pck:
        tmp1 = pk^num
        tmp += struct.pack('<I', tmp1)
    flag = 0
    for i in range(len(tmp)-4):
        if tmp[i] < 0x20 or tmp[i] > 0x7f:
            flag = 1
            break
    if (flag == 0):
        if b'http://' in tmp:
            print(f"{tmp}  {num}")
            exit(0)
```
> Mình code khá kém nên mất tới gần 50p mới ra được số cần tìm. Và số đó chính là:
> ![image_2023-11-30_17-21-19](https://hackmd.io/_uploads/rys9eS8Sa.png)

Sau khi có được số chúng ta đang cần, ta sẽ xor nốt với các dữ liệu còn lại ở `off_62FE4020`:

```python
data = b'\xc1\x8c\xed\x14\x93\xd7\xb6U\x9b\xcf\xb7T\x87\xc8\xb7U\x93\xcd\xaeW\x9b\xc0\xb6V\x86\x8b\xec\t\xc4\x99\xeb\x1d\xa9\xfb\x9ag'
data1 = b'\xcb\x99\xf7\x05\xc7\x99\xfb\x0b\xdd\xd8\xacT\x99\xc8\x99e'
data2 = b'\x8b\x8e\xfc\x16\xda\x91\xf6\n\x8b\xc2\xb9F\xa9\xfb\x9ag'
data3 = b'\xdd\x90\xfcD\xcd\x9d\xfa\x16\xd0\x88\xed\r\xc6\x96\xb9\x0b\xcf\xd8\xed\x0c\xc0\x8b\xb9\x06\xc5\x97\xfbD\xde\x99\xeaD\xda\x8d\xfa\x07\xcc\x8b\xea\x02\xdc\x94\x99e\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'

def dec(tmp):
    res = b''
    for i in range(0, len(tmp), 4):
        tmp1 = 0x6499f8a9^struct.unpack('<I',tmp[i:i + 4])[0]
        res += struct.pack('<I',tmp1) 

    return res

print(dec(data))
print(dec(data1))
print(dec(data2))
print(dec(data3))

# b'http://127.0.0.1:57328/2/summary\x00\x03\x03\x03'
# b'bananabot 5000\x00\x01'
# b'"version": "\x00\x03\x03\x03'
# b'the decryption of this blob was successful\x00\x01\xa9\xf8\x99d\xa9\xf8\x99d\xa9\xf8\x99d'
```

Sau khi có được kết quả ta lại tiếp tục đến phần code tiếp theo:

![image](https://hackmd.io/_uploads/HkR2OSUBp.png)

Như đã đặt lại tên toàn bộ hàm, thì ở đây chương trình đang cố gắng giải mã một đoạn shellcode bằng `AES_ECB`, và sau cùng so sánh 42byte đầu của đoạn shellcode vưa giải mã với kết quả được lấy từ đoạn `dec` ở `data3` tức là `the decryption of this blob was successful`. Sau đó chương trình sẽ thực thi shellcode nằm ở byte thứ 43 trở đi của đoạn shellcode vừa được giải mã bằng `AES_ECB`. Vậy bây giờ ta cần phải tìm được key của `AES` để có thể giải mã đoạn shellcode này. Việc tìm key ta cũng hoàn toàn bf, bởi đã biết được cipher cũng như plaintext:

```python
plaintext = b"the decryption of this blob was successful"
key1 = '"version": "'
alphabet = ['1','2','3','4','5','6','7','8','9','0','.']
alt = ["".join(i) for i in product(alphabet,repeat=4)]
data = b'\xa8;\x15G~\xd2\x99\xbc\x05\x9d\xc5\x87e\x9c\x0f\x90y\x9b\xf7\xb0\x8b\xb6js\xd6\x05[\xd0\x1e\xcc\xc6\xe2\x06eQ\xeb\xa1$\xc8\xc6\xed\xab.884o\xa8'
for key2 in alt:
    keymain = (key1 + key2).encode()
    pl = AES.new(keymain, AES.MODE_ECB).decrypt(data)
    if plaintext in pl:
        print(keymain)
        exit(1)

# b'"version": "6.20'
```
> Đoạn data sẽ được lấy từ `datashellcode` của ` memcpy(shellcode, &datashellcode, 0x4470ui64);` và chỉ lấy 48 byte đầu tiên. Còn việc biết được `"version": "` là key1 là bởi nhìn vào mã giả thì thấy được rằng `key` sẽ liên quan tới `v5` và `v5` chính là kết quả tìm kiếm `v4` trong `v2` mà `v4` là kết quả xor[2] *(b'"version": "\x00\x03\x03\x03')* nên ta chỉ việc bf 4 bytes còn lại của key.

### shellcode1

Sau khi có được key, bỏ toàn bộ đoạn shellcode vào sau đó giải mã và viết vào 1 file mới ta sẽ có được mã giả của đoạn shellcode đầu tiên như sau:

![image](https://hackmd.io/_uploads/rJ4U19wHp.png)

> Ta lướt nhanh qua các hàm con ở phần shellcode đầu tiên này sẽ bắt gặp được thuật toán `RC4` và `ror13`.

Vì các hàm ở shellcode đã bị strip toàn bộ nên việc rev kỹ từng hàm là không thể, nên ta sẽ chỉ đoán dựa trên path và những symbol có được. Như trong đoạn này thì chương trình đang tìm file `config.vdf` sau đó sao chép file này sang `C:\\depot\\steam_ssfn`. Sau đó tại đoạn mã này:

![image](https://hackmd.io/_uploads/ByU_g5wBa.png)

Chương trình đang thực hiện giải mã một đoạn shellcode thứ 2 với key là 16 byte đầu tiên của tệp `config.vdf`. Sau khi giải mã xong, sẽ so sánh 42 bytes đầu tiên với chuỗi `the decryption of this blob was successful`. Vậy bây giờ ta sẽ lấy shellcode thứ 2 và key tiến hành giải mã và tạo ra một file tiếp theo

> key = b'"InstallConfigSt'
> shellcode2: `unk_929`

### shellcode2


## chall5

*Vì bài này mặc dù mình đã tắt ASLR đi rồi nhưng vì VirtualAlloc khá nhiều nên địa chỉ mỗi lần sẽ nhảy địa chỉ khác nhau, và bài này mình chia thành nhiều ngày để viết nên sẽ chỉ ghi offset của nó.*

- Đầu tiên ta sẽ chạy chương trình xem thử đang làm gì ở đây:

![image](https://hackmd.io/_uploads/Hy5qLn-20.png)

- Dường như chỉ pop up lên message box. Mở IDA xem thử nó có gì ở đây:

![image](https://hackmd.io/_uploads/SyYOwhWn0.png)

- Nhìn qua thì chả thu được gì, đặt bp tại đầu winmain xem thử chương trình đang thật sự làm gì. Chương trình lập tức thoát ngay và còn lại mỗi message box và đi kèm đó là rất nhiều thread được chạy khi chạy chương trình:

![image](https://hackmd.io/_uploads/S1DfunW20.png)

- Có vẻ như có một vài thứ chạy trước cả khi chạy ở winmain.
- Tiến hành kiểm tra xem chương trình có đang chạy kèm gì ở trong hay không bằng [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) và đây là kết quả trả về từ tool:

![image](https://hackmd.io/_uploads/rkqD5hWnC.png)

- Nhìn chung thì chả có gì đặc biệt ngoại trừ tạo mới tiến trình `explorer.exe`
- Chuyển sang capa để xem chương trình liệu có dùng mã hóa gì ở đây không. Đầu tiên sẽ thấy được RC4:

![image](https://hackmd.io/_uploads/HyVXlaZh0.png)

- Tiếp đến change mem RWX (*có khả năng cao có shellcode*):

![image](https://hackmd.io/_uploads/ByxLg6W2C.png)

- Có virtualAlloc:

![image](https://hackmd.io/_uploads/HJJ3xpZ30.png)

- Nhìn qua cũng đoán được chắc chắn có shellcode. Bây giờ xem thử RC4 trong chương trình thì nó nằm ở đâu và ở đâu gọi RC4, ta thấy được `sub_448650` gọi `RC4`. Đặt bp ngay tại hàm này để xem được key cũng như data lấy từ đâu.
    - Data:`0x0448AC6` là nơi load shellcode vào với len `0x6C44` và sau đó `VirtualAlloc` với option quyền `0x40` (*cái này tự tìm hiểu mình sẽ không đề cập đến nhiều vì khá dễ*). Tiếp đến chương trình sẽ load đoạn shellcode vào mem vừa cấp phát (*có thể xem giá trị `eax` ngay sau gọi `VirtualAlloc` để biết rõ địa chỉ*). 
    - Key: `67 C6 8F 1E DA AD 12 60 00 00 00 00 00 ... (255)`
- Tiếp đến tại `0x00448725` là nơi gọi shellcode vừa decrypt. 
- Debug liên tục sẽ tới được những offset với chức năng như sau:
    -  `0x415B`: Load encrypted shellcode2
        -  `0x12E4`: FindResourceA
        -  `0x02263277`: SizeofResource (0x15160)
        -  `0x02265D88`: LoadResource (0x0048B67C)
        -  `0x022663CD`: LockResource
        -  `0x022611AC`: VirtualAlloc (0x022B0000)
        -  `0x02265920`: end.
    - `0x022669DF`: decrypt shellcode2
        - `0x0226592A`: GetProcessHeap
        - `0x0226594B`: HeapAlloc
        - `0x02265832`: Init key RC6. Biết được RC6 bởi vì thấy được những hằng số như `0x0B7E15163` cứ lặp lại và xuất hiện trong mỗi lần lặp, chỉ cần cop số này lên google sẽ dễ dàng thấy được của RC6. Tại `0x02265384` so sánh với 0x24.
        - `0x022605CD`: decrypt
        - `0x02266419`: end.
        - Sau khi decrypt thì đoạn shellcode sẽ có data dạng như sau:
        ![image](https://hackmd.io/_uploads/BywhRQw20.png)
- Cứ tiếp tục debug sẽ tới địa chỉ `0x02262E57` thì sẽ thoát chương trình. Debug lại và đặt bp tại đây, trong lần call này với những chức năng như sau:
    - `0x02265CA6`: OpenMutexA (name: `welcome_main`)
    - `0x02266A59`: CreateProcessA (lpCommandLine: `Explorer.exe`, [CREATE_SUSPENDED](https://learn.microsoft.com/en-us/windows/win32/procthread/process-creation-flags))
    - `0x00671A9D`: VirtualAllocEx (flAllocationType: MEM_COMMIT | MEM_RESERVE, dwSize: 0x15160, flProtect: PAGE_EXECUTE_READWRITE)
    - `0x00676538`: WriteProcessMemory (nSize: 0x1512F, bắt đầu viết ở offset 0x31 của shellcode2, *các thông số khác tự theo dõi vì khác nhau mỗi lần chạy*)
    - `0x00674BC6`: QueueUserAPC
    - `0x00673601`: ResumeThread
- Tới đây, vì khi CreateProcessA có option CREATE_SUSPENDED nên khi gặp ResumeThread thì tiến trình `explorer.exe` sẽ chạy. Sở dĩ biết được `explorer.exe` sẽ chạy mà không phải tiến trình khác bởi vì lúc đầu chạy với monitor thì chương trình chỉ sinh ra thêm 1 process mới đó là explorer.exe. Thêm nữa việc WriteProcessMemory chỉ có thể viết vào explorer.exe chứ không còn process nào khác.
- Công việc bây giờ là dump được shellcode 2 ra xem có gì và tìm cách debug được shellcode 2. Để dump được shellcode 2 có 2 cách. 
    - Cách 1: dump thẳng từ task mananger. Đối với cách này chỉ cần mở task mananger và tìm explorer.exe sau đó chuột phải và dump. Tiếp đến muốn tìm đúng địa chỉ của shellcode 2 thì tra segment `debug RWX`
    - Cách 2: dump từ lúc WriteProcessMemory sử dụng thẳng IDA, vì đã biết len của shellcode 2
- Khi có được shellcode 2 việc static là không thể, tiến hành debug bằng cách attach thẳng vào explorer.exe sau đó nhảy đến địa chỉ load shellcode. Để xem được địa chỉ nào thì xem WriteProcessMemory tham số `lpBaseAddress`. Sau khi có được địa chỉ đặt bp ngay tại địa chỉ đó sau đó sang chương trình chính đặt bp tại ResumeThread. Tiêp tục debug đến khi nào IDA trả về `thread has started` thì sang explorer.exe F8 để ăn vào bp.
- Qúa trình debug shellcode 2 có được những kết quả sau:
    - `0x0280007D`: xor sth. Giá trị xor được lấy từ base `0x02800655`. Kết quả sau khi xor được lưu tại base `0x02800665` *(nhìn vào ins mov tại `0x0280049D`)*. Lần xor này lặp với idx = 8
    - Tiếp tục tại ins có addr `0x0280044F` thực hiện xor base `0x02800655` với base `0x0280066D`. Lần xor này lặp với idx = 0x14AC2
    - Tiếp đến `0x02800511` gọi shellcode 3
- Debug shellcode 3 có được những kết quả sau:
    - Ở đây có ror 13 có vẻ như đang hash những api cần thiết của chương trình
    - `0x02AC10A6`: thấy được push những hằng số giống như VirtualAlloc. Đặt bp tại đây, thấy được `0x02AC10B2` đang gọi VirtualAlloc. Đặt tiếp bp tại địa chỉ VirtualAlloc vừa cấp phát.
    - Sau khi cấp phát xong chương trình load những api như `GetProcAddress`, `LocalFree`, `DisconnectNamedPipe`,...
    - `0x02AC1291`: gọi shellcode 4
- Debug shellcode 4 có được những kết quả sau:
    - ins call tại `0x02DA18C6` lấy những thông tin `kernel32_GetSystemTimeAsFileTime, kernel32_GetCurrentThreadId, kernel32_GetCurrentProcessId, kernel32_QueryPerformanceCounter`
    - Đặt bp `0x02DA17F2` sẽ thấy được những kết quả sau:
        - `0x02DA108B`: push `"\\\\.\\pipe\\\\whereami"`
        - `0x02DA1092`: kernel32_CreateNamedPipeA
        - `0x02DA109D`: kernel32_ConnectNamedPipe
        - Tới đây chương trình sẽ treo, đợi đến khi nào IDA trả về `thread xxx has exited` thì sang bên chương trình shellcode 1 debug tiếp để tạo ra pipe mới có thể connect.
        - Debug shellcode 1 tới `0x006C250C`: CreateFileA (lpFileName: $\.\...\.p.i.p.e.\.w.h.e.r.e.a.m.i)
        - shellcode 4 `0x032810B4`: ReadFile, tiếp tục chạy debug ở shellcode 1 để viết nội dung vào file (`0x006C5565` shellcode 1)
        - `0x032810BB`: kernel32_DisconnectNamedPipe
        - `0x032810C2`: kernel32_CloseHandle
        - `0x032810D8`: MessageBox (FLARE-ON 10: where_am_i?)
        - `0x032810E2`: advapi32_GetUserNameA
        - `0x006110F6`: compare `username` vs `flare`
        - `0x0061111D`: push C:\\Users\\Public\\ 
        - Đặt file ở path `C:\\Users\\Public\\ `
        - `0x00611134`: compare `0x0BAADBEEF`
        - Sau cùng sẽ MessageBox 
        ![image](https://hackmd.io/_uploads/S1I4XYunA.png)
- Sau khi MessageBox hiện lên thì cũng kết thúc chương trình, vậy flag nằm ở đâu ???
- MessageBox chứa RC6 vừa hay khi lúc shellcode1 có đoạn RC6 decrypt shellcode2 và trong lúc debug ở đoạn init key sẽ thấy khá nhiều `0x0BAADBEEF` lặp lại. Trong shellcode 4 này có đoạn cũng check `0x0BAADBEEF` khả năng cao đây là key. Vậy để tìm flag cuối cùng chỉ cần vứt đoạn shellcode 4 này vào đoạn decrypt của shellcode 1 là được. Có 2 cách làm ở đoạn này. 
    - Đầu tiên có thể làm lại hàm decrypt RC6 và lấy key
    - Thứ hai có thể thay toàn bộ shellcode 4 vào đoạn chuẩn bị đưa shellcode 2 vào giải mã, thay toàn bộ shellcode 2 thành shellcode 4 thì sẽ thấy flag.

## chall 6

*Trước khi đến với chall này hãy xem qua **DOS là gì**,  **DOS trong PE có dạng như thế nào**, **làm thế nào để debug một file DOS**.*

- Đầu tiên, chạy thử chương trình nhưng dường như không hiển thị lên bất cứ thứ gì cũng như kiểm tra xem có chạy ngầm hay không cũng không tìm thấy.
- Phân tích file với CFF ta sẽ thấy được EntryPoint `0x00008F40` và `invalid`.  Cũng như có rất nhiều section với value là 0 như `sizeOfCode, sizeOfStackCommit,...`. Điều đặc biệt ở đây `e_flanew` có value khá lớn và không hề thấy những section `.data, .text,...` ở bất cứ đâu:
![image](https://hackmd.io/_uploads/rJytOUbaC.png)
- Tuy phần DOS khá nhiều, nhưng như thường lệ load vào IDA phân tích với option PE đầu tiên để xem có gì ở đây:
![image](https://hackmd.io/_uploads/H1gcKUZaC.png)
- Như đã nói ở trên ở đây chỉ toàn `HEADER` không có những section nào khác. Nhảy đến EntryPoint nhấn `C` rồi nhấn `P` để IDA tự động chuyển thành mã giả và tạo hàm. Và đây là kết quả IDA trả về sau khi rev cũng như rename lại sẽ được như sau:
![image](https://hackmd.io/_uploads/r1iVxPW60.png)
- Có thể thấy được chương trình đầu tiên làm một loạt phép tính sau đó check với `0x31D9F5FF`, sau khi check thành công lấy 84 bytes để decrypt và cuối cùng in ra MessageBox Winning.
- Vào kiểm tra hàm decrypt thấy được kết quả như sau:
![image](https://hackmd.io/_uploads/r1ZTeDWaA.png)
- Có thể thấy ở đây có "rất rất" nhiều antidebug. Thật sự thì mình cũng chả biết bypass mớ này như nào và cũng không rõ thuật toán ở đây là gì và dùng capa càng không được. 
- Vậy là ở phần PE chả thu được gì, chuyển qua phân tích DOS. Như đã nói ở trên phần DOS khá dài và dài hơn những file bình thường, chắc có lẽ sẽ có điều gì đó ở đây. Load vào IDA với option DOS ta sẽ được như sau:
![image](https://hackmd.io/_uploads/HJDgSwZ6A.png)
- Kết hợp với debug ta sẽ có được những kết quả như sau:
    - Đầu tiên tại `0x0055` chính là set up `Splash Screen`:
    ![image](https://hackmd.io/_uploads/Bk0kYEXpR.png)
    - Ở đây sau quá trình debug và rename ta sẽ có hàm `check_input_Init` tại `0x08BE`. Tại đây chương trình sẽ như sau:
    ![image](https://hackmd.io/_uploads/HJfUtVX6C.png)
    - Sau khi user nhập vào chương trình sẽ dùng `eax` để lưu sau đó chia làm 2 loại để check. Đầu tiên check với `al`, các giá trị được so sánh lần lượt là `A` và `B`. Tiếp đến là [scan code](https://www.win.tue.nl/~aeb/linux/kbd/scancodes-1.html) được lưu tại `ah` và so sánh với `H, P, K, M`. Nếu so sánh đúng thì chương trình sẽ lưu tại `[di]` và đến khi nào đủ 11 ký tự thì thôi. Sau khi đọc xong chương trình sẽ so sánh với `HHPPKMKMBA` nếu đúng thì sẽ làm một số phép toán như `shr, add` nếu không đúng thì chương trình sẽ nhảy vào random. *Vậy sau khi rev thì chuỗi cần nhập ở đoạn này sẽ là `up, up, down, down, left, right, left, right, F8, F7` hoặc `up, up, down, down, left, right, left, right, B, A`*. Cuối cùng chương trình sẽ di chuyển giá trị vừa tính toán này vào một biến được rename là `seed`:
    ![image](https://hackmd.io/_uploads/ry6P347aA.png)
    - Sau khi Splash Screen xong tiếp đến phần chính của game `0x0058`. Ở đây sau khi rev và rename ta sẽ được một số thứ như sau:
    ![image](https://hackmd.io/_uploads/S174zHmpA.png)
    ![image](https://hackmd.io/_uploads/By_rMSXaA.png)
    - Với rename như trên có thể thấy được tổng quan của chương trình. 
    ![image](https://hackmd.io/_uploads/S1A_XSm60.png)
    - Đầu tiên sẽ hiển thị ra messageBox (Welcome to Flare Says, where the points,...). 
    ![image](https://hackmd.io/_uploads/H1c8QSQaR.png)
    - Tiếp đến sau khi so sánh xong level hiện tại cũng như in ra màn hình chính level và score hiện tại thì chương trình sẽ tự động gen ra input tiếp theo. Input này phụ thuộc hoàn toàn vào giá trị của seed ban đầu người dùng đã nhập ở Spash Screen.
    ![image](https://hackmd.io/_uploads/HJ1W4rQT0.png)
    - Tiếp theo chương trình sẽ tự động chơi trước với input được sinh ra.
    ![image](https://hackmd.io/_uploads/SyHXVSXa0.png)
    - Sau khi chơi xong, người dùng sẽ nhập vào và chương trình kiểm tra xem input của user có giống với input của chương trình đưa ra hay không. Nếu giống thì update điểm cũng như level, nếu không thì chương trình sẽ thoát và hiển thị messageBox (Never Give Up!!!). 
    ![image](https://hackmd.io/_uploads/BJRCrr7pC.png)
    - Cứ thế đến hết 128 vòng, nhưng khi chiến thắng chương trình lại có một hàm `Write_to_FlareSay_exe` tại `0x0445`.
    ![image](https://hackmd.io/_uploads/SyJ78rX60.png)
    - Tại đây ta thấy được chương trình đang handle chính nó để ghi một thứ gì đó vào offset đã được định sẵn. Biết được handle chính nó là nhờ vào kết quả của phép xor trước khi handle chính nó. Hơn nữa còn phát hiện thêm 1 dãy 16 bytes (0xCC) cũng có mặt tại đây. Có thể đoán được một thứ gì đó sẽ được ghi vào ngay tại vị trí này. Tiến hành mở IDA với option PE lên và check xem đó là ở đâu. Có vẻ như nó sẽ nằm tại đây:
    ![image](https://hackmd.io/_uploads/BkepDSXpA.png)
- Phân tích dường như đã xong, bây giờ việc chơi hết 128 vòng là không thể, ta sẽ nghĩ đến việc để chương trình tự chơi và tự hoàn thành. Vì chương trình tự gen ra input tiếp theo dựa vào seed đã nhập trước đó nên bây giờ chỉ cần patch đoạn nhập ngay tại Splash Screen để tránh random và patch tiếp đoạn chương trình yêu cầu nhập vào sau khi chương trình tự chơi. Thế là hoàn thành.
![image](https://hackmd.io/_uploads/rkfYYBX6A.png)
![image](https://hackmd.io/_uploads/SyPiFB7TC.png)
- Sau khi chạy xong 128 vòng, mở file với window sẽ hiển thị flag.


## chall8

*Đây có thể là chall cuối cùng bản thân hoàn thành trong fl10, đáng nhẽ sẽ không làm về thử thách này vì đã hết thời gian cũng như ngày mai sẽ diễn ra fl11, nhưng bản thân khá thích và ngưỡng mộ author nên quyết định bỏ ra 2 ngày để hoàn thành cho bằng được. Và cũng chính thử thách này phải chạy hơn 30km để sửa máy =))))*

Đầu tiên ta sẽ chạy chương trình và tra theo Process Monitor ta sẽ thấy được như sau:

![image](https://hackmd.io/_uploads/H1p9oS4RC.png)

Tuy nhiên ở thư mục chứa các file không chỉ có `Procmon64.exe` bị mã hóa mà còn thêm một file nữa bị mã hóa:

![image](https://hackmd.io/_uploads/Bklp3S4C0.png)

Khi chạy chương trình `rustup-init` này thì đây là kết quả trả về:

![image](https://hackmd.io/_uploads/S1cfTrNRR.png)

Còn chương trình `Procmon64` được sinh ra bởi mal thì không in ra được gì. Vậy có thể đoán được mal này gồm 2 phần. Một phần để enc file bình thường thành fake flag, phần còn lại đang làm gì đó. 

Bây giờ sẽ phân tích đầu tiên từ file mal bằng IDA:

![image](https://hackmd.io/_uploads/r1W1k8VA0.png)

Ở đây, hàm `sub_7FF637C17A50` sẽ thực hiện enc file. Ta cùng xem bên trong hàm này đang thực hiện điều gì:

![image](https://hackmd.io/_uploads/Hk7u1IVAR.png)

Đầu tiên chương trình sẽ lấy path `StartUp` sau đó cop file `svchost.exe` vào folder này.

![image](https://hackmd.io/_uploads/BknRJINCA.png)

Tiếp đến chương trình sẽ tiến hành sửa đổi file này bao gồm cả icon và nội dung file.

Sau đó tại hàm `sub_7FF637C16780` sẽ thực hiện duyệt qua các file khác và tiến hành mã hóa các file bằng cách lấy giá trị của `byte_7FF637C4BC00` xor với `@cPeterr`.

Tới đây sau khi chương trình xor xong và mã hóa xong file thì mình đã đặt bp tại đây để xem thử sau khi mã hóa xong thì làm gì tiếp theo:

![image](https://hackmd.io/_uploads/rygoZIV0R.png)

Nhưng chương trình ngay lập tức thoát và mình cũng chả hiểu tại sao. Quyết định chuyển hướng sang tìm hiểu xem file bị mã hóa và file được sinh ra bởi mal (xem với Monitor) đang làm gì. Đây là kết quả trả về từ IDA file bị mã hóa :

![image](https://hackmd.io/_uploads/ryxnzL40R.png)

Nhìn qua thì chả có gì quan trọng ngoại trừ một link rick roll =))))

Tiếp đến phân tích với file được sinh ra bởi mal:

![image](https://hackmd.io/_uploads/rkJGLL4C0.png)

![image](https://hackmd.io/_uploads/SkNz88N0C.png)

Toàn bộ mình đã rev lại, nhìn sơ qua thì chương trình đang mở port ở local port 8345 sau đó đợi client connect tới. Khi client connect thành công sẽ spawn một thread mới để xử lý.

Đến đây mình lại stuck vì mình không rõ tạo thread mới sẽ xử lý connect tới như thế nào. Chuyển sang phân tích gói pcapng. 

Ở đây sẽ filter tất cả các port 8345:

![image](https://hackmd.io/_uploads/BJkmfPV0C.png)

Follow theo TCP stream ta sẽ thấy được như sau:

![image](https://hackmd.io/_uploads/B1NVEvEA0.png)

Ta thấy được rằng ở đây có gửi dữ liệu lên và nhận `ACK_K` trả về từ server. Bây giờ ta sẽ tìm xem đoạn nào trong chương trình thực hiện các chức năng này.

Sau một hồi tìm kiếm với strings thì phát hiện có hàm `sub_140001490` thực hiện việc này. Tiến hành đặt bp và để debug (*kết hợp dùng pwntool để connect và gửi data*) kết hợp static ta sẽ rev được như sau:

![image](https://hackmd.io/_uploads/BJZpvD4AA.png)

![image](https://hackmd.io/_uploads/HkdpDPERR.png)

![image](https://hackmd.io/_uploads/Sk3Twv40R.png)

![image](https://hackmd.io/_uploads/H1lRPw4AR.png)

![image](https://hackmd.io/_uploads/rkS0wDE0R.png)

Hàm trên khá dài nhưng kết hợp rev sơ và nhìn vào TCP stream có thể hiểu được chương trình đang hoạt động như sau. Đầu tiên victim mở kết nối port 8345 sau đó chương trình sẽ spawn một thread để handle client. Sau khi handle client chương trình đợi client gửi data đến lần đầu tiền (key) sau khi có được data đầu tiên (key) thì chương trình sẽ chuyển về cho attacker "ACK_K". Tương tự đến lần thứ 2 là gửi nonce sau đó gửi lên các lệnh như "exec, upload, ..." thì chương trình sẽ kiểm tra xem có thỏa mãn hay không. Nếu thỏa mãn chương trình sẽ thực thi. Sau khi gửi command cũng như data thành công trước khi phiên làm việc kết thúc attacker sẽ xóa hết folder đã tạo.

Vậy chương trình đã rõ, bây giờ chỉ việc viết script gửi lên server đã được mở. Với data sẽ extract từ file pcapng. Trong đó bao gồm cả key và nonce cũng như các file ps1 và png cần thiết kết hợp với chỉnh lại đường dẫn. Khi thực hiện xong gửi lên sẽ nhận được flag.

```python=
from pwn import *
data = """get data from pcapng """

data2 = """get data from pcapng"""
io = remote("your ip", 8345)
try:
   io.send(bytes.fromhex("6574212c9b4d9334d893bec2477cb86a70983b3c33952d68a8cc5c0226070abf"))
   print(io.recv(100))
   io.send(bytes.fromhex("0e02f4a9a8b5beeaba8348d6d2f87c606849df9a5eef49a65c98cf07d4c238a6"))
   print(io.recv(100))
   io.send(bytes.fromhex("path file data"))
   print(io.recv(100))
   io.send(bytes.fromhex(data))
   print(io.recv(100))
   io.send(bytes.fromhex("path file data2"))
   print(io.recv(100))
   io.send(bytes.fromhex(data2))
   print(io.recv(100))
except:
   exit()
```
