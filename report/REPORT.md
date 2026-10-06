# Báo cáo Lab: Self evolving Agentic

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| Đinh Tiến Cảnh | 2A202602918 | Cài đặt harness, chạy thí nghiệm, phân tích và viết báo cáo |

- Nhà cung cấp/mô hình: OpenAI `gpt-4.1-mini`; `LAB_TEMPERATURE=0`; `recursion_limit=60` cho các lượt chính thức.
- Deep Agents `0.7.21`; máy chủ Windows, tác tử chạy trong container Linux từ `python:3.12-slim` để có shell tương thích.
- 18 lượt chính thức (3 điều kiện × 6 tác vụ), 3 lượt phát triển skills-auto và 3 lượt chẩn đoán lỗi recursion trước khi đổi mô hình; không đặt ngân sách cứng. Tổng token tác vụ ghi nhận xấp xỉ 2,30 triệu, chưa gồm một lần gọi curator.
- Commit `hypotheses`: `d020771`; tag `freeze`: `be69779`.

## 2. Giả thuyết (commit TRƯỚC tag `freeze`, Phần 4.0)

- H1 (subagents so với baseline): Baseline được dự đoán đạt điểm đánh giá cao hơn hoặc bằng subagents. Trên tập học, điểm trung bình của subagents giảm từ 0,445 xuống 0,408 trong khi tổng token tăng 89,5% (112.738 lên 213.631); hai lần giao việc đều dùng `general-purpose`, còn ba subagent tự định nghĩa không được gọi. Kết quả `logs-learn` còn giảm từ 1/9 xuống 0/9 vì tác tử chính phát hiện tệp rỗng nhưng không sửa, nên chưa có bằng chứng rằng chi phí giao việc tạo ra lợi ích có thể chuyển giao.
- H2 (skills-auto so với baseline): Skills-auto được dự đoán xấp xỉ baseline, không phải điều kiện tốt nhất. Cả ba lần chạy học đều có `skills_read = 0`; `code-learn` và `data-learn` giữ nguyên điểm, còn mức tăng 1/9 lên 6/9 của `logs-learn` không thể quy cho skill vì trace không đọc `SKILL.md`. Nếu hành vi chọn skill không đổi trên tập đánh giá, các quy tắc tự sinh dù hợp lệ cũng không tác động đến kết quả.
- H3 (tác vụ học so với tác vụ đánh giá): Điểm trên tác vụ đánh giá được dự đoán thấp hơn hoặc biến động mạnh hơn điểm học ở cả ba điều kiện vì dữ liệu và quy ước mới tạo ra dịch chuyển phân phối. Baseline chủ yếu thất bại ở các check `rule_*`; subagents không cải thiện lỗi này; skill tự sinh có dấu hiệu bám các chi tiết học như `order_id`, `-999` và giữ bản ghi đầu tiên, đồng thời chưa được đọc. Do đó baseline được dự đoán có điểm trung bình đánh giá cao nhất, nhưng chênh lệch nhỏ và có thể bị nhiễu giữa các lần gọi model chi phối.

## 3. Làm quen Deep Agents (Phần 0.3)

1. Công cụ mặc định gồm `ls`, `read_file`, `write_file`, `edit_file`, `delete`, `glob`, `grep`, `execute` và `task`; `execute` là công cụ chạy lệnh shell trong sandbox.
2. `task` cung cấp subagent `general-purpose` cho công việc nhiều bước. Mỗi lần gọi mặc định là stateless: subagent chỉ thấy prompt giao việc, không thấy toàn bộ hội thoại của tác tử chính, rồi trả về một báo cáo cuối; vì vậy lời giao việc phải chứa đủ mục tiêu, quy tắc và đường dẫn.
3. Hướng dẫn hành vi tiêu biểu: `task` yêu cầu “Put full detail in the prompt and state exactly what it should return”; `execute` yêu cầu dùng `grep`, `glob`, `read_file` thay cho các lệnh tìm kiếm shell như `find`, `grep`, `cat`, `head`, `tail`. System prompt mặc định của Deep Agents rỗng trước khi harness truyền `BASE_PROMPT`.

## 4. Đường cơ sở và phân loại lỗi (Phần 2.2)

