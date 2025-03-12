# **O. Openning**
Focus on algo and protocol
Chapter 8: packets can be lost, reordered, duplicated, or arbitrarily delayed in the network; clocks are approximate at best; and nodes can pause (e.g., due to garbage collection) or crash at any time.

# **I. Consistency Guarantees**
Replication Lag - timing issues
Read-after-write consistency
<img width="551" alt="image" src="https://github.com/user-attachments/assets/d4a2d7e4-bf33-4803-b769-a278b7541137" />

## **Nhất quán trong hệ thống phân tán có ý nghĩa gì?**

- Khi một hệ thống được gọi là "nhất quán", điều đó có nghĩa gì về dữ liệu mà nó lưu trữ?
- Sự khác biệt giữa **consistency** (trong CAP theorem) và **consistency** (trong ACID) là gì?
  -> Đồng bộ giữa các nút vs toàn vẹn dữ liệu trước và sau transaction
## **Các loại nhất quán khác nhau (strong, eventual, causal...) ảnh hưởng đến ứng dụng như thế nào?**

- **Strong consistency**: Nghĩa là tất cả các node nhìn thấy cùng một giá trị tại cùng một thời điểm. Khi nào điều này quan trọng?
- **Eventual consistency**: Nếu không có cập nhật mới, tất cả các bản sao của dữ liệu sẽ hội tụ về cùng một giá trị theo thời gian. Khi nào có thể chấp nhận điều này? (Most replicated db provide at least)
- **Causal consistency**: Một thao tác chỉ có thể được nhìn thấy sau khi tất cả các thao tác trước đó mà nó phụ thuộc vào đã được áp dụng. Điều này giúp ích gì trong hệ thống thực tế?
## **Khi nào nên chọn nhất quán mạnh (strong consistency) thay vì nhất quán cuối cùng (eventual consistency)?**

- Có tình huống nào mà nhất quán mạnh là **bắt buộc** không? (Ví dụ: hệ thống tài chính, ngân hàng, giao dịch...)
- Ngược lại, có ứng dụng nào có thể **chấp nhận** eventual consistency để cải thiện hiệu suất không?

# **II. Linearizability (One of strongest consistency models)**
linearizability essentially means “behave as though there is only a single copy of
the data, and all operations on it are atomic” 
<img width="540" alt="image" src="https://github.com/user-attachments/assets/3f5feea8-9ab4-4f7f-8ad6-a7e36bd69e91" />
<img width="551" alt="image" src="https://github.com/user-attachments/assets/af9de813-00ff-4b3d-a18d-790f838ca39e" />
<img width="549" alt="image" src="https://github.com/user-attachments/assets/bbd8be26-cb3e-49da-8c18-6f2dd7567115" />
## **1. Linearizability khác với Serializability như thế nào?**
Linearizability is easily confused with serializability (see “Serializability” on page 251),
as both words seem to mean something like “can be arranged in a sequential order.”

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

## **2. Dựa vào Linearizability?**

- Long and Leader Election
- Constraints and uniqueness guarantees
- Cross-channel timing dependencies
  <img width="547" alt="image" src="https://github.com/user-attachments/assets/5e2601c1-3cde-4278-a849-d13f7548f133" />

## **3. Implement Linearizable Systems**
Since linearizability essentially means “behave as though there is only a single copy of
the data, and all operations on it are atomic,” the simplest answer would be to really
only use a single copy of the data. However, that approach would not be able to toler‐
ate faults: if the node holding that one copy failed, the data would be lost, or at least
inaccessible until the node was brought up again.

The **most common** approach to making a system fault-tolerant is to use **replication**.
Let’s revisit the replication methods from Chapter 5, and compare whether they can
be made linearizable
**Single-leader replication (potentially linearizable)**

**Consensus algo (linearizable)**
**Multileader replication (not linearizable)**
Write conflict -> lack of single copy data
**Leaderless replication (probably not linearizable)**
For systems with leaderless replication (Dynamo-style; see “Leaderless Replica‐
tion” on page 177), people sometimes claim that you can obtain “strong consis‐
tency” by requiring quorum reads and writes (w + r > n). Depending on the exact configuration of the quorums, and depending on how you define strong consistency, this is not quite true.

