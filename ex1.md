 

## Bước 1: Viết 2 prompt mẫu theo 2 phương án
 
### Phương án 1 — Prompt ngắn gọn, đơn giản

```
Thực hiện 3 nhiệm vụ sau theo thứ tự:
1. Tính tổng các số chẵn từ 1 đến 20.
2. Chia kết quả trên cho 3, làm tròn 2 chữ số.
3. Nếu kết quả > 35 thì in "PASS", ngược lại in "FAIL".
```

### Phương án 2 — Prompt chi tiết, nhiều ràng buộc

```
Bạn là một agent xử lý chuỗi nhiệm vụ (prompt chaining). Thực hiện NHIỆM VỤ A → B → C đúng thứ tự, mỗi nhiệm vụ một mục rõ ràng

RÀNG BUỘC:
1. NHIỆM VỤ A: tính tổng tất cả các số chẵn từ 1 đến 20. Bắt buộc liệt kê dãy số được cộng, sau đó mới ghi tổng không được nhảy cóc kết quả
2. ĐIỂM NGẮT : sau mỗi nhiệm vụ, dừng và in dòng "CHECKPOINT <tên nhiệm vụ>: <kết quả>" để xác nhận trước khi sang nhiệm vụ kế. Nếu dữ liệu truyền giữa các nhiệm vụ bị thiếu hoặc sai định dạng, DỪNG và báo "LỖI ĐỨT GÃY tại <vị trí>" thay vì đoán mò
3. NHIỆM VỤ B: lấy đúng kết quả đã xác nhận của A, chia cho 3, làm tròn đến 2 chữ số thập phân 
4. NHIỆM VỤ C: so sánh kết quả B với ngưỡng 35. Nếu B > 35, in "PASS" kèm mốc thời gian giả lập 10:35; nếu B ≤ 35, in "FAIL". Không được tự bổ sung dữ liệu ngoài đề bài
5. Kiểm tra chéo: trước khi trả lời cuối, đối chiếu lại mỗi kết quả với phép tính một lần nữa
Định dạng đầu ra: A → B → C, mỗi dòng một nhiệm vụ, cuối cùng in "KẾT QUẢ CUỐI: <PASS/FAIL>".
```

---

## Bước 2: Bảng so sánh

| Tiêu chí đánh giá | Phương án 1 (Ngắn gọn) | Phương án 2 (Chi tiết) |
|---|---|---|
| Độ dễ hiểu | Cao — ngắn, đọc là hiểu ngay | Trung bình  |
| Mức tiêu thụ token | Ít (~50 token) | Nhiều hơn  |
| Chặn lỗi biên | Trung bình — không có checkpoint, mô hình dễ nhảy bước hoặc tự đoán khi thiếu dữ liệu | Rất tốt |

**Nhận xét:** Phương án 1 phù hợp tác vụ ngắn, một bước, ưu tiên tốc độ và tiết kiệm token. Phương án 2 đánh đổi token lấy độ tin cậy: với chuỗi nhiều bước như bài toán CoT + Prompt Chaining, các checkpoint và quy tắc xử lý đứt gãy giúp phát hiện lỗi sớm thay vì để sai số lan truyền đến kết quả cuối.

---

## Bước 3: Chọn phương án tối ưu + kết quả chạy thử

### Lựa chọn: **Phương án 2 (Prompt chi tiết)**

**Lý do:**
1. Đề bài yêu cầu chuỗi 3 nhiệm vụ phụ thuộc nhau đúng kịch bản mà prompt chaining dễ đứt gãy nhất; chỉ phương án 2 có cơ chế breakpoint để chặn lỗi biên
2. Chi phí token cao hơn (~250 so với ~50) là đánh đổi chấp nhận được vì độ chính xác của chuỗi dài quan trọng hơn vài trăm token.
3. Khi chạy thử, phương án 2 xuất ra đủ dấu vết từng bước giúp đối chiếu với đáp án chuẩn trong khi phương án 1 dễ bỏ qua việc liệt kê dãy số và không có mốc thời gian ở nhiệm vụ C

### Kết quả chạy thử 
 
 
**NHIỆM VỤ A:** Dãy số chẵn được cộng từ 1 đến 20 là: $2 + 4 + 6 + 8 + 10 + 12 + 14 + 16 + 18 + 20$. Tổng = $110$.
CHECKPOINT A: 110

**NHIỆM VỤ B:** Lấy kết quả xác nhận từ A là $110$, thực hiện phép chia: $\frac{110}{3} \approx 36.666...$, làm tròn đến 2 chữ số thập phân = $36.67$.
CHECKPOINT B: 36.67

**NHIỆM VỤ C:** So sánh kết quả B ($36.67$) với ngưỡng $35$ ($36.67$ lớn hơn $35$): PASS 10:35
CHECKPOINT C: PASS 10:35

KẾT QUẢ CUỐI: PASS