| Tác vụ | Check thất bại | Nhóm lỗi (A-G) | Bằng chứng (trích ngắn từ `detail` hoặc vết) |
|---|---|---|---|
| `code-learn` | `tests_not_modified` | G — môi trường | Checker báo test gốc bị đổi, nhưng trace không có lệnh sửa test. Hash lệch do CRLF của checkout Windows so với hash LF được hard-code. |
| `code-learn` | `rule_type_hints` | E — quy ước tổ chức | `RULE: every public function ... has type annotations`. |
| `code-learn` | `rule_regression_tests` | E — quy ước tổ chức | `RULE: add tests/test_regressions.py ... at least 3`. |
| `code-learn` | `rule_changelog` | E — quy ước tổ chức | `RULE: record each fix in CHANGELOG.md under ... ## Unreleased`. |
| `data-learn` | `rule_money_in_cents` | E — quy ước tổ chức | `RULE: money values in answer.json are integer cents`. |
| `data-learn` | `rule_meta_block` | E — quy ước tổ chức | Checker yêu cầu object `meta` với `source`, `rows_in`, `rows_used`. |
| `data-learn` | `rule_clean_csv` | E — quy ước tổ chức | Checker yêu cầu `clean.csv`, header và chuẩn hóa cụ thể. |
| `logs-learn` | `entry_count` | D — bỏ sót dữ liệu/định dạng | Chỉ tạo 21 entry trong khi parser phải nhận đủ các entry ERROR/CRITICAL. |
| `logs-learn` | `timestamps_utc` | D — bỏ sót định dạng | Chỉ `8/25 timestamps match`; trace cho thấy agent tự chép JSON dài thay vì viết parser. |
| `logs-learn` | `exception_fields` | D — parser nhiều dòng sai | Có 17 giá trị `exception` sai. |
| `logs-learn` | `repeat_counts` | D — parser nhiều dòng sai | Có 17 giá trị `repeat_count` sai. |
| `logs-learn` | `counts_by_service` | B — không kiểm chứng | Tổng hợp service sai; agent không chạy checker trước khi kết thúc. |
| `logs-learn` | `rule_service_names` | E — quy ước tổ chức | `RULE: service names ... lower-case with '-' replaced by '_'`. |
| `logs-learn` | `rule_sorted_errors` | E — quy ước tổ chức | `RULE: errors is sorted by service, then by timestamp_utc`. |
| `logs-learn` | `rule_schema_header` | E — quy ước tổ chức | Checker yêu cầu `schema_version: 2` và `generated_by: log-triage`. |

Nhóm E chiếm đa số: 9/15 check thất bại có tên `rule_*`. Bằng chứng phủ định cho lỗi kỹ thuật là baseline vẫn đạt 12/18 check kỹ thuật trên tập học; các lỗi kỹ thuật tập trung ở `logs-learn`. Một skill có thể mã hóa quy ước E, nhưng chỉ có tác dụng nếu description kích hoạt đúng và agent thực sự đọc/làm theo skill.

## 5. Điều kiện `subagents` (Phần 2.3)

- Các subagent tự định nghĩa: `explorer` khảo sát read-only và báo bằng chứng; `implementer` sửa trong phạm vi rõ và kiểm chứng; `reviewer` kiểm tra độc lập, không sửa. Ba vai trò tách khám phá–triển khai–kiểm tra để giảm lỗi do một agent tự xác nhận kết quả của mình.
- Trên tập học, `subagent_calls`: `code-learn=0`, `data-learn=1`, `logs-learn=1`. Cả hai lần giao việc đều chọn `general-purpose`; `explorer`, `implementer`, `reviewer` không được gọi. Trường hợp bằng 0 ở code là hợp lệ vì tác tử chính tự giải tác vụ.
- Lời giao `data-learn` chứa đủ quy tắc dữ liệu, khoảng thời gian và schema; subagent tạo script đúng phần kỹ thuật. Lời giao `logs-learn` cũng chứa đủ quy tắc, nhưng subagent báo hoàn thành trong khi ghi `{"entries":[],"counts_by_service":{}}`; tác tử chính đọc lại tệp, phát hiện rỗng nhưng không sửa hay giao lại. Đây là thất bại kiểm chứng sau ủy quyền.
- So với baseline, tổng token học tăng 112.738 → 213.631 (+89,5%) trong khi điểm trung bình giảm 0,445 → 0,408. Theo từng tác vụ: code 44.706/36,3s → 80.106/28,0s; data 46.413/24,4s → 112.847/46,8s; logs 21.619/16,8s → 20.678/10,1s. Đa tác tử không cải thiện hiệu quả trong mẫu này.

## 6. Self-evolving: skill do curator sinh (Phần 3)

- Số lần chạy curator: 1. Curator sinh 3 skill; chưa xóa skill nào. Cả ba hợp lệ về định dạng và không chứa định danh của tác vụ đánh giá. Skill làm sạch dữ liệu có hai chỉ dẫn có nguy cơ quá khớp, nên nhóm giữ nguyên để đo thực nghiệm ở Phần 3.4 thay vì sửa tay nội dung do curator sinh.

| Skill | Tổng quát hay riêng cho tác vụ học? | Đúng hay sai (nêu chỗ sai nếu có) | Độ dài, `description` và `skills_read` ở Phần 3.4 |
|---|---|---|---|
| `enforce-code-style-and-structure` | Khá tổng quát cho tác vụ sửa mã, nhưng các yêu cầu cụ thể về type hint, tệp regression và CHANGELOG phản ánh quy ước Acme học được. Tên hàm và đáp án của tác vụ học không bị chép lại. | Đúng với phản hồi của checker. Chỉ dẫn không sửa test gốc, thêm type hint, regression test, CHANGELOG và chạy test đều an toàn. Hai ví dụ về rounding và sắp xếp hơi lệch khỏi trọng tâm cấu trúc nhưng có cụm “as specified”, nên không ép quy tắc khi đề không yêu cầu. | 12 dòng, ngắn và không lặp. `description` nêu đúng lúc sửa hoặc chuẩn bị mã nguồn. `skills_read = 0` trên `code-learn`: skill không được mở và các quy tắc type hint/regression/CHANGELOG vẫn thất bại. |
| `normalize-and-clean-data-for-analysis` | Tổng quát cho dữ liệu bảng, nhưng còn dấu vết tác vụ học qua `order_id`, sentinel `-999` và quy tắc giữ bản ghi đầu tiên. Vì vậy mức tổng quát chỉ trung bình và có nguy cơ quá khớp. | Phần lớn đúng: chuẩn hóa, xử lý ngày/múi giờ, sentinel, tiền tệ, metadata và kiểm chứng đầu ra. Hai chỉ dẫn có thể gây hại nếu áp dụng máy móc: “capitalize first letter only” làm sai acronym/tên nhiều từ; “keeping the first occurrence” không đúng khi đặc tả yêu cầu chọn bản ghi mới nhất hoặc hợp nhất. Agent phải ưu tiên đặc tả tác vụ. | 13 dòng, ngắn. `description` kích hoạt đúng khi chuẩn bị dữ liệu thô để phân tích. `skills_read = 0` trên `data-learn`: skill không được mở; điểm và ba check quy ước thất bại giống baseline. |
| `parse-and-structure-logs-for-error-reporting` | Tổng quát tốt cho log nhiều dòng và báo cáo lỗi có cấu trúc. Các trường metadata, chuẩn hóa service và thứ tự sắp xếp là quy ước Acme được phép học; không chứa dữ liệu hay đáp án cụ thể. | Đúng với toàn bộ phản hồi thất bại: đọc tuần tự, lọc level, gắn traceback, cộng repeat count, đổi UTC, tổng hợp service, sắp xếp và kiểm tra schema. Không thấy hướng dẫn sai; lưu ý chỉ áp dụng schema/metadata khi đặc tả hoặc quy ước dự án yêu cầu. | 15 dòng, dài nhất nhưng vẫn gọn và mỗi dòng là một bước kiểm chứng được. `description` nêu chính xác tình huống trích lỗi từ service log. `skills_read = 0` trên `logs-learn`: skill không được mở. |

Kết quả Phần 3.4: không tác vụ nào đọc skill (`skills_read = 0` ở cả ba lần chạy), dù `skills_modified = false` và không lần chạy nào có lỗi. `code-learn` giữ nguyên 6/10, `data-learn` giữ nguyên 5/8, còn `logs-learn` tăng từ 1/9 lên 6/9. Vì trace không có lệnh đọc `skills/.../SKILL.md`, mức tăng của `logs-learn` là nhiễu giữa các lần chạy hoặc do cách model tự giải khác, không phải bằng chứng về tác dụng của skill. Token tăng ở cả ba tác vụ: 44.706 → 97.117, 46.413 → 70.309 và 21.619 → 34.216.

## 7. Kết quả so sánh (Phần 4.3, 4.4)

