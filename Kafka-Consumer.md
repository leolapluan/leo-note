# 🔑 **5.1 Consumer trong Kafka System**
![image.png](https://images.viblo.asia/f0e4d8f4-2e36-4965-a4df-20b1bd5f2728.png)
### 🔍 **Consumer làm gì?**

Trong Kafka, **client (consumer)** đóng vai trò quan trọng bằng cách:  
- **Đọc dữ liệu từ các topic**: Lấy thông điệp từ log phân tán của Kafka.  
- **Cung cấp dữ liệu cho ứng dụng**: Như bảng điều khiển (metrics dashboards) hoặc các công cụ phân tích.  
- **Lưu trữ dữ liệu vào hệ thống khác**: Đảm bảo truy cập lâu dài hoặc xử lý thêm.

### ⏱ **Kiểm soát tốc độ tiêu thụ**

Consumer trong Kafka có lợi thế đặc biệt: **kiểm soát tốc độ tiêu thụ dữ liệu**. Điều này cho phép:  
- Consumer quyết định **lượng dữ liệu cần lấy** và **thời điểm lấy dữ liệu**.  
- Ứng dụng được thiết kế để **xử lý tải thay đổi một cách hiệu quả**, tránh bị quá tải. 
## 5.1.1. **Consumer Options**:  
   - Sử dụng các **deserializer** phù hợp cho khóa và giá trị (ví dụ: `StringDeserializer` hoặc `LongDeserializer`).  
   - Đảm bảo các cấu hình như `bootstrap.servers`, `group.id`, và các tham số timeout (`heartbeat.interval.ms`).  

2. **Xử Lý Dữ Liệu Từ Topic**:  
   - Poll dữ liệu từ topic `kinaction_promos`.  
   - Áp dụng công thức xử lý giá trị từ các sự kiện với **magic number** (ví dụ: nhân 1.543).  

---

### 📋 **Bảng Cấu Hình Consumer (Bảng 5.1)**

| **Key**                | **Mục Đích**                                                                 |
|-------------------------|-----------------------------------------------------------------------------|
| **bootstrap.servers**   | Một hoặc nhiều Kafka broker để kết nối khi khởi động client.                |
| **value.deserializer**  | Cần thiết để giải mã (deserialization) giá trị từ topic.                    |
| **key.deserializer**    | Cần thiết để giải mã (deserialization) khóa từ topic.                       |
| **group.id**            | Tên được sử dụng để tham gia một consumer group.                            |
| **client.id**           | ID để xác định một người dùng (chương 10 sẽ sử dụng).                       |
| **heartbeat.interval.ms** | Khoảng thời gian giữa các lần consumer gửi tín hiệu (ping) đến group coordinator. |

---

### 🖥️ **Code: Listing 5.1 - Consumer xử lý khuyến mãi**
```java
“public class KinactionStopConsumer implements Runnable {
     private final KafkaConsumer<String, String> consumer;
     private final AtomicBoolean stopping =
                              new AtomicBoolean(false);
     ...
 
    public KinactionStopConsumer(KafkaConsumer<String, String> consumer) {
      this.consumer = consumer;
    }
 
     public void run() {
         try {
             consumer.subscribe(List.of("kinaction_promos"));
             while (!stopping.get()) {                         ❶
                 ConsumerRecords<String, String> records =
                   consumer.poll(Duration.ofMillis(250));
                 ...
             }
         } catch (WakeupException e) {                         ❷
             if (!stopping.get()) throw e;
         } finally {
             consumer.close();                                 ❸
         }
     }
 
     public void shutdown() {                                  ❹
         stopping.set(true);
         consumer.wakeup();
     }
}
  ❶ The variable stopping determines whether to continue processing.

  ❷ The client shutdown hook triggers WakeupException.

  ❸ Stops the client and informs the broker of the shutdown

  ❹ Calls shutdown from a different thread to stop the client properly
```
## 5.1.2 Hiểu rõ về Offset trong Kafka
![image.png](https://images.viblo.asia/3bfd0df4-1b6b-45a5-b4d8-67f49a4aebb0.png)

### 🔍 **Offset là gì?**

**Offset** là một khái niệm quan trọng trong Kafka, được sử dụng như một **chỉ mục (index)** trong log.  
- Consumer gửi **offset** đến broker để **xác định thông điệp cần đọc** và **vị trí bắt đầu trong log**.  
- Offset giúp consumer biết **nên lấy dữ liệu từ đâu** trong topic.
### ⚙️ **Cấu hình Offset – Ví dụ minh họa**

Trong ví dụ sử dụng **console consumer**, ta dùng cờ `--from-beginning`.  
- Cờ này tương ứng với cấu hình `auto.offset.reset` được đặt thành **earliest**.  
- Khi đó, consumer sẽ đọc **toàn bộ thông điệp trong topic**, bao gồm cả những thông điệp được gửi **trước khi consumer khởi động**

**Figure 5.2** minh họa:  
- Consumer **luôn đọc từ đầu log** mỗi khi chạy với chế độ `--from-beginning`.  
- Điều này cho thấy **cách offset quản lý quá trình đọc dữ liệu** hiệu quả.


