# Báo cáo Lab: Self evolving Agentic

> Sao chép tệp này thành `report/REPORT.md` (đã làm ở Phần 0) và điền dần qua các Phần của lab. Xóa các dòng hướng dẫn dạng trích dẫn (bắt đầu bằng `>`). Văn phong kỹ thuật, ngắn gọn, mọi nhận định đi kèm số liệu hoặc bằng chứng. Trong buổi học: điền mục 1 đến 7 (bản nháp). Sau buổi học: hoàn thiện mục 8 đến 10.

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| | | |

- Nhà cung cấp và mô hình (`LAB_MODEL`, không ghi khóa API), nhiệt độ (`LAB_TEMPERATURE`), `recursion_limit`:
- Phiên bản Deep Agents (`pip show deepagents`), hệ điều hành, chạy trực tiếp hay trong Docker:
- Số lần chạy tác vụ đã dùng / ngân sách:
- Commit của tag `freeze`:

## 2. Giả thuyết (commit TRƯỚC tag `freeze`, Phần 4.0)

> Dự đoán điều kiện nào đạt điểm cao nhất trên **tác vụ đánh giá** và vì sao. Nêu căn cứ từ phân loại lỗi (mục 4) và từ tài liệu tham khảo. Điền cả ba dòng; `verify_freeze.py` kiểm tra điều này.

- H1 (subagents so với baseline): Baseline được dự đoán đạt điểm đánh giá cao hơn hoặc bằng subagents. Trên tập học, điểm trung bình của subagents giảm từ 0,445 xuống 0,408 trong khi tổng token tăng 89,5% (112.738 lên 213.631); hai lần giao việc đều dùng `general-purpose`, còn ba subagent tự định nghĩa không được gọi. Kết quả `logs-learn` còn giảm từ 1/9 xuống 0/9 vì tác tử chính phát hiện tệp rỗng nhưng không sửa, nên chưa có bằng chứng rằng chi phí giao việc tạo ra lợi ích có thể chuyển giao.
- H2 (skills-auto so với baseline): Skills-auto được dự đoán xấp xỉ baseline, không phải điều kiện tốt nhất. Cả ba lần chạy học đều có `skills_read = 0`; `code-learn` và `data-learn` giữ nguyên điểm, còn mức tăng 1/9 lên 6/9 của `logs-learn` không thể quy cho skill vì trace không đọc `SKILL.md`. Nếu hành vi chọn skill không đổi trên tập đánh giá, các quy tắc tự sinh dù hợp lệ cũng không tác động đến kết quả.
- H3 (tác vụ học so với tác vụ đánh giá): Điểm trên tác vụ đánh giá được dự đoán thấp hơn hoặc biến động mạnh hơn điểm học ở cả ba điều kiện vì dữ liệu và quy ước mới tạo ra dịch chuyển phân phối. Baseline chủ yếu thất bại ở các check `rule_*`; subagents không cải thiện lỗi này; skill tự sinh có dấu hiệu bám các chi tiết học như `order_id`, `-999` và giữ bản ghi đầu tiên, đồng thời chưa được đọc. Do đó baseline được dự đoán có điểm trung bình đánh giá cao nhất, nhưng chênh lệch nhỏ và có thể bị nhiễu giữa các lần gọi model chi phối.

## 3. Làm quen Deep Agents (Phần 0.3)

1.
2.
3.

## 4. Đường cơ sở và phân loại lỗi (Phần 2.2)

> Chỉ dùng tác vụ học. Mỗi dòng là một check thất bại.

| Tác vụ | Check thất bại | Nhóm lỗi (A-G) | Bằng chứng (trích ngắn từ `detail` hoặc vết) |
|---|---|---|---|
| | | | |

Nhận xét: nhóm lỗi nào chiếm đa số? Skill có thể phòng ngừa nhóm đó không?

## 5. Điều kiện `subagents` (Phần 2.3)

- Các subagent đã định nghĩa (tên, vai trò, lý do thiết kế):
- `subagent_calls` ở từng tác vụ và nhận xét (kể cả trường hợp bằng 0):
- Thông tin thiếu hoặc thừa khi giao việc (nếu có giao việc):
- Ảnh hưởng đến token và thời gian:

## 6. Self-evolving: skill do curator sinh (Phần 3)

- Số lần chạy curator: 1. Curator sinh 3 skill; chưa xóa skill nào. Cả ba hợp lệ về định dạng và không chứa định danh của tác vụ đánh giá. Skill làm sạch dữ liệu có hai chỉ dẫn có nguy cơ quá khớp, nên nhóm giữ nguyên để đo thực nghiệm ở Phần 3.4 thay vì sửa tay nội dung do curator sinh.