| Task | baseline | subagents | skills-auto |
|---|---|---|---|
| code-learn | 6/10 | 6/10 | 7/10 |
| data-learn | 5/8 | 5/8 | 5/8 |
| logs-learn | 1/9 | 0/9 | 1/9 |
| code-eval | 6/11 | 7/11 | 6/11 |
| data-eval | 5/9 | 2/9 | 5/9 |
| logs-eval | 1/10 | 0/10 | 1/10 |
| **Mean score - learning tasks** | 0.45 | 0.41 | 0.48 |
| **Mean score - evaluation tasks** | 0.40 | 0.29 | 0.40 |
| **Mean tokens per run** | 44,477 | 78,538 | 72,457 |
| **Runs that read a skill** | 0/6 | 0/6 | 0/6 |

Thống kê theo loại check:

```text
condition     role    technical  house rules  mean tokens  read a skill
baseline      eval     12/18         0/12          51,376      0/3
baseline      learn    12/18         0/9           37,579      0/3
subagents     eval      9/18         0/12          85,866      0/3
subagents     learn    11/18         0/9           71,210      0/3
skills-auto   eval     12/18         0/12          71,038      0/3
skills-auto   learn    13/18         0/9           73,877      0/3
```

Không lần chạy chính thức nào có `error` hoặc `skills_modified = true`. `verify_freeze.py` kiểm tra 6 lần chạy skill và trả về `OK`.

## 8. Phân tích

1. Trên tập học, skills-auto tăng điểm trung bình từ 0,45 lên 0,48, còn subagents giảm xuống 0,41. Trên evaluation, skills-auto bằng baseline (0,40), subagents giảm còn 0,29; chỉ riêng `code-eval` của subagents tăng 6/11 → 7/11 nhưng bị lấn át bởi data 5/9 → 2/9 và logs 1/10 → 0/10. Skills-auto cải thiện học nhưng không cải thiện evaluation là dấu hiệu không chuyển giao; do không skill nào được đọc, chênh lệch này phù hợp với nhiễu hơn là học được quy trình.
2. Skills-auto đạt 13/18 check kỹ thuật trên tập học, hơn baseline 12/18 đúng một check, nhưng bằng baseline 12/18 trên evaluation. Mọi điều kiện đều đạt 0 check quy ước: 0/9 khi học và 0/12 khi đánh giá. Do `skills_read=0`, các quy tắc type hint, metadata, chuẩn hóa service và schema trong skill không được áp dụng; các quy ước mới của evaluation cũng không được giúp.
3. Không có check nào có thể quy nhân quả là “skill giúp đạt”: `tests_not_modified` chuyển từ fail ở baseline/dev sang pass trong lượt skills-auto đóng băng, nhưng trace không đọc skill và lỗi này liên quan hash/line ending, nên chỉ là biến thiên môi trường. Ví dụ rõ về check không được giúp là `rule_type_hints`: skill code ghi trực tiếp yêu cầu type annotation, nhưng `skills_read=0`, trace không mở `SKILL.md` và check vẫn fail. Tương tự, ba check quy ước data và logs tiếp tục fail.
4. Token trung bình mỗi lượt là baseline 44.477, skills-auto 72.457 (+62,9%) và subagents 78.538 (+76,6%). Riêng evaluation, hiệu quả điểm/token xấp xỉ: baseline 0,40/51.376 = 7,79 điểm chuẩn hóa trên một triệu token; skills-auto 5,63; subagents 3,38. Baseline hiệu quả nhất; đa tác tử không đáng chi phí trong thí nghiệm này vì vừa tốn token nhất vừa có điểm evaluation thấp nhất.
5. Không thấy rò rỉ evaluation: curator chỉ đọc run `role=learn`, `validate_skill` loại marker evaluation và `verify_freeze.py` xác nhận hash skill đóng băng. Có dấu hiệu quá khớp trong skill dữ liệu qua `order_id`, sentinel `-999`, quy tắc luôn giữ bản ghi đầu tiên; skill code/log cũng ghi chính xác các house rule học được. Nhóm không sửa tay, ghi nhận rủi ro trước evaluation, commit giả thuyết rồi freeze; kết quả 0/12 house rule evaluation cho thấy các quy tắc đó không chuyển giao trong lần chạy này.
6. Với cùng bộ skill, Phần 3.4 → sau đóng băng: `code-learn` 6/10 → 7/10 (+0,10), `data-learn` 5/8 → 5/8 (0), `logs-learn` 6/9 → 1/9 (-0,556); mean giảm khoảng 0,631 → 0,479 (-0,152). Biến động rất lớn ở logs dù skill không đổi và không được đọc cho thấy một lượt chạy/model có nhiễu đáng kể. Vì vậy chênh lệch một check không đủ để kết luận tác dụng; cần nhiều lần lặp và khoảng tin cậy.

## 9. Hạn chế và tính hợp lệ

1. Chỉ có ba họ tác vụ và ba tác vụ evaluation; một ngoại lệ như `logs-*` chi phối mạnh điểm trung bình, nên không thể khái quát sang mọi tác vụ agentic.
2. Mỗi cặp condition–task chính thức chỉ chạy một lần. Chênh lệch `logs-learn` 6/9 → 1/9 với cùng skill chứng minh phương sai theo lần gọi lớn; không có độ lệch chuẩn hay khoảng tin cậy.
3. Chỉ dùng một mô hình (`gpt-4.1-mini`, temperature 0). Temperature 0 không loại bỏ hoàn toàn bất định dịch vụ; kết luận có thể đổi với model mạnh/yếu hơn hoặc provider khác.
4. Không skill nào được đọc, nên thí nghiệm đo cả cơ chế chọn skill lẫn chất lượng nội dung nhưng không tách được hai yếu tố. Không thể kết luận skill “không hữu ích”; chỉ kết luận pipeline hiện tại không tạo hiệu quả quan sát được.
5. House rule do bộ tác vụ thiết kế và hoàn toàn ẩn khỏi đề, làm 27/27 check quy ước học và 36/36 check quy ước evaluation của ba điều kiện đều thất bại. Thiết kế này hữu ích để đo học quy ước nhưng không đại diện mọi dự án thực tế.
6. Checkout Windows gây lệch hash CRLF cho `tests_not_modified`, còn run thực hiện trong Linux Docker. Nhóm ghi nhận đây là lỗi môi trường và chuẩn hóa đường dẫn hash khi verify; tuy nhiên một check code học vẫn bị nhiễu bởi khác biệt nền tảng.

## 10. Kết luận

Baseline đạt điểm evaluation trung bình 0,40 với chi phí thấp nhất, skills-auto cũng đạt 0,40 nhưng tốn thêm 62,9% token, còn subagents chỉ đạt 0,29 và tốn thêm 76,6%. Không có lần chạy nào đọc skill và không điều kiện nào đạt house rule evaluation, nên chưa có bằng chứng rằng skill tự sinh hoặc đa tác tử cải thiện khả năng chuyển giao. Các subagent tự định nghĩa không được chọn, còn hai lần dùng `general-purpose` không cải thiện điểm tổng thể. Kết quả giữa hai lượt skills-auto học biến động mạnh, vì vậy các chênh lệch nhỏ không đáng tin khi chỉ chạy một lần. Bước tiếp theo nên cải thiện description/cơ chế buộc đọc skill rồi chạy lặp mỗi cấu hình nhiều lần để tách hiệu quả skill khỏi nhiễu.

## Phụ lục

- Lệnh chính: `pytest`; `python scripts/tour.py`; `python -m lab.runner --condition baseline --tasks ...`; `python -m lab.runner --condition subagents --tasks ...`; `python -m lab.curator`; `python -m lab.runner --condition skills-auto --tasks learn`; commit `hypotheses`; tag `freeze`; chạy evaluation; `python scripts/verify_freeze.py`; `python -m lab.compare`; `python scripts/check_breakdown.py`. Các lượt agent chạy trong Docker với repo bind vào `/lab`.
- Thử thách mở rộng v2: thay ba vai trò tuần tự chung (`explorer`, `implementer`, `reviewer`) bằng ba specialist end-to-end (`code-fixer`, `data-analyst`, `log-analyst`), dùng description tiếng Anh và routing ưu tiên specialist. Trên tập học v2b, code giữ 6/10, data giữ 5/8, logs tăng 0/9 → 1/9; tổng token giảm 213.631 → 124.787 (-41,6%). `data-analyst` được chọn một lần; code và logs vẫn do agent chính xử lý. Một biến thể ép buộc delegation (v2c) làm điểm giảm còn 5/10, 0/8, 0/9 do subagent báo hoàn thành sai và agent chính không kiểm chứng, nên bị loại. Không chạy lại evaluation cho v2 vì kết quả evaluation v1 đã được xem; làm vậy sẽ gây leakage.
- `results/skills-auto-dev/` giữ kết quả Phần 3.4 trước khi chạy lại skills-auto sau freeze. Công cụ verify ban đầu gặp hai lỗi portability Windows/Docker (encoding Git và dấu phân cách đường dẫn hash); đã sửa ngoài `skills/auto/`, sau đó kết quả là `checked 6 runs of skill conditions: OK`.
