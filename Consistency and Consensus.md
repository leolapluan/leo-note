# **O. Openning**
Focus on algo and protocol
Chapter 8: packets can be lost, reordered, duplicated, or arbitrarily delayed in the network; clocks are approximate at best; and nodes can pause (e.g., due to garbage collection) or crash at any time.

# **I. Consistency Guarantees**
Replication Lag - timing issues
Read-after-write consistency
<img width="551" alt="image" src="https://github.com/user-attachments/assets/d4a2d7e4-bf33-4803-b769-a278b7541137" />

#### **Nhất quán trong hệ thống phân tán có ý nghĩa gì?**

- Khi một hệ thống được gọi là "nhất quán", điều đó có nghĩa gì về dữ liệu mà nó lưu trữ?
- Sự khác biệt giữa **consistency** (trong CAP theorem) và **consistency** (trong ACID) là gì?
  -> Đồng bộ giữa các nút vs toàn vẹn dữ liệu trước và sau transaction
#### **Các loại nhất quán khác nhau (strong, eventual, causal...) ảnh hưởng đến ứng dụng như thế nào?**

- **Strong consistency**: Nghĩa là tất cả các node nhìn thấy cùng một giá trị tại cùng một thời điểm. Khi nào điều này quan trọng?
- **Eventual consistency**: Nếu không có cập nhật mới, tất cả các bản sao của dữ liệu sẽ hội tụ về cùng một giá trị theo thời gian. Khi nào có thể chấp nhận điều này? (Most replicated db provide at least)
- **Causal consistency**: Một thao tác chỉ có thể được nhìn thấy sau khi tất cả các thao tác trước đó mà nó phụ thuộc vào đã được áp dụng. Điều này giúp ích gì trong hệ thống thực tế?
#### **Khi nào nên chọn nhất quán mạnh (strong consistency) thay vì nhất quán cuối cùng (eventual consistency)?**

- Có tình huống nào mà nhất quán mạnh là **bắt buộc** không? (Ví dụ: hệ thống tài chính, ngân hàng, giao dịch...)
- Ngược lại, có ứng dụng nào có thể **chấp nhận** eventual consistency để cải thiện hiệu suất không?

# **II. Linearizability (One of strongest consistency models)**
<img width="540" alt="image" src="https://github.com/user-attachments/assets/3f5feea8-9ab4-4f7f-8ad6-a7e36bd69e91" />
<img width="551" alt="image" src="https://github.com/user-attachments/assets/af9de813-00ff-4b3d-a18d-790f838ca39e" />
<img width="549" alt="image" src="https://github.com/user-attachments/assets/bbd8be26-cb3e-49da-8c18-6f2dd7567115" />

Serializability

Serializability is an isolation property of transactions, where every transaction
may read and write multiple objects (rows, documents, records)—see “SingleObject and Multi-Object Operations” on page 228. It guarantees that transactions behave the same as if they had executed in some **serial order** (each
transaction running to completion before the next transaction starts). It is okay
for that serial order to be **different from the order** in which transactions were
actually run 

Linearizability

Linearizability is a recency guarantee on reads and writes of a register (an individual object). It doesn’t group operations together into transactions, so it does
not prevent problems such as write skew (see “Write Skew and Phantoms” on
page 246), unless you take additional measures such as materializing conflicts
(see “Materializing conflicts” on page 251).

#### **1. Linearizability khác với Sequential Consistency như thế nào?**

- Sequential consistency đảm bảo rằng tất cả các thao tác xảy ra theo **một thứ tự hợp lý**, nhưng không nhất thiết phải phản ánh thời gian thực. Linearizability có gì khác biệt?
- Tại sao Linearizability kém hơn Sequential Consistency về mặt hiệu năng?

#### **2. Điều kiện nào cần thiết để đảm bảo Linearizability?**

- Khi nào một hệ thống có thể tuyên bố là Linearizable?
- Làm thế nào để kiểm tra xem một hệ thống có thực sự đảm bảo tính Linearizability không?

#### **3. Có những hệ quả gì khi sử dụng Linearizability trong hệ thống thực tế?**

- Linearizability giúp ích gì trong các hệ thống như khóa phân tán (distributed locks) hoặc bộ đếm phân tán?
- Hạn chế của Linearizability là gì? Tại sao không phải hệ thống nào cũng sử dụng nó?
- Khi nào nên chọn Linearizability và khi nào có thể chấp nhận một mô hình nhất quán yếu hơn?

# **III. Ordering Guarantees (Các cam kết về thứ tự)**

