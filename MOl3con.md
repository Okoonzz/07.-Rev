# Molecon 2023

> Đề:
> ![](https://hackmd.io/_uploads/B1el0V2Sn.png)

Tải file [tại đây.](https://github.com/Okoonzz/07.-Rev/blob/File/produc_key_check.zip)

Trước khi giải bài này, hãy xem qua đoạn code sau:

```
void decimal_to_hexadecimal(int decimal_num) {
    int i = 0, quotient, remainder;
    char hexadecimal_num[100];

    while(decimal_num != 0) {
        quotient = decimal_num / 16;
        remainder = decimal_num % 16;
        if(remainder < 10) {
            hexadecimal_num[i] = remainder + 48; 
        } else {
            hexadecimal_num[i] = remainder + 55; 
        }
        i++;
        decimal_num = quotient;
    }

    for(int j = i - 1; j >= 0; j--) {
        printf("%c", hexadecimal_num[j]);
    }
}
```
> Đây là đoạn code chuyển từ hệ 10 sang hệ 16. Nếu như bình thường thì không có gì để nói ở đoạn code này, nhưng hãy nhìn vào bản chất của nó. Ở các đoạn code *+55*  và *+48* nếu ta chỉ nhìn nó theo hướng học thuộc hoặc một số magic nào đó thì không hề liên quan đến bài này. Bản chất của việc cộng những số ấy là để tương ứng với ký tự trong bảng ASCII, vì bảng ASCII đã có sẵn nên không cần phải định nghĩa lại nó nên chỉ cần cộng 48 và 55 thì có thể chuyển đến những ký tự tương ứng.

Quay lại bài toán, ở đây ta có được đoạn mã của chương trình như sau:

![](https://hackmd.io/_uploads/B1y9kIhSh.png)

Đầu tiên, chương trình yêu cầu nhập vào key, qua 5 lần kiểm tra chính, nếu đúng sẽ có được flag của bài. Chuẩn key nhập vào phải có chiều dài là 40 và được chia làm 5 phần mỗi phần cách nhau bởi dấu "-".

Kiểm tra các hàm `sub_DBBD`, `sub_E03E`, `sub_E196` thấy rằng tham số truyền vào rất đặc biệt đó là tham số đầu tiên là chiều dài của mỗi phần tương ứng, tham số thứ hai là mảng key của phần đó.

Lần lượt 3 lần kiểm tra của chương trình trên theo thứ tự sau:
* Đầu tiên kiểm tra phần thứ nhất phải có độ dài là 3 và phải khác lần lượt các bộ số "123", "456", "789"
* Tiếp đến phần thứ hai phải có độ dài là 7 và tổng các chữ số phải chia hết cho 7
* Phần thứ 3 kiểm tra xem có phải là bộ ba các chuỗi "ptm", "ctf", "plt" hay không.
> Có một điều thú vị ở đây đó là có những đoạn code như sau:
> ```
>   for ( i = 0LL; i < 3; ++i )
>   {
>     v4 = &v5[2 * i];
>     if ( *v4 == a1 && !memcmp((const void *)v4[1], a2, *v4) )
>       return 0LL;
>   }
> ```
> Đoạn code trên lấy những phần tử chẵn để làm độ dài chuỗi và những phần tử lẻ chính là chuỗi đó. Chúng tương đương với đoạn code sau 

```
    int arr[6] = {3, 456, 3, 345, 3, 301};
    int *ptr;
    int *ptr2;
    ptr = arr;
    for (int i = 0; i < 3; ++i){
        ptr2 = &ptr[2*i];
        printf("%i %i \n", *ptr2, ptr2[1]);
    }
    #output: 3 456 
             3 345
             3 301
```

![](https://hackmd.io/_uploads/SkGcOsl82.png)

Lượt check cuối cùng rất nhiều tham số được truyền vào, trong đó thấy rằng hai tham số `v12` và `v14` được truyền vào mà những tham số này được lấy từ chung một hàm encrypt `sub_E282`.

Ở hàm `sub_E282` việc đầu tiên nhìn thấy đó là một chuỗi có độ dài là 24. Dưới đó có thêm một mảng gồm 24 ký tự. Tiếp đến là có những biến được "/16", khả năng cao "/16" này chính là chuyển về hex. Như đã nói ở đầu, vừa chuyển về hex kết hợp có một bảng gồm 24 ký tự thì có thể đoán rằng đây chính là base24.

Đoạn kiểm tra này biến `v12` được truyền vào chứng tỏ giữa hai phần kiểm tra 4 và 5 sẽ có liên quan với nhau. Tiến hành debug và nhập key tương ứng khác nhau mỗi lần thì sẽ thấy rõ được phần 5 kiểm tra hoàn toàn phụ thuộc vào phần 4.

> *Đặt breakpoint trước vị trí cmp và xem tại `rsi`.*

Đây là script để giải bài này:

```
def base24_encode(num):
    charset = ['B','C','D','F','G','H','J','K','M','P','Q','R','T','V','W','X','Y','2','3','4','6','7','8','9']
    base = len(charset)
    result = ''
    while num > 0:
        digit = num % base
        result = charset[digit] + result
        num //= base
    return result

# key: 756-1234567-ptm-99999-GFMGM6R73K6M8JKR6M
print(base24_encode(0xFEE32BF827D21830F4A8))
```

=> Qua thử thách này giúp nhận biết được base24 cũng như cách lấy một phần tử ở vị trí chẵn lẻ.
