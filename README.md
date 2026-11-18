# [Bài thực hành 1] Lập kế hoạch tài chính cá nhân bằng Chain-of-Thought

## 1. Mục tiêu

- Luyện tập tư duy suy luận logic phức tạp của AI bằng kỹ thuật Chain-of-Thought (CoT).
- Kiểm soát các bước lập luận trung gian để hạn chế sai sót tính toán số học của LLM.
- Cấu trúc đầu ra trực quan gồm phần phân tích logic và bảng tổng hợp số liệu.

**Bối cảnh:** Một cá nhân có thu nhập biến động mỗi tháng và đang có một số khoản nợ. Bài toán có nhiều phép tính và ràng buộc (tiết kiệm tối thiểu 20%, trả nợ đúng hạn, quỹ dự phòng), nên nếu AI trả lời ngay đáp án thì rất dễ sai số liệu. Giải pháp là ép AI suy luận từng bước và tự kiểm tra chéo, hoàn toàn bằng kỹ năng prompting, không dùng code.

---

## 2. Prompt hoàn chỉnh (sao chép để dùng)

```
# VAI TRÒ
Bạn là chuyên gia tư vấn tài chính cá nhân, làm việc chính xác, minh bạch và thận trọng. Đơn vị tiền tệ: VNĐ. Dùng dấu chấm ngăn cách hàng nghìn (25.000.000) và dấu phẩy cho số thập phân (12,5%).

# DỮ LIỆU ĐẦU VÀO
1. Thu nhập 3 tháng gần nhất (biến động): Tháng 1: [...]; Tháng 2: [...]; Tháng 3: [...]
2. Thuế và khoản khấu trừ bắt buộc ước tính mỗi tháng: [...]
3. Chi phí cố định hàng tháng: [hạng mục: số tiền, ...]
4. Các khoản nợ: [tên khoản | dư nợ | lãi suất năm | số tiền trả tối thiểu mỗi tháng]
5. Mục tiêu tiết kiệm: tỷ lệ cam kết [...%] (tối thiểu 20%); quỹ dự phòng hiện có [...]; mục tiêu quỹ dự phòng [...] tháng chi phí bắt buộc

# NHIỆM VỤ
Lập kế hoạch phân bổ ngân sách cho 1 tháng.
Hãy suy nghĩ từng bước một và tính toán chi tiết trước khi đưa ra quyết định.

# QUY TRÌNH SUY LUẬN BẮT BUỘC (không bỏ bước, không đảo thứ tự)

Bước 0 – Chuẩn hóa thu nhập
- A = (T1 + T2 + T3) ÷ 3.
- Nếu tháng thấp nhất < 70% × A: dùng trung bình của 2 tháng thấp nhất để thận trọng.
- Ghi rõ đã áp dụng trường hợp nào.

Bước 1 – Thu nhập khả dụng
- N = A − Thuế và khấu trừ (N là cơ sở tính % cho toàn bộ bảng).
- Tổng chi phí cố định = cộng lần lượt từng khoản.
- K1 = N − Tổng chi phí cố định.

Bước 2 – Trả nợ tối thiểu
- Tổng nợ tối thiểu = cộng lần lượt từng khoản.
- Lãi tháng của từng khoản = dư nợ × lãi suất năm ÷ 12. Kiểm tra khoản trả tối thiểu > lãi tháng.
- K2 = K1 − Tổng nợ tối thiểu.

Bước 3 – Trích lập tiết kiệm
- Tỷ lệ áp dụng = max(tỷ lệ cam kết, 20%). S = N × tỷ lệ áp dụng.
- K3 = K2 − S.
- Chi phí bắt buộc/tháng = Tổng chi phí cố định + Tổng nợ tối thiểu.
  Mục tiêu quỹ dự phòng = Chi phí bắt buộc/tháng × số tháng mục tiêu.
- Nếu quỹ chưa đủ: S chia 60% quỹ dự phòng, 40% tiết kiệm mục tiêu. Nếu đã đủ: 100% tiết kiệm mục tiêu.
- Ước tính số tháng đạt quỹ dự phòng = phần còn thiếu ÷ khoản trích hằng tháng.

Bước 4 – Chia phần còn lại theo quy tắc 50/30/20 cải tiến
- R = K3. Quy tắc áp dụng lên R (không phải lên toàn bộ thu nhập, vì chi phí cố định và nợ đã chiếm phần lớn nhu cầu):
  • 50% R: chi tiêu thiết yếu linh hoạt (ăn uống, y tế, sinh hoạt)
  • 30% R: chi tiêu cá nhân (giải trí, mua sắm, giao lưu)
  • 20% R: trả nợ vượt mức, ưu tiên khoản lãi suất cao nhất. Nếu hết nợ thì chuyển sang tiết kiệm.

Bước 5 – Kiểm tra chéo (bắt buộc)
(a) Tổng mọi khoản = N; (b) tổng % = 100,0%; (c) tổng tiết kiệm ≥ 20% N; (d) không có số âm; (e) mỗi khoản nợ được trả ≥ mức tối thiểu.
Nếu bất kỳ mục nào không đạt: sửa và tính lại, không bỏ qua.

# QUY TẮC TÍNH TOÁN (chống sai số)
- Mỗi phép tính viết theo mẫu: Tên đại lượng = công thức chữ = thay số = kết quả.
- Mỗi dòng chỉ một phép tính. Phép cộng nhiều số thì cộng dồn từng cặp.
- Không làm tròn ở bước trung gian. Phần dư do làm tròn (nếu có) dồn vào hạng mục "Chi tiêu cá nhân".
- Thiếu dữ liệu: nêu giả định rõ ràng hoặc hỏi lại, tuyệt đối không bịa số.
- Nếu K1, K2 hoặc K3 < 0: dừng, báo mức thâm hụt và đề xuất phương án (tăng thu, đàm phán lại nợ, cắt chi phí cố định). Không tự ý cắt tiết kiệm xuống dưới 20%.

# ĐỊNH DẠNG ĐẦU RA (đúng thứ tự, không đưa kết luận trước Phần 1)

## PHẦN 1: NHẬT KÝ SUY LUẬN LOGIC (Reasoning Steps)
Với mỗi bước (0 → 5): mục tiêu → công thức → thay số → kết quả trung gian → dòng kiểm tra ✓/✗.

## PHẦN 2: BẢNG PHÂN BỔ NGÂN SÁCH CUỐI CÙNG
Markdown Table, cột: | Hạng mục | Số tiền (VNĐ) | Tỷ lệ % | Ghi chú |
Dòng cuối là TỔNG = N và 100,0%. Sau bảng: 3–5 nhận xét ngắn về rủi ro và khuyến nghị.
```

