# Báo cáo cá nhân — Day 01

- **MSSV:** 2A202603785
- **Họ và tên:** TRương Đặng Minh Khuê
- **SEED:** L3B

## 1. Bằng chứng

| Tệp                                   | Có? |
| ------------------------------------- | --- |
| day01_lab.ipynb (đã chạy, còn output) | ☑   |
| lab1_report.json                      | ☑   |
| hand_calc.jpg                         | ☑   |

## 2. Kết quả chính

| Phương án             | F1 test | F1 SUB 40 | p95 ms/tin | USD/1k tin |
| --------------------- | ------- | --------- | ---------- | ---------- |
| 1. BoW + LogReg       | 0.904   | 0.919     | 0.08       | ~0         |
| 2. BoW + MLP          | 0.948   | 0.974     | 0.25       | ~0         |
| 3a. Emb + kNN         | 0.888   | 0.952     | 112.350    | ~0         |
| 3b. Emb + LogReg      | 0.943   | 0.947     | 44.581     | ~0         |
| 3c. Emb + MLP head    | 0.927   | 0.950     | 39.34      | ~0         |
| 4a. LLM nhỏ zero-shot | 0.909   | 0.909     | ~895       | ~0.0068    |
| 4b. LLM lớn zero-shot | 0.976   | 0.976     | ~1227      | ~0.067     |
| 4c. LLM nhỏ few-shot  | 0.952   | 0.952     | ~1087      | ~0.019     |

## 3. Dự đoán và thực tế

1. **MLP so với logistic regression (BoW):** đoán F1 tăng +0.01 đến +0.03, thực tế +0.044 (0.904 → 0.948). Tôi đánh giá thấp lợi ích của lớp ẩn: ReLU cho phép học tương tác giữa các từ nên recall tăng từ 0.833 lên 0.911.
2. **Thí nghiệm 3.B (tin ngụy trang):** đoán Encoder + LogReg đúng nhiều hơn BoW (5/6 so với 3/6), thực tế cả hai cùng 5/6. Cả hai cùng sai tin "Last chance! Your account rewards expire tonight...". Chỉ 6 mẫu nên chênh lệch là nhiễu, và BoW không tệ như tôi nghĩ vì một từ khoá đơn lẻ chưa đủ vượt ngưỡng 0.5 khi bias nghiêng về ham.
3. **Bộ 50 case khó:** đoán LLM nhỏ bị lừa ở prompt injection và accuracy ~75-85%, thực tế đúng 100% nhóm này, accuracy 98%. Đoán BoW yếu nhất ở `ham_looks_spam`, thực tế BoW đúng 100% nhóm này và yếu ở `natural_spam` (10%), `thao túng` (12%), `né từ khoá` (17%).

## 4. Phân tích 50 case khó

BoW + LogReg tụt nhiều nhất: F1 từ 0.919 (test thường) xuống 0.333, accuracy 52%. BoW + MLP cũng tụt (0.974 → 0.368). LLM nhỏ gần như không tụt (0.909 → 0.983).

- **Case 1 (né từ khoá / obfuscation):** các tin kiểu `fr33`, `w1n` bị tách thành token không có trong vocab nên bị bỏ qua, BoW không còn tín hiệu spam nào để cộng điểm.
- **Case 2 (spam giọng tự nhiên):** [điền 1 tin cụ thể từ cell "Đọc lỗi" với INSPECT = "1. BoW + LogReg"]. Tin không có các từ khoá như free/call/txt, văn phong lịch sự, nên logit bị bias âm (ham chiếm ~87% dữ liệu) kéo xuống dưới ngưỡng.

Điểm test thường "quá lạc quan" vì cùng phân phối với tập train (SMS spam kiểu cũ nhiều từ khoá). Bộ khó có spam kiểu mới nên lộ điểm yếu. Lưu ý 4d (prompt phòng thủ) đạt 100% nhưng được thiết kế sau khi thấy bộ test này nên có thể bị overfit vào bộ test.

## 5. Thử thách cá nhân

Chưa hoàn thành do hết thời gian.

## 6. Phương án cho sản phẩm thật

Yêu cầu: p95 < 50 ms/tin, 10 triệu SMS/ngày.

**Chọn: Encoder + LogReg (3b) hoặc BoW + MLP làm lớp chính**, vì p95 dưới 1 ms (BoW) so với ~0.9 đến 1.2 giây của LLM, LLM không đạt yêu cầu 50 ms và chạy tuần tự chỉ ~1 tin/giây. Chi phí suy luận gần như bằng 0. LLM nhỏ cho 10 triệu tin/ngày tốn khoảng $105/ngày (100 token input + 1 token output, giá $0.10/$0.50 mỗi 1M token).

**Đánh đổi:** mô hình nhỏ yếu hơn rõ trên spam mới/né từ khoá (3b chỉ ~70% trên 50 case khó, BoW 52%). Giảm thiểu bằng phương án lai: mô hình nhỏ xử lý phần lớn tin, chỉ gửi các tin có xác suất mơ hồ (gần 0.5) cho LLM chạy bất đồng bộ, và dùng bộ test khó làm test hồi quy.

## 7. AI log và điều đã học

| Hỏi gì                                                                  | Nhận gì                                                                                                    | Tôi sửa gì                          |
| ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| `vectorize` báo lỗi truth value của mảng                                | AI chỉ ra dòng `if X:` do tôi tự thêm                                                                      | Bỏ dòng đó, return X                |
| `sigmoid/predict_proba` báo `exceptions must derive from BaseException` | AI nghi còn dòng `raise` giữ chỗ                                                                           | Xoá dòng raise, chạy lại cell       |
| `metrics` có lỗi gõ `recal`, điều kiện F1 ngược, `accuracy: ?`          | AI chỉ 3 lỗi                                                                                               | Sửa theo                            |
| `top_k_similar` NameError (q_norm/M_unit)                               | AI chỉ chỗ sai tên biến                                                                                    | Sửa tên cho khớp                    |
| `call_openai` lỗi 400 `max_tokens`                                      | Dùng `**provider_kwargs`, trả đúng 2 giá trị                                                               | Sửa và chạy lại cell                |
| Rate limit 429 (50 request/ngày)                                        | AI giải thích là hạn mức tài khoản                                                                         | Nạp tiền                            |
| F1 = 0 vì response rỗng                                                 | AI đoán sai lúc đầu; test cho thấy cần 4 token                                                             | Tự đổi `CLASSIFY_MAX_TOKENS` sang 8 |
| Các câu dự đoán/giải thích                                              | AI viết bản mẫu, có chỗ lệch thực tế (ví dụ đoán chi phí few-shot 4-5 lần, thực tế ~2.8 lần)               | Đối chiếu với kết quả chạy thật     |
| Thử thách cá nhân                                                       | Nếu vừa học vừa làm vừa suy nghĩ nghiên cứu mà không có AI thì không có đủ thời gian (tối còn có workshop) | Không làm kịp                       |

**Điều hiểu ra:** ReLU là phi tuyến, bỏ nó thì MLP thu về logistic regression dù có 128k tham số. Điểm cao trên test thường chưa chắc bền trước dữ liệu lạ, BoW tụt từ 0.92 xuống 0.33 trên case khó. LLM bền hơn nhưng chậm và đắt, nên sản phẩm thật cần cân giữa độ chính xác, độ trễ và chi phí.
