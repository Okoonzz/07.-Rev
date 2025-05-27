Hôm nay nhân dịp làm lab **bảo mật web và ứng dụng** về phần hook android sử dụng **frida** nên tiện làm về chủ đề này biết đâu sau này có dùng đỡ bỡ ngỡ =)))

File thực hành: [Tại đây](https://github.com/dineshshetty/Android-InsecureBankv2)

## Emulate Android

- Ở phần này, ta sẽ chọn platform
    - [**Android Studio**](https://developer.android.com/studio?gad_source=1&gclid=CjwKCAiAl4a6BhBqEiwAqvrquv4yLIZzxBTQVwhKm_owzSzuyeLwJ4ul0FDKrFmicLEYJGiFpHe2TxoCngAQAvD_BwE&gclsrc=aw.ds) khi dùng Android Studio sẽ cung cấp sẵn luôn các tool như **adb, apksigner, sqlite** chỉ cần bỏ vào path env là có thể sử dụng được bình thường.
        - Để tìm được các path của các tool cung cấp sẵn từ android sutdio chỉ cần tìm `<your path>\Android\android-sdk`. Ở đây sẽ có 2 folder bao gồm `build-tools` và `platform-tools` chỉ cần bỏ 2 path này vào env là chạy bình thường.
    - [**Genymotion**](https://www.genymotion.com/product-desktop/download/) đối với tool này khi cài đặt sẽ chọn gói cài đặt chung với VirtualBox (nếu chưa có VirtuallBox). Và đặc biệt khi chạy emulate với genymotion nên **mở virtuallBox chạy android đã cài trước, sau đó mới mở genymotion lên và start android**. Làm điều này sẽ thao tác đỡ giật lag hơn so với việc chỉ mở bằng mỗi genymotion. Một điểm hạn chế của genymotion đó là không có sẵn các tool như **adb, apksigner, sqlite** nên phải tải từng tool riêng nếu muốn cần sử dụng.
        - Để sử dụng được các tool tìm lần lượt các tool đi kèm với github là tải thành công.

- Sau khi cài xong emulate android vào **settings -> About -> nhấn vào build đến khi nào mở chế độ developer -> tìm kiếm developer options -> mở USB debugging và Wireless debugging**

![image](https://hackmd.io/_uploads/BJNU2Eem1g.png)


## Root Android

- Muốn cài được frida và hook vào thì việc cần làm sau khi emulate android đó là phải root được android đó. Nếu thiết bị không được root thì sẽ không thể chạy được frida server.

- Đối với **Android Studio** để root được android ta sẽ dùng gitlab [rootAVD](https://gitlab.com/newbit/rootAVD). Một điều lưu ý đó là tool này chỉ tìm system images tại `%LOCALAPPDATA%\Android\Sdk`, nên **khi cài đặt system images từ Android Studio cần phải lưu ý đặt đúng path**. Để nhận diện được đúng folder hay không thì trong đó phải chứa như sau:

![image](https://hackmd.io/_uploads/S1CAjdJmye.png)

- Thường thì khi cài với kiểu custom thì nên lưu ý điều này.

- Đối với **Genymotion** thì ưu tiên việc chọn những image có quyền root. Tham khảo những image có quyền root [tại đây](https://support.genymotion.com/hc/en-us/articles/360003125397-Is-it-possible-to-un-root-or-hide-root).

## Use ADB

- Một số command adb phổ biến và thường xuyên sử dụng:
    - `adb shell`: Truy cập shell của andoird.
    - `adb devices`: Check thiết bị nào đang được emulate.
    - `adb install <filename>.apk`: Cài file apk vào android.
    - `adb pull <filename>`: Kéo file về máy thật.
    - `adb push <path máy thật><path máy ảo>`: Tải lên từ máy thật lên andoird.
    ...
    
## Install Frida and Frida-Server

- Cài Frida trên máy thật là Frida client: `pip3 install frida-tools`
- Trên máy android truy cập link [github](https://github.com/frida/frida) vào phần release. Tại đây thấy rất nhiều frida, nhưng chỉ tìm **frida-server-<phiên bản>-android-x86 (cho 32bit) x86_64(cho 64 bit)**. Nếu dùng Android Studio thì thường chọn những bản android x86_64 sau đó thì dùng tool để root nên sẽ thường chọn bản 64bit. Còn đối với genymotion thường sẽ chọn image android có sẵn root, mà những android có sẵn root thì thường sẽ là những bản android 32bit.
- Sau khi tải xong từ github tiến hành giải nén file và rename file lại thành `frida-server`
- Mở terminal và nhập lệnh `adb push frida-server /data/local/tmp` để cài frida-server vào android.
- Truy cập vào shell android và `cd` vào `/data/local/tmp` (tất cả đều phải thực hiện root) kiểm tra xem đã được cài thành công hay chưa

![image](https://hackmd.io/_uploads/Skh_TElmkx.png)

- Tiếp đến cấp quyền cho frida-server `chmod +x frida-server`
- Chạy frida-server `./frida-server &`
- Ở máy thật mở một terminal mới kiểm tra xem đã thực sự chạy frida-server hay chưa `frida-ps -U`

![image](https://hackmd.io/_uploads/ryezCVgmye.png)

![image](https://hackmd.io/_uploads/SkufCVe7yx.png)

- Thành công cài đặt frida-server lên android.
- **Lưu ý:** Nếu như bản image 32 nhưng cài frida 64 thì khi chạy server sẽ không được và thông báo lỗi. Còn nếu như để shell bình thường không root thì chạy sẽ hiển thị permission denied.

## Hook Android

- Vì lười chỉnh SDK về đúng phiên bản nên phần hook mình sẽ dùng genymotion.
- Mình sẽ hook thẳng hàm `aesDeccryptedString` 
![image](https://hackmd.io/_uploads/B1IpEBg7Jg.png)
- Sau quá trình làm bài thì mình nhận ra được rằng có vài cách mình tìm hiểu được để hook vào:
    - Đầu tiên là hook không cần tương tác với UI tức là sẽ tạo một instance mới và chạy song song để hook vào không cần tương tác UI để gọi tới hàm đó.
    - Tiếp đến là hook cần đợi tương tác với UI tức là phải đợi tương tác với UI và gọi hàm đó sau đó thì mới hook vào được. Cách này thì không cần tạo mới instance.
- Cách nào trong hai cách trên mình cũng dùng `java.use()` nhiều hơn là `java.choose()`. Còn chúng khác gì nhau thì phần này mình không đào sâu về chúng :3 

```java
import frida
import time

device = frida.get_usb_device()
pid = device.spawn("com.android.insecurebankv2")
device.resume(pid)

time.sleep(1) 

session=device.attach(pid)

hook_script = """
Java.perform(function () {
   var CryptoClass = Java.use("com.android.insecurebankv2.CryptoClass");
   
   var cryptoInstance = CryptoClass.$new();
   try {
       var decrypted = cryptoInstance.aesDeccryptedString("v/sJpihDCo2ckDmLW5Uwiw==");
       console.log("Decrypted value: " + decrypted);
   } catch(e) {
       console.log("Error: " + e);
   }
});
"""
script=session.create_script(hook_script)
script.load()

input('...?')

// output: Decrypted value: Jack@123$
//         ...?
```
- Có thể thấy được rằng hook thẳng vào mà chưa cần nhập username cũng như password
![image](https://hackmd.io/_uploads/ryWFwHemkl.png)
- Tiếp theo hook vào `doesSuperuserApkExist` , đối với hàm này cần phải tương tác UI nên sẽ không tạo instance mới, thay vào đó phải nhập username và password thì mới hook được.
```java
import frida
import time

device = frida.get_usb_device()
pid = device.spawn("com.android.insecurebankv2")
device.resume(pid)

time.sleep(1) 

session=device.attach(pid)

hook_script = """
Java.perform(function () {
    // Tạo một runtime environment để thực thi code Java
    
    var PostLogin = Java.use("com.android.insecurebankv2.PostLogin");
    // Lấy reference tới class PostLogin để có thể hook vào
    
    PostLogin.doesSuperuserApkExist.implementation = function (s) {
        // Hook vào method doesSuperuserApkExist
        // s: tham số được truyền vào từ app gốc
        // Signature ở đây: function (s)
        // - Tên hàm: doesSuperuserApkExist
        // - Tham số: (s) phải khớp với hàm gốc
        
        var ret = this.doesSuperuserApkExist(s);
        // Gọi hàm gốc với tham số s
        // this: đối tượng PostLogin hiện tại
        // Lưu kết quả vào biến ret
        
        console.log("[+] doesSuperuserApkExist called");
        // In ra log khi hàm được gọi
        
        console.log("\tPath: " + s);
        // In ra đường dẫn được truyền vào
        
        console.log("\tExists: " + ret);
        // In ra kết quả từ hàm gốc
        
        return ret;
        // Trả về kết quả như hàm gốc
    };
});
"""
script=session.create_script(hook_script)
script.load()

input('...?')
    
//output: doesSuperuserApkExist called
//       Path: /system/app/Superuser.apk
//        Exists: false
```
![image](https://hackmd.io/_uploads/BygjOBlQkx.png)

```
Luồng thực thi:

1. App gọi hàm doesSuperuserApkExist
2. Frida chặn lại (hook)
3. Hook gọi hàm gốc và lưu kết quả
4. In ra các thông tin debug
5. Trả về kết quả cho app
```

## Patch file

- Để chỉnh sửa cụ thể file apk sử dụng [**apktool**](https://apktool.org/docs/install) với command `apktool.jar d <file apk>`
- Sau khi decompile và chỉnh sửa thường với `smali` ta cần phải build lại file và ký để có thể sử dụng.
- Để build lại file `apktook.jar b <old folder decompile> -o <new file name>.apk`
- Để ký được file vừa build ta dùng `keytool` nếu như không tìm thấy thì có thể tìm ở path liên quan tới `java` hoặc `jdk` vào `bin` thì sẽ thấy. Sau khi kiểm tra đã có thể sử dụng được `keytool` ta dùng lệnh `keytool -genkeypair -v -keystore <file name output>.keystore -alias <file name output> -keyalg RSA -keysize 2048 -validity 10000 `
- Sau khi gen xong key tiến hành ký bằng **apksigner** `apksigner sign --ks <name file key gen>.keystore <name file build>.apk`