---

## 3. Chạy thử với dữ liệu mẫu

**Đầu vào:**

- Thu nhập 3 tháng: 23.000.000 / 26.000.000 / 29.000.000
- Thuế và khấu trừ: 1.000.000
- Chi phí cố định: thuê nhà 6.500.000; điện, nước, internet 1.500.000; đi lại 1.000.000; bảo hiểm 800.000
- Nợ:
  - Thẻ tín dụng: dư nợ 15.000.000, lãi 30%/năm, tối thiểu 1.500.000
  - Vay tiêu dùng: dư nợ 40.000.000, lãi 18%/năm, tối thiểu 1.700.000
- Tiết kiệm: cam kết 20%; quỹ dự phòng hiện có 10.000.000; mục tiêu 3 tháng chi phí bắt buộc

### PHẦN 1: NHẬT KÝ SUY LUẬN LOGIC

**Bước 0 – Chuẩn hóa thu nhập**

- Tổng 3 tháng = 23.000.000 + 26.000.000 = 49.000.000; 49.000.000 + 29.000.000 = 78.000.000
- A = 78.000.000 ÷ 3 = **26.000.000**
- Ngưỡng 70% = 26.000.000 × 0,7 = 18.200.000. Tháng thấp nhất là 23.000.000 ≥ 18.200.000, nên dùng trung bình. ✓

**Bước 1 – Thu nhập khả dụng**

- N = 26.000.000 − 1.000.000 = **25.000.000**
- Tổng chi phí cố định: 6.500.000 + 1.500.000 = 8.000.000; + 1.000.000 = 9.000.000; + 800.000 = **9.800.000**
- K1 = 25.000.000 − 9.800.000 = **15.200.000** (≥ 0 ✓)

**Bước 2 – Nợ tối thiểu**

- Tổng nợ tối thiểu = 1.500.000 + 1.700.000 = **3.200.000**
- Lãi tháng thẻ = 15.000.000 × 0,30 ÷ 12 = 4.500.000 ÷ 12 = 375.000 → 1.500.000 > 375.000 ✓
- Lãi tháng vay = 40.000.000 × 0,18 ÷ 12 = 7.200.000 ÷ 12 = 600.000 → 1.700.000 > 600.000 ✓
- K2 = 15.200.000 − 3.200.000 = **12.000.000** (≥ 0 ✓)