| Skill | Tổng quát hay riêng cho tác vụ học? | Đúng hay sai (nêu chỗ sai nếu có) | Độ dài, `description` và `skills_read` ở Phần 3.4 |
|---|---|---|---|
| `enforce-code-style-and-structure` | Khá tổng quát cho tác vụ sửa mã, nhưng các yêu cầu cụ thể về type hint, tệp regression và CHANGELOG phản ánh quy ước Acme học được. Tên hàm và đáp án của tác vụ học không bị chép lại. | Đúng với phản hồi của checker. Chỉ dẫn không sửa test gốc, thêm type hint, regression test, CHANGELOG và chạy test đều an toàn. Hai ví dụ về rounding và sắp xếp hơi lệch khỏi trọng tâm cấu trúc nhưng có cụm “as specified”, nên không ép quy tắc khi đề không yêu cầu. | 12 dòng, ngắn và không lặp. `description` nêu đúng lúc sửa hoặc chuẩn bị mã nguồn. `skills_read = 0` trên `code-learn`: skill không được mở và các quy tắc type hint/regression/CHANGELOG vẫn thất bại. |
| `normalize-and-clean-data-for-analysis` | Tổng quát cho dữ liệu bảng, nhưng còn dấu vết tác vụ học qua `order_id`, sentinel `-999` và quy tắc giữ bản ghi đầu tiên. Vì vậy mức tổng quát chỉ trung bình và có nguy cơ quá khớp. | Phần lớn đúng: chuẩn hóa, xử lý ngày/múi giờ, sentinel, tiền tệ, metadata và kiểm chứng đầu ra. Hai chỉ dẫn có thể gây hại nếu áp dụng máy móc: “capitalize first letter only” làm sai acronym/tên nhiều từ; “keeping the first occurrence” không đúng khi đặc tả yêu cầu chọn bản ghi mới nhất hoặc hợp nhất. Agent phải ưu tiên đặc tả tác vụ. | 13 dòng, ngắn. `description` kích hoạt đúng khi chuẩn bị dữ liệu thô để phân tích. `skills_read = 0` trên `data-learn`: skill không được mở; điểm và ba check quy ước thất bại giống baseline. |
| `parse-and-structure-logs-for-error-reporting` | Tổng quát tốt cho log nhiều dòng và báo cáo lỗi có cấu trúc. Các trường metadata, chuẩn hóa service và thứ tự sắp xếp là quy ước Acme được phép học; không chứa dữ liệu hay đáp án cụ thể. | Đúng với toàn bộ phản hồi thất bại: đọc tuần tự, lọc level, gắn traceback, cộng repeat count, đổi UTC, tổng hợp service, sắp xếp và kiểm tra schema. Không thấy hướng dẫn sai; lưu ý chỉ áp dụng schema/metadata khi đặc tả hoặc quy ước dự án yêu cầu. | 15 dòng, dài nhất nhưng vẫn gọn và mỗi dòng là một bước kiểm chứng được. `description` nêu chính xác tình huống trích lỗi từ service log. `skills_read = 0` trên `logs-learn`: skill không được mở. |

Kết quả Phần 3.4: không tác vụ nào đọc skill (`skills_read = 0` ở cả ba lần chạy), dù `skills_modified = false` và không lần chạy nào có lỗi. `code-learn` giữ nguyên 6/10, `data-learn` giữ nguyên 5/8, còn `logs-learn` tăng từ 1/9 lên 6/9. Vì trace không có lệnh đọc `skills/.../SKILL.md`, mức tăng của `logs-learn` là nhiễu giữa các lần chạy hoặc do cách model tự giải khác, không phải bằng chứng về tác dụng của skill. Token tăng ở cả ba tác vụ: 44.706 → 97.117, 46.413 → 70.309 và 21.619 → 34.216.

## 7. Kết quả so sánh (Phần 4.3, 4.4)

> Dán nội dung `report/table.md` và kết quả `python scripts/check_breakdown.py`. Nêu các lần chạy có `error` hoặc `skills_modified = true` (nếu có) và cách xử lý.

```text
(dán bảng ở đây)
```

## 8. Phân tích

> Trả lời từng câu bằng số liệu từ mục 7 và bằng chứng từ vết. Kết quả âm hoặc không có khác biệt vẫn hợp lệ nếu được phân tích tốt.

1. So với `baseline`, điều kiện nào cải thiện điểm tác vụ **học**? Điều kiện nào cải thiện điểm tác vụ **đánh giá**? Có điều kiện nào cải thiện tác vụ học nhưng không cải thiện tác vụ đánh giá? Nếu có, đó là dấu hiệu gì?
2. Tách điểm thành check kỹ thuật và check quy ước (`rule_`). Skill do curator sinh giúp nhóm check nào? Check quy ước **mới** của tác vụ đánh giá có được skill giúp không, và vì sao?
3. Dựa vào vết và `skills_read`, giải thích một check mà skill giúp đạt và một check mà skill không giúp (skill chưa được đọc, đọc nhưng không làm theo, skill thiếu hoặc sai).
4. Chi phí: so sánh số token trung bình giữa các điều kiện. Điều kiện nào có hiệu quả tốt nhất theo điểm trên mỗi token? Đa tác tử có đáng chi phí trong thí nghiệm này không?
5. Có dấu hiệu rò rỉ dữ liệu hoặc quá khớp nào trong skill sinh ra không? Nhóm đã phòng tránh như thế nào?
6. Nhiễu: so sánh điểm tác vụ học của cùng bộ skill ở Phần 3.4 (đã sao lưu) và sau đóng băng. Chênh lệch bao nhiêu? Nó cho biết điều gì về độ tin cậy của các chênh lệch trong bảng ở mục 7?

## 9. Hạn chế và tính hợp lệ

> Nêu ít nhất 3 hạn chế và ảnh hưởng của từng hạn chế đến kết luận (ví dụ: chỉ 3 tác vụ mỗi vai trò, mỗi cấu hình chạy một lần, nhiễu của mô hình, tác vụ do giảng viên thiết kế sẵn quy ước, chỉ một mô hình).

1.
2.
3.

## 10. Kết luận

> Tối đa 5 câu. Chỉ khẳng định điều số liệu hỗ trợ. Nêu một đề xuất cải tiến tiếp theo.

## Phụ lục

- Lệnh đã chạy (theo thứ tự):
- Thử thách mở rộng (nếu có): hướng chọn, kết quả, nhận xét.
- Ghi chú khác:
