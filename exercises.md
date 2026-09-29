# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng đánh dấu của đề bài bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Ngô Văn Giáp  Mã học viên: 2A202602644

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Việc app "chết sớm" ngay khi khởi động giúp ta phát hiện ngay lỗi cấu hình (chưa set API key) ở môi trường hiện tại trước khi app nhận traffic thực tế. Nếu để mặc định là "changeme", app vẫn chạy nhưng sẽ bị hổng bảo mật nghiêm trọng vì bất kỳ ai cũng có thể gọi API bằng mật khẩu mặc định "changeme", hoặc hệ thống chạy sai logic mà không có cảnh báo nào cho đến khi hậu quả đã xảy ra.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log: `{"level": "INFO", "message": "Request processed", "user_id": "sv-test", "cost": 0.0015, "tokens": 150}`
> 
> Hai việc làm được: 1) Có thể dùng code/tool để parse JSON và truy vấn tự động dễ dàng (ví dụ: filter theo `user_id` hoặc tính tổng `cost`). 2) Dễ dàng đẩy log vào các hệ thống theo dõi (như Elasticsearch, Datadog) để tạo dashboard thống kê đồ thị hoặc cài đặt hệ thống cảnh báo tự động khi `cost` tăng cao bất thường.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~ 1730 MB  |
| Multi-stage | ~ 310 MB  |

Giải thích: phần dung lượng chênh lệch đó là những công cụ build (build-essential, compiler, gcc), các file cache của pip tải về lúc cài, mã nguồn thừa không cần thiết lúc chạy, và base image hệ điều hành chứa các phần mềm không cần thiết. Multi-stage giúp vứt bỏ tất cả phần thừa thãi đó, chỉ lấy đúng mã nguồn Python và các thư viện đã được build xong chuyển sang stage cuối cùng để chạy.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi sửa code trong `main.py`, các layer cài đặt dependencies (OS dependencies, pip install) sẽ được lấy lại từ cache. Chỉ có layer từ lệnh `COPY . .` trở đi mới phải chạy lại.
> 
> Nếu đặt `COPY . .` lên trước `RUN pip install`, mỗi khi bạn sửa dù chỉ 1 ký tự trong code Python, lệnh COPY sẽ phát hiện sự thay đổi và phá vỡ bộ nhớ cache của mọi lệnh phía sau. Do đó, lệnh `RUN pip install` sẽ bắt buộc phải chạy lại từ đầu tải toàn bộ thư viện về, làm quá trình build mất rất nhiều thời gian vô ích.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi sự kiện: Kẻ tấn công lợi dụng lỗ hổng trong code Python (ví dụ injection) để thực thi lệnh hệ thống bên trong container. Nếu chạy quyền root, kẻ tấn công sẽ có quyền root bên trong container, từ đó có thể tìm cách leo thang đặc quyền (container breakout) để chiếm luôn quyền root trên máy chủ host vật lý.
> 
> Lệnh `USER nonroot` cắt đứt chuỗi đó ở bước đầu tiên: dù chiếm được quyền chạy code, kẻ tấn công vẫn bị giới hạn quyền như một user thường, không thể cài phần mềm, đọc file hệ thống quan trọng hay tìm đường tấn công ra máy host được.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa 20 request trong 2 giây liên tiếp.
> 
> Giải thích: Giả sử người dùng gửi 10 request ở giây thứ 59 (thuộc phút trước) và được cho qua. Ngay khi đồng hồ chuyển sang giây số 00 (phút hiện tại), bộ đếm bị reset ngay lập tức về 0. Lúc này, người dùng lập tức gửi thêm 10 request nữa và vẫn hợp lệ. Tổng cộng trong 2 giây bản lề đó, hệ thống đã phải xử lý dồn dập 20 request, gây ra hiện tượng spike tải đột biến (điểm yếu lớn của Fixed Window so với Sliding Window).

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Hai cơ chế khác nhau ở mục tiêu: Rate limit giới hạn **tốc độ** (tần suất) request trong một khoảng thời gian rất ngắn (chống spam/quá tải server). Cost guard giới hạn **tổng chi phí** tích lũy trong một thời gian dài (chống cạn kiệt ngân sách/tiền bạc).
> 
> - Qua rate nhưng chặn cost: Một user gửi 5 request mỗi ngày liên tục đặn đặn cả tháng. Không có ngày nào vượt mức rate limit, nhưng đến cuối tháng tổng số tiền sử dụng đã quá 10$ -> bị Cost guard chặn.
> - Bị chặn rate nhưng qua cost: Một user vừa tạo tài khoản mới (chi phí = $0), lập tức dùng tool bắn 20 request trong 10 giây. Lập tức bị Rate limit chặn, dù tổng tiền vẫn chưa đáng bao nhiêu.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> 1. Redis mất kết nối.
> 2. K8s/Nền tảng cloud định kỳ gọi liveness probe vào `/health` để xem app còn sống không (lúc này `/health` đang kiêm luôn check Redis).
> 3. Do Redis đang đứt, endpoint `/health` trả về lỗi (ví dụ mã 500 hoặc timeout).
> 4. Nền tảng nghĩ rằng tiến trình app đã bị treo hoàn toàn (dead).
> 5. Nền tảng tự động kill (restart) liên tục toàn bộ 3 container để cố khôi phục, dẫn đến sập toàn bộ dịch vụ (Cascading failure), dù bản thân app vẫn bình thường và có thể đang phục vụ các API không cần tới Redis.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu lưu lịch sử hội thoại vào dict Python trên bộ nhớ cục bộ (RAM) thay vì dùng Redis, con số `history_length` sẽ tăng lên không ổn định và chập chờn.
> 
> Lý do là hệ thống Load Balancer phân bổ request luân phiên qua lại giữa 3 container khác nhau. Nếu request 1 vào container A, lịch sử của A = 1. Khi request 2 rơi vào container B, bộ nhớ của B trống rỗng nên nó lại báo lịch sử = 1 thay vì 2. Mỗi container sẽ lưu một mảnh lịch sử riêng biệt, khiến ứng dụng không còn tính "nhất quán" (stateful). Dùng Redis làm bộ nhớ chung bên ngoài mới giải quyết được vấn đề này.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi gặp phải: "Application failed to bind to port" (App không thể kết nối tới port).
> 
> Cách tìm nguyên nhân: Mở trang xem Runtime Logs của service trên giao diện Railway, thấy uvicorn báo lỗi không thể start server trên port 8000 vì Railway yêu cầu chạy trên port ngẫu nhiên.
> 
> Cách sửa: Mở file `railway.toml` (hoặc cấu hình lệnh start trong dashboard) và sửa lệnh khởi chạy thành `uvicorn app.main:app --host 0.0.0.0 --port $PORT` để uvicorn linh hoạt nhận đúng cổng mà Railway phân công động cho nó.
