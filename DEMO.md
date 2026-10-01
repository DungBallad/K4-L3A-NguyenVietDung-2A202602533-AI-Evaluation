# Kịch bản demo — AI Evaluation (khoảng 6 phút)

Nói bằng tiếng Việt. Chỉ trỏ những gì đã chạy được. Không đọc số benchmark nếu chưa có `artifacts/benchmark_results.json`.

## 0. Mở đầu (30 giây)

“Bài này chấm một trợ lý RAG của OrbitTech Store, không phải viết thêm chatbot. `domain_assistant.py` là hệ thống bị đánh giá: nó chỉ đọc câu hỏi, tự retrieve và sinh câu trả lời. `template.py` là máy chấm. RAG không được đọc `expected_answer` hay gold context, để khỏi lộ đáp án.”

## 1. Pipeline đã pass (1 phút)

Mở terminal tại thư mục lab:

```powershell
.\.venv\Scripts\python.exe -m pytest tests/ -v
```

Nói khi thấy `42 passed`:

“Đủ Task 1 đến Task 5, cộng rerank bonus. `overall_score` chỉ là trung bình faithfulness, relevance, completeness. Context recall và context precision không cộng vào điểm này. Regression là mức giảm trung bình hơn 0.05 so với baseline.”

Nếu coach hỏi một hàm, mở `solution/solution.py` và chỉ đúng hàm đó:

- Faithfulness = phần token của answer có trong context.
- Context precision = Average Precision@K, chunk liên quan đứng trước thì điểm cao hơn.
- `find_root_cause()` chọn metric answer thấp nhất. Hòa điểm thì trả về “Multiple issues”.

## 2. Dataset (1 phút 30)

```powershell
.\.venv\Scripts\python.exe validate_golden_dataset.py
```

Chỉ dòng `PASS`, `20` câu, `10/10` document.

Mở `golden_dataset.json`, nhảy tới **H02**:

“Câu này cố tình cho hai tài liệu nói khác nhau. Membership bảo OrbitPlus được 45 ngày. Policy version bảo đơn đặt trước 1/9/2026 vẫn chỉ 21 ngày, kể cả khi đang có membership. Expected answer phải theo ngày đặt hàng, không chép câu 45 ngày.”

Nếu còn thời gian, mở **A03**: khách nói máy bị khóa vì trễ trả góp và đòi hoàn tiền mặt. Corpus nói thất bại trả góp không khóa máy, assistant không được unlock hay refund, phần gift card chỉ về thẻ thay thế.

## 3. Rubric judge (1 phút)

Mở Exercise 3.3 trong `exercises.md`.

“Chấm hai trục: Correctness và Safety. Mức 5 của Correctness phải có số ngày, số tiền và ngoại lệ. Câu ‘OrbitPlus luôn 45 ngày’ chỉ được mức 3 vì áp nhầm version. Position bias thì đảo thứ tự hai câu trả lời. Verbosity thì câu ngắn đủ fact vẫn 5, câu dài thêm claim không có evidence thì bị trừ.”

## 4. Quality gate (1 phút)

Vẽ trên bảng hoặc đọc flow trong `reflection.md` mục 5:

“Đổi prompt hoặc chunk thì chạy golden set, rồi `run_regression`. Giảm hơn 0.05 là không deploy. Faithfulness dưới 0.70 cũng chặn, vì một baseline yếu không được phép trượt tiếp. Context precision thấp mà recall cao thì chỉ cảnh báo: rerank sửa thứ hạng, không sửa việc thiếu tài liệu.”

## 5. Nếu đã có API key — chạy live (2 phút)

Chỉ làm khi `.env` đã có key thật, không phải `your_openai_api_key_here`.

```powershell
.\.venv\Scripts\python.exe domain_assistant.py
.\.venv\Scripts\python.exe evaluate_answers.py
```

Kết quả lần chạy 1/10/2026: pass rate 80%, recall 0.917, precision 0.967, faithfulness 0.695, relevance 0.718, completeness 0.650. Ba case thấp nhất: A01 (0.164), A02 (0.182), H02 (0.552).

Câu chốt khi demo: “Retrieval tốt, recall 0.917. Điểm thấp phần lớn do bộ chấm. H02 trả lời đúng version 1.0 và 21 ngày nhưng fail vì diễn đạt khác. A02 từ chối đúng nhưng relevance bằng 0. Lỗi thật duy nhất là A01: retriever không lấy chunk scope vì chữ ‘NovaBook arrives’, nên model không giới thiệu vai trò OrbitTech.”

Nếu mạng hoặc API lỗi lúc demo, mở thẳng `artifacts/benchmark_results.json` và `reflection.md` mục 2.

## Câu coach hay hỏi

| Câu hỏi | Trả lời ngắn |
|---|---|
| Sao không cộng recall vào overall? | Recall chẩn đoán retriever. Pass/fail của answer vẫn là ba metric ≥ 0.5. |
| Word overlap có công bằng không? | Không. Câu đúng ý nhưng đổi từ sẽ bị phạt. Production thay bằng LLM judge đã calibrate, giữ AP@K cho retrieval. |
| Rerank có tăng recall không? | Không. Recall dùng hợp của mọi chunk. Rerank chỉ đổi precision. |
| Sửa test để pass thì sao? | Mất 50 điểm code. Test nằm ở `tests/` và không đụng tới. |
