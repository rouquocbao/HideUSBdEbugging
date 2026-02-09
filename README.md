# HideUSBdEbugging
Bước 1: Mở file mục tiêu
Tìm đến file: frameworks/base/core/java/android/provider/Settings.java
Bước 2: Tìm hàm xử lý đọc giá trị Global
Trong file này, bạn tìm lớp tĩnh public static final class Global. Bên trong lớp này, tìm hàm chịu trách nhiệm lấy giá trị số nguyên (vì adb_enabled thường được truy vấn qua getInt).

Hàm bạn cần tìm là: public static int getInt(ContentResolver cr, String name, int def)
Bước 3: Chèn logic "Tàng hình"
Bạn sẽ sửa logic để kiểm tra xem ứng dụng đang gọi là ai. Nếu tên biến là adb_enabled và ứng dụng gọi không phải là ứng dụng tin cậy, hãy trả về 0.
```
// Tìm hàm getInt trong class Global
public static int getInt(ContentResolver cr, String name, int def) {
    String v = getString(cr, name);
    try {
        // --- ĐOẠN CODE CHÈN THÊM ---
        if (v != null && ("adb_enabled".equals(name) || "adb_wifi_enabled".equals(name))) {
            // Lấy UID của tiến trình đang gọi
            int callingUid = android.os.Binder.getCallingUid();
            
            // UID của System (1000) và Root (0) hoặc Shell (2000) thì cho phép xem sự thật
            if (callingUid > 2000) { 
                // Trả về giá trị mặc định (thường là 0) để đánh lừa app bên thứ 3
                return 0; 
            }
        }
        // --- KẾT THÚC ĐOẠN CHÈN ---
        
        return v != null ? Integer.parseInt(v) : def;
    } catch (NumberFormatException e) {
        return def;
    }
}
```
