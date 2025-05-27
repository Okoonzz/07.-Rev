# Đôi chút về Windows process 

## Hình dung sơ bộ

Tài liệu được tham khảo tại đây.

https://github.com/Faran-17/Windows-Internals/blob/main/Processes%20and%20Jobs/Processes/PEB%20-%20Part%201.md

https://lequangkhai.wordpress.com/2010/10/12/process-management-part-1-processes-ti%E1%BA%BFn-trnh/#:~:text=M%E1%BB%99t%20process%20bao%20g%E1%BB%93m%20c%C3%A1c,dung%20c%E1%BB%A7a%20c%C3%A1c%20processor's%20registers.

https://securitycafe.ro/2015/10/30/introduction-to-windows-shellcode-development-part1/

https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getprocaddress


Nhìn chung một chương trình muốn hiển thị được hộp thoại (đối với chương trình chạy ở Window) thì đầu tiên phải handle được thư viện, ví dụ như kernel32, hay comdlg32,... Tiếp đến sau khi handle được những thư viện đó thì nó mới xử lý tiếp được việc xuất hiện hộp thoại hay tiếng beep,...

Sơ đồ tổng quan một tiến trình được gọi:

![Screenshot 2025-05-28 002354](https://github.com/user-attachments/assets/e8ab4a95-e7d6-45a6-ab0c-dd35ca455aac)

## Đinh nghĩa

Có thể tham khảo [ở đây.](https://learn.microsoft.com/en-us/windows/win32/procthread/processes-and-threads)

Về cơ bản, một ứng dụng được tạo ra thì sẽ bao gồm một hay nhiều process (tiến trình). Có thể ví dụ một đoạn code C sau:

```cpp
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid;
    /* fork a child process */
    pid = fork();

    if (pid < 0) { /* error occurred */
        fprintf(stderr, "Fork Failed");
        return 1;
    }
    else if (pid == 0) { /* child  */
        printf("I am the child %d\n",pid);
        execlp("/bin/ls","ls",NULL);
    }
    else { /* father  */
        printf("I am the parent %d\n",pid);
        wait(NULL);
        printf("Child Complete");
    }

    return 0;
}
```
Đoạn code trên tuy là một đoạn code C chạy trên ubuntu nhưng có thể hình dung được đó đang sinh ra 2 process. Thì trên Window cũng tương tự như thế.

## _EPROCESS & Process Environment Block (PEB)

Tài liệu sẽ được tham khảo [ở đây.](https://eforensicsmag.com/windows-process-internals-a-few-concepts-to-know-before-jumping-on-memory-forensics-by-kirtar-oza/)

Đơn giản như là mỗi process sẽ được thể hiện bởi một cấu trúc có tên là _EPROCESS và _EPROCESS này sẽ chứa những thuộc tính liên quan đến process.

Đây là link sẽ chứa đầy đủ những trường có trong _EPROCESS: https://www.vergiliusproject.com/kernels/x64/Windows%2011/21H2%20(RTM)/_EPROCESS

Những thứ khác ta sẽ không đề cập đến ta chỉ chú ý đến trường **PEB** nằm ở offset 0x550

![image](https://github.com/user-attachments/assets/b307b709-e730-49ff-9139-9d67e4573986)

Tương tự như _EPROCESS  thì PEB cũng là một struct và đây là những trường mà PEB chứa:

![image](https://github.com/user-attachments/assets/103581a3-bc63-4131-80f6-9ce348d57678)

Có thể tham khảo thêm:

https://www.vergiliusproject.com/kernels/x64/Windows%2011/21H2%20(RTM)/_PEB

https://securitycafe.ro/2015/12/14/introduction-to-windows-shellcode-development-part-2/


## LDR

Tài liệu tham khảo [tại đây.](https://imphash.medium.com/windows-process-internals-a-few-concepts-to-know-before-jumping-on-memory-forensics-part-2-4f45022fb1f8)

Như trong tài liệu tham khảo ở PEB đã nói, trong PEB có rất nhiều trường nhưng bài này chỉ đề cập đến **LDR**. Ta sẽ có được sơ đồ như sau:

![image](https://github.com/user-attachments/assets/f9cd78f4-7e7d-4628-81a7-819d477e2935)

![image](https://github.com/user-attachments/assets/7ab01827-7f7b-402e-a4f5-45966408ce6c)


Ta sẽ chỉ chú ý đến 3 danh sách liên kết đôi bao gồm:

- InLoadOrderModuleList
- InMemoryOrderModuleList
- InInitializationOrderModuleList

Chúng đại diện cho mọi loại module như lib, process được tải vào ngay tại thời điểm đó

### _LDR_DATA_TABLE_ENTRY

Ta sẽ có thêm một định nghĩa về _LDR_DATA_TABLE_ENTRY, có thể tham khảo thêm [tại đây.](https://learn.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-peb_ldr_data)

Thì với mỗi trường trong liên kết đôi này lại chứa thêm những điểm entry có thể tham khảo [ở đây.](https://www.vergiliusproject.com/kernels/x64/Windows%2011/22H2%20(2022%20Update)/_LDR_DATA_TABLE_ENTRY)

![image](https://github.com/user-attachments/assets/93f65ef4-b744-42e1-b8e9-acb667c3fa3e)

Ở đây ta sẽ chú ý đến một việc, đó chính là mỗi khi muốn chuyển đổi trường, ví dụ ta đang ở `InMemoryOrderLinks` nhưng ta muốn đến `BaseDllName` thì ta phải trừ cho offset về đầu tiên rồi mới cộng lại địa chỉ 0x58. Tương tự cho các phần khác muốn di chuyển đến trường khác cũng phải trừ để cho offset về đầu rồi mới bắt đầu tính toán.

## Find function in DLL

Tài liệu tham khảo [tại đây.](https://www.ired.team/offensive-security/code-injection-process-injection/finding-kernel32-base-and-function-addresses-in-shellcode#finding-kernel32-address-in-assembly)

Tức là sau khi tìm được module, thì chương trình sẽ lặp qua để tìm hàm thực thi thích hợp bằng cách tính toán RVA cũng như table. Các offset cũng đã được quy định sẵn ở tài liệu tham khảo

=> Qua đó ta sẽ kết luận nhỏ rằng muốn thực thi một tiến trình nào đó thì ta phải handle được thư viện chứa process đó rồi sau đó mới thực thi. Việc handle và thực thi gồm những gì và ra sao thì ở trên đã nói toàn bộ.

## Example 

Ở đây mình sẽ lấy luôn bài ở flare10

![image](https://github.com/user-attachments/assets/1f8b98a1-e5cc-412b-b58f-de8de4c73cd2)

Thì đây chính là đoạn handle được kernel nơi chứa function "beep". Tất cả đã được chú thích rõ ràng và sau khi handle được thì module sẽ được lưu ở `RAX`

![image](https://github.com/user-attachments/assets/4d38e65e-9766-4304-80f7-8b562c556928)


Sau đó thì chương trình sẽ tiến hành lặp để tìm hàm "Beep" trong module 

![image](https://github.com/user-attachments/assets/26195fe2-283a-47f8-8690-4c1e996abd88)

![image](https://github.com/user-attachments/assets/92dd48fc-dd2a-4730-b3fd-ab29ac270f75)

Ở đoạn code trên có đoạn chương trình chuyển `RVA` về `R9` để tính toán chứ không còn để ở `RAX` như lúc đầu load signature vào.
