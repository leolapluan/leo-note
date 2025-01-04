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
## 🔢 **Offset và cách hoạt động**  
- **Offset** là chỉ mục xác định vị trí thông điệp trong log.  
- Offset **luôn tăng dần** và không tái sử dụng.  
- Mỗi partition có chuỗi offset riêng, giảm nguy cơ vượt giới hạn kiểu dữ liệu.

---
## 📌 **auto.offset.reset và cách cấu hình**  
- **Mặc định**: `auto.offset.reset = latest`. Chỉ nhận các thông điệp mới sau khi consumer khởi động.  
- **Chế độ đọc từ đầu**: Dùng flag `--from-beginning` để thiết lập `auto.offset.reset = earliest`, cho phép đọc toàn bộ dữ liệu, kể cả thông điệp cũ.

---

## 🖼 **Phân bổ partition và leader**  
- Mỗi topic được chia thành **nhiều partition**, mỗi partition có một leader replica.  
- **Consumer chỉ đọc từ leader replica** của partition.
- <img width="660" alt="image" src="https://github.com/user-attachments/assets/b8249149-1796-4c4b-b739-80d1e6670a4e" />

- Hình 5.3 minh họa:  
  - Partition 1, 2, 3 có leader trên các broker khác nhau.  
  - Các bản sao (replica) được lưu trữ trên các broker phụ nhưng không được consumer đọc trực tiếp.

---

## 🌍 **Ảnh hưởng của số lượng partition**  
- **Nhiều partition** tăng khả năng xử lý song song nhưng đi kèm chi phí:  
  - **Tăng độ trễ** khi đồng bộ giữa các broker.  
  - **Tốn tài nguyên bộ nhớ** nếu consumer phải xử lý nhiều partition.  
- **Khuyến nghị**: Lựa chọn số lượng partition phù hợp với luồng dữ liệu và yêu cầu ứng dụng.

---

## 📊 **Phân bổ consumer và partition**  
- **Số lượng consumer không nên vượt quá số partition.**  
- Hình 5.4 minh họa: Với 4 consumer và 3 partition, consumer dư thừa sẽ ở trạng thái chờ mà không xử lý dữ liệu.
  <img width="657" alt="image" src="https://github.com/user-attachments/assets/6dbd4774-d5c8-4202-92bb-a64c29b84234" />


---

## 📚 **Khả năng tương thích với Apache ZooKeeper**  
- Kafka hiện không sử dụng **ZooKeeper** cho consumer.  
- Trước đây, ZooKeeper được dùng để lưu trữ offset, nhưng hiện nay Kafka client đã loại bỏ phụ thuộc này.


# 5.2 How consumers interact 
**Consumer Group** là một nhóm gồm một hoặc nhiều consumer làm việc cùng nhau để đọc dữ liệu từ một topic trong Kafka
**Cùng group**
Các consumer phối hợp làm việc như một hệ thống duy nhất
**Khác group**
Các group làm việc độc lập, phù hợp với các logic xử lý khác nhau
## Các đặc điểm chính của Consumer Groups:
**Phối hợp (Coordination)**: Các consumer trong cùng một nhóm phối hợp để đảm bảo mỗi partition của topic chỉ được xử lý bởi một consumer duy nhất trong nhóm.
**Chia sẻ Offset**: Các consumer trong cùng một nhóm chia sẻ thông tin offset (vị trí đọc dữ liệu cuối cùng) để đảm bảo không có dữ liệu bị đọc lặp lại hoặc bỏ sót.

Ví dụ:
Giả sử bạn có một topic tên là hr-data với 3 partitions:

Topic: hr-data
```
Partition 0: [message1, message2, message3]
Partition 1: [message4, message5]
Partition 2: [message6, message7, message8]
```

Nếu bạn có 3 consumer trong cùng một nhóm, mỗi consumer sẽ được Kafka phân công đọc một partition.
Nếu bạn thêm hoặc bớt consumer trong nhóm, Kafka sẽ tự động phân phối lại các partition để đảm bảo mỗi partition vẫn được xử lý (rebalancing)

# 5.3 Tracking
Ở các hệ thống khác, VD như RabbitMQ, nó sẽ xoá record ngay sau khi consumer ack message. Tuy nhiên đối với Kafka, những record này sẽ được lưu lại
Sự khác biệt quan trọng giữa Kafka với các hệ thống khác, ở việc cách consumer có được message. Kafka consumer sẽ poll dữ liệu từ Kafka về, trong khi đó các hệ thống khác lại thực hiện việc push message tới consumer cần tiêu thụ. Quan sát hình 5.5 phía dưới mô tả vấn đề khi push message tới consumer
<img width="653" alt="image" src="https://github.com/user-attachments/assets/f28da6a0-4b6a-408b-915d-91e3d4e6c25e" />
Vấn đề 1: Do message bị xoá sau khi ack, nên chỉ có thể tiếp tục consume từ message 3 (Có 1 vấn đề Leo vẫn thắc mắc, mình sẽ để ngỏ ở đây, tại sao RabbitMQ không thực hiện việc lưu trữ lại message giống như Kafka làm??? mình sẽ research và bổ sung dưới comment sau :>>)
Vấn đề 2: Nếu có nhiều consumer cần tiêu thụ cùng 1 message, message đó sẽ bị duplicate trong nhiều queue khác nhau

2 vấn đề trên đều được Kafka giải quyết, nó sẽ chủ động chìa message và consumer nào cần sẽ tự chủ động đi poll dữ liệu về.

“it is important that the **offsets** and **partitions** are **specific** to a certain consumer group”

## 5.3.1 Group coordinator
Group coordinator làm việc với các consumer client để giữ thông tin về vị trí mà một nhóm cụ thể đã đọc trong topic
<img width="500" alt="image" src="https://github.com/user-attachments/assets/dec06095-f959-4d51-bd33-7c624a8e992e" />
Hình 5.7 minh họa một scenario mà các partition giống nhau tồn tại trên ba broker khác nhau cho hai consumer group khác nhau, là kinaction_teamoffka0 và kinaction_teamsetka1. Các consumer trong mỗi group sẽ nhận một bản sao dữ liệu riêng từ các partition trên mỗi broker. Chúng không làm việc cùng nhau trừ khi thuộc cùng một group
<img width="664" alt="image" src="https://github.com/user-attachments/assets/0b12a4f2-e49b-4509-a16b-f04b94ff3274" />

1 quy tắc cần lưu ý là chỉ 1 consumer của 1 group có thể đọc 1 partition tại 1 thời điểm (mặc dù 1 partition có thể được đọc bởi nhiều consumer)