**Bước 3 – Tiết kiệm**

- Tỷ lệ áp dụng = max(20%; 20%) = 20%
- S = 25.000.000 × 0,2 = **5.000.000**
- K3 = 12.000.000 − 5.000.000 = **7.000.000** (≥ 0 ✓)
- Chi phí bắt buộc/tháng = 9.800.000 + 3.200.000 = 13.000.000
- Mục tiêu quỹ = 13.000.000 × 3 = 39.000.000; còn thiếu = 39.000.000 − 10.000.000 = 29.000.000 → chưa đủ, áp dụng chia 60/40
- Quỹ dự phòng = 5.000.000 × 0,6 = **3.000.000**; tiết kiệm mục tiêu = 5.000.000 − 3.000.000 = **2.000.000**
- Thời gian đạt quỹ ≈ 29.000.000 ÷ 3.000.000 = 9,67 → **khoảng 10 tháng**

**Bước 4 – Quy tắc 50/30/20 cải tiến (R = 7.000.000)**

- Thiết yếu linh hoạt = 7.000.000 × 0,5 = **3.500.000**
- Chi tiêu cá nhân = 7.000.000 × 0,3 = **2.100.000**
- Trả nợ vượt mức = 7.000.000 × 0,2 = **1.400.000**, dồn vào thẻ tín dụng vì lãi 30% > 18%
- Kiểm tra: 3.500.000 + 2.100.000 = 5.600.000; 5.600.000 + 1.400.000 = 7.000.000 = R ✓

**Bước 5 – Kiểm tra chéo**

- (a) Cộng dồn: 9.800.000 + 1.500.000 = 11.300.000; + 1.700.000 = 13.000.000; + 3.000.000 = 16.000.000; + 2.000.000 = 18.000.000; + 3.500.000 = 21.500.000; + 2.100.000 = 23.600.000; + 1.400.000 = **25.000.000 = N** ✓
- (b) Tỷ lệ cộng dồn: 39,2 + 6,0 + 6,8 + 12,0 + 8,0 + 14,0 + 8,4 + 5,6 = **100,0%** ✓
- (c) Tiết kiệm = 3.000.000 + 2.000.000 = 5.000.000 → 5.000.000 ÷ 25.000.000 = 20,0% ≥ 20% ✓
- (d) Không có số âm ✓
- (e) Thẻ trả 1.500.000 + 1.400.000 = 2.900.000 ≥ 1.500.000; vay trả 1.700.000 ≥ 1.700.000 ✓

### PHẦN 2: BẢNG PHÂN BỔ NGÂN SÁCH CUỐI CÙNG

Cơ sở tính %: thu nhập sau thuế N = 25.000.000đ.

| Hạng mục | Số tiền (VNĐ) | Tỷ lệ % | Ghi chú |
|---|---:|---:|---|
| Chi phí cố định bắt buộc | 9.800.000 | 39,2% | Thuê nhà, điện nước, internet, đi lại, bảo hiểm |
| Trả nợ tối thiểu – Thẻ tín dụng | 1.500.000 | 6,0% | Lãi 30%/năm, bắt buộc đúng hạn |
| Trả nợ tối thiểu – Vay tiêu dùng | 1.700.000 | 6,8% | Lãi 18%/năm, bắt buộc đúng hạn |
| Quỹ dự phòng | 3.000.000 | 12,0% | Đạt mục tiêu 39.000.000 sau khoảng 10 tháng |
| Tiết kiệm/đầu tư mục tiêu | 2.000.000 | 8,0% | Cùng quỹ dự phòng tạo đúng 20% tiết kiệm |
| Chi tiêu thiết yếu linh hoạt | 3.500.000 | 14,0% | Ăn uống, y tế, sinh hoạt (50% phần còn lại) |
| Chi tiêu cá nhân | 2.100.000 | 8,4% | Giải trí, mua sắm (30% phần còn lại) |
| Trả nợ vượt mức – Thẻ tín dụng | 1.400.000 | 5,6% | Ưu tiên nợ lãi cao nhất (20% phần còn lại) |
| **TỔNG** | **25.000.000** | **100,0%** | Khớp thu nhập sau thuế |