### **1. Tại sao thứ tự ghi dữ liệu quan trọng trong hệ thống phân tán?**

- Nếu hai client thực hiện cập nhật cùng một dữ liệu, làm thế nào để hệ thống quyết định thứ tự đúng?
- Điều gì có thể xảy ra nếu một hệ thống **không** duy trì thứ tự ghi dữ liệu?

### **2. Sự khác biệt giữa causal consistency và sequential consistency là gì?**

- **Sequential consistency**: Tất cả các thao tác đều xuất hiện theo một thứ tự nào đó, nhưng không nhất thiết phải phản ánh thời gian thực.
- **Causal consistency**: Nếu một thao tác phụ thuộc vào thao tác trước đó, thì tất cả các node trong hệ thống phải thấy các thao tác đó theo đúng thứ tự nhân quả.
- Khi nào cần dùng **causal consistency** thay vì **sequential consistency**?

### **3. Những tình huống nào yêu cầu duy trì causal consistency thay vì eventual consistency?**

- Trong một ứng dụng chat nhóm, nếu một người gửi tin nhắn "Xin chào", sau đó gửi tin nhắn "Bạn khỏe không?", điều gì xảy ra nếu các tin nhắn này đến tay người nhận theo thứ tự ngược lại?
- Khi nào hệ thống có thể chấp nhận **eventual consistency** mà không cần đảm bảo **causal consistency**?

# **IV. Distributed Transactions and Consensus (Giao dịch phân tán và đồng thuận)**

### **1. Tại sao 2PC (Two-Phase Commit) lại có vấn đề về sẵn sàng (availability)?**

- 2PC hoạt động như thế nào, và tại sao nó được sử dụng trong giao dịch phân tán?
- Điều gì xảy ra nếu coordinator (trình điều phối) bị lỗi trong quá trình thực hiện 2PC?
- Có giải pháp nào để khắc phục hạn chế này không?

---

### **2. Các thuật toán đồng thuận như Paxos và Raft giải quyết vấn đề gì trong hệ thống phân tán?**

- Vì sao đồng thuận (consensus) là cần thiết trong hệ thống có nhiều bản sao dữ liệu (replicated data)?
- Paxos hoạt động như thế nào, và tại sao nó được xem là khó triển khai trong thực tế?
- Raft được thiết kế để dễ hiểu hơn Paxos như thế nào?

---

### **3. Vì sao đồng thuận (consensus) là yếu tố cốt lõi để đạt được nhất quán mạnh trong các hệ thống phân tán?**

- Khi nào hệ thống cần sử dụng consensus protocol như Raft hoặc Paxos?
- Có cách nào đạt được tính nhất quán mà không cần đến consensus không?
- Những hệ thống nào trong thực tế sử dụng consensus? (Ví dụ: Apache Kafka, Google Spanner...)

# **V. Limitations of Consensus (Hạn chế của cơ chế đồng thuận)**

### **1. FLP Impossibility Theorem nói lên điều gì về hệ thống phân tán?**

- Định lý FLP (Fischer-Lynch-Paterson) chứng minh rằng trong một hệ thống phân tán không đồng bộ, không có thuật toán nào có thể đảm bảo cả **tính an toàn (safety)** và **tính sống (liveness)** khi có ít nhất một lỗi (fault).
- Điều này có ý nghĩa gì đối với các hệ thống thực tế?
- Làm thế nào các thuật toán như Paxos và Raft đối phó với giới hạn này?

---

### **2. Trong thực tế, các hệ thống giải quyết hạn chế của đồng thuận như thế nào?**

- Các hệ thống có thể chấp nhận **nhất quán yếu hơn** để đạt được hiệu suất cao hơn không?
- Một số hệ thống sử dụng **leader-based consensus** (như Raft), trong khi các hệ thống khác dùng **leaderless approaches** (như DynamoDB). Khi nào nên chọn mỗi loại?
- Các kiến trúc **CRDTs (Conflict-Free Replicated Data Types)** hoặc **giao dịch không đồng bộ** có thể thay thế consensus trong một số trường hợp không?

---

### **3. Khi nào nên sử dụng consensus protocol như Raft/Paxos, và khi nào có thể tránh sử dụng nó?**

- Đồng thuận có chi phí cao về mặt hiệu suất. Khi nào nên dùng Raft/Paxos thay vì giải pháp đơn giản hơn?
- Có thể tránh đồng thuận bằng cách sử dụng **eventual consistency** hoặc **causal consistency** không?
- Ví dụ thực tế: Hệ thống nào trong thực tế **tránh sử dụng đồng thuận** để tối ưu hiệu năng?