“Last write wins” conflict resolution methods based on time-of-day clocks (e.g.,
in Cassandra; see “Relying on Synchronized Clocks” on page 291) are almost cer‐
tainly nonlinearizable, because clock timestamps cannot be guaranteed to be
consistent with actual event ordering due to clock skew.

Sloppy Quorums là một kỹ thuật trong cơ sở dữ liệu phân tán (như Cassandra, DynamoDB) cho phép ghi dữ liệu vào bất kỳ nút khả dụng nào thay vì chờ các nút cụ thể trong nhóm chính (quorum). Điều này giúp hệ thống duy trì hoạt động ngay cả khi có lỗi mạng nhưng có thể dẫn đến dữ liệu bị trễ và không nhất quán, làm mất tính tuyến tính hóa. Ngay cả khi sử dụng Strict Quorums, nơi yêu cầu một số lượng nút đồng ý trước khi ghi dữ liệu, tuyến tính hóa vẫn có thể bị phá vỡ do độ trễ mạng hoặc độ lệch đồng hồ, dẫn đến việc một nút có thể đọc dữ liệu cũ trong khi nút khác đã ghi dữ liệu mới.

### Linearizability and quorums
Strict quorum: w + r > n
Mặc dù là strict quorum nhưng do network delay nên vẫn non-linearizable
<img width="537" alt="image" src="https://github.com/user-attachments/assets/cc788b37-4501-41ff-8eae-d7b0107e55e1" />

## **4. Cost of Linearizability?**
multi-leader replication is often a good choice for multidatacenter replication
<img width="557" alt="image" src="https://github.com/user-attachments/assets/03759492-5fbd-4ba3-aeec-c3a63f57deaa" />
With a multi-leader database, each datacenter can continue operating normally: since
writes from one datacenter are asynchronously replicated to the other, the writes are
simply queued up and exchanged when network connectivity is restored.

On the other hand, if **single-leader replication is used, then the leader must be in one
of the datacenters**. Any writes and any linearizable reads must be sent to the leader—
thus, for any clients connected to a follower datacenter, those read and write requests
must be sent synchronously over the network to the leader datacenter.

If the network between datacenters is interrupted in a single-leader setup, **clients con‐
nected to follower datacenters cannot contact the leader, so they cannot make any
writes to the database, nor any linearizable reads**. They can still make reads from the
follower, but they might be stale (nonlinearizable). **If the application requires linear‐
izable reads and writes**, the network interruption causes the **application** to become
unavailable in the datacenters that **cannot contact the leader**.
### The CAP theorem


- Linearizability giúp ích gì trong các hệ thống như khóa phân tán (distributed locks) hoặc bộ đếm phân tán?
- Hạn chế của Linearizability là gì? Tại sao không phải hệ thống nào cũng sử dụng nó?
- Khi nào nên chọn Linearizability và khi nào có thể chấp nhận một mô hình nhất quán yếu hơn?

# **III. Ordering Guarantees (Các cam kết về thứ tự)**

## **1. Tại sao thứ tự ghi dữ liệu quan trọng trong hệ thống phân tán?**

- Nếu hai client thực hiện cập nhật cùng một dữ liệu, làm thế nào để hệ thống quyết định thứ tự đúng?
- Điều gì có thể xảy ra nếu một hệ thống **không** duy trì thứ tự ghi dữ liệu?

## **2. Sự khác biệt giữa causal consistency và sequential consistency là gì?**

- **Sequential consistency**: Tất cả các thao tác đều xuất hiện theo một thứ tự nào đó, nhưng không nhất thiết phải phản ánh thời gian thực.
- **Causal consistency**: Nếu một thao tác phụ thuộc vào thao tác trước đó, thì tất cả các node trong hệ thống phải thấy các thao tác đó theo đúng thứ tự nhân quả.
- Khi nào cần dùng **causal consistency** thay vì **sequential consistency**?

## **3. Những tình huống nào yêu cầu duy trì causal consistency thay vì eventual consistency?**

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
