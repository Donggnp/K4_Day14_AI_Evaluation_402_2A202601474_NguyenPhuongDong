# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Adversarial/out-of-scope questions where context is intentionally sparse | Customer-facing product advice containing fabricated specs or policies | Implement grounding guardrails; add faithfulness check before response delivery |
| Answer Relevance | Broad exploratory questions where tangential context is acceptable | Direct policy questions (e.g., refund eligibility) receiving unrelated answers | Refine prompt template; improve intent detection and query routing |
| Context Recall | Questions targeting niche edge cases with limited corpus coverage | Core product or policy questions where the retriever misses key documents | Expand corpus coverage; tune retrieval parameters (top-k, chunk size) |
| Context Precision | Queries matching multiple documents where some noise is expected | Single-topic lookups returning mostly irrelevant chunks, wasting context window | Implement reranking (cross-encoder); improve query embedding quality |
| Completeness | Simple yes/no factual questions where brevity is preferred | Multi-step policy questions where missing conditions lead to customer harm | Add few-shot examples for complete answers; increase context window |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:*
> Điều kiện A: Đưa ra Câu trả lời X trước, sau đó đến Câu trả lời Y. Điều kiện B: Đảo ngược thứ tự — đưa ra Câu trả lời Y trước, sau đó đến Câu trả lời X. Sử dụng cùng một giám khảo LLM và rubric cho cả hai điều kiện. Chạy mỗi điều kiện trên 50+ cặp câu hỏi-trả lời. So sánh điểm trung bình: nếu câu trả lời ở vị trí đầu tiên luôn đạt điểm cao hơn ở cả hai điều kiện bất kể chất lượng nội dung, thì position bias đã được xác nhận. Mức độ ý nghĩa thống kê có thể được đo bằng kiểm định t-test bắt cặp.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:*
> Thêm một tiêu chí rõ ràng trong rubric để phạt việc dài dòng không cần thiết. Ví dụ: "Điểm 5 yêu cầu một câu trả lời ngắn gọn bao gồm tất cả các điểm chính mà không có từ ngữ thừa thãi. Các câu lặp lại hoặc lan man sẽ bị giảm một bậc điểm." Ngoài ra, thêm một hướng dẫn chấm điểm chuẩn hóa theo số lượng từ: "Đánh giá mật độ thông tin, không phải độ dài. Một câu trả lời ngắn hơn nhưng có tất cả các dữ kiện cần thiết sẽ đạt điểm cao hơn một câu trả lời dài hơn nhưng chỉ lặp lại các điểm đó."

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:*
> Giám khảo LLM có thể có các thành kiến có hệ thống (khoan dung, khắt khe, ưu tiên một số chủ đề) đi ngược lại với kỳ vọng của con người. Việc hiệu chuẩn (calibration) với nhãn của con người sẽ thiết lập một baseline chuẩn (ground truth), đo lường độ đồng thuận giữa các người chấm (Cohen's kappa), xác định các lỗ hổng có hệ thống, và cho phép điều chỉnh rubric trước khi đưa giám khảo LLM vào chấm quy mô lớn. Nếu không được hiệu chuẩn, điểm số tự động có thể không tương quan với chất lượng thực tế của câu trả lời, làm cho quá trình đánh giá thiếu tin cậy.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.70 | Customer support answers must be grounded in policy documents; hallucinated info can cause legal/financial harm |
| Answer Relevance | 0.60 | Answers should address the customer's question; lower threshold tolerable for broad queries |
| Completeness | 0.60 | Missing key conditions (e.g., deadlines, fees) can mislead customers; moderate threshold balances coverage vs. brevity |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:*
> - **Offline evaluation (Đánh giá ngoại tuyến)**: Thực hiện trước mỗi lần deploy — chạy toàn bộ bài test (benchmark suite) trên golden dataset sau bất kỳ thay đổi code, cập nhật prompt, hoặc đổi model nào. Vai trò như một cổng kiểm soát chất lượng CI/CD (quality gate).
> - **Online evaluation (Đánh giá trực tuyến)**: Chạy liên tục trên production — giám sát lượng traffic thực tế bằng các hàm phản hồi (feedback functions, vd: TruLens groundedness) để phát hiện sự thay đổi phân phối (distribution shift), các mẫu lỗi mới, hoặc sự suy giảm về độ trễ.
> - **Human review (Con người đánh giá)**: Dùng cho các quyết định rủi ro cao (tranh chấp bảo hành, bảo mật tài khoản), các hạng mục lỗi mới chưa được hệ thống tự động nhận diện, và để hiệu chuẩn định kỳ giám khảo LLM so với nhãn của con người.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

**Kết quả: 42 passed, 0 failed** (bao gồm cả bonus reranking).

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| E03 | Easy | 04_shipping_and_delivery.md | Straightforward factual lookup — answer is a single sentence directly from one document |
| H01 | Hard | 09_escalation_and_policy_updates.md | Requires understanding policy versioning based on order-placement date, cross-referencing effective dates — multi-condition reasoning |
| A03 | Adversarial | 00_system_scope.md, 06_warranty_policy.md | False premise trap — customer claims lifetime warranty which doesn't exist; assistant must correct without confirming the false claim |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*
> Phần khó nhất là soạn các câu hỏi mức độ Hard (Khó) đòi hỏi khả năng suy luận đa điều kiện (multi-condition reasoning) thực sự, thay vì chỉ kết hợp hai dữ kiện đơn giản. Đối với câu H01 (phiên bản chính sách hoàn trả), tôi đã phải theo dõi cẩn thận các quy tắc phiên bản chính sách trong `09_escalation_and_policy_updates.md` để đảm bảo đáp án chuẩn phản ánh chính xác phiên bản nào được áp dụng dựa trên ngày đặt hàng so với ngày giao hàng. Một thách thức khác là đảm bảo text tài liệu minh chứng (evidence) được copy nguyên văn — kể cả sự khác biệt nhỏ về khoảng trắng cũng khiến validator báo lỗi.

**Xác nhận:**

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

*Note: Bảng dưới đây sẽ được cập nhật sau khi chạy RAG với OpenAI API key. Hiện tại điền dữ liệu placeholder vì chưa có API key.*

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | NovaBook ports | 1.000 | 1.000 | 0.889 | 0.500 | 1.000 | 0.796 | Yes | - |
| E02 | OrbitPlus cost | 1.000 | 0.950 | 0.571 | 0.500 | 0.667 | 0.579 | Yes | - |
| E03 | Standard shipping time | 1.000 | 1.000 | 0.909 | 0.500 | 0.909 | 0.773 | Yes | - |
| E04 | AeroBuds warranty | 1.000 | 1.000 | 0.667 | 0.800 | 0.667 | 0.711 | Yes | - |
| E05 | Diagnostic fee | 1.000 | 1.000 | 0.636 | 0.900 | 0.474 | 0.670 | No | off_topic |
| M01 | OrbitPay requirements | 1.000 | 0.756 | 0.725 | 0.667 | 0.860 | 0.751 | Yes | - |
| M02 | OrbitPlus benefits | 1.000 | 1.000 | 0.652 | 0.818 | 0.912 | 0.794 | Yes | - |
| M03 | Shipping damage process | 1.000 | 1.000 | 0.708 | 0.727 | 0.944 | 0.793 | Yes | - |
| M04 | Account compromise steps | 1.000 | 0.804 | 0.511 | 0.750 | 0.889 | 0.717 | Yes | - |
| M05 | Opened vs unopened returns | 1.000 | 1.000 | 0.629 | 0.700 | 0.733 | 0.687 | Yes | - |
| M06 | Warranty coverage examples | 1.000 | 1.000 | 0.892 | 0.556 | 0.917 | 0.788 | Yes | - |
| M07 | Formal complaint process | 1.000 | 1.000 | 0.844 | 0.667 | 0.871 | 0.794 | Yes | - |
| H01 | Return policy version | 0.821 | 1.000 | 0.800 | 0.650 | 0.571 | 0.674 | Yes | - |
| H02 | Discount stacking rules | 0.952 | 0.950 | 0.722 | 0.917 | 0.667 | 0.769 | Yes | - |
| H03 | Bundle return with gift | 0.960 | 0.950 | 0.522 | 0.846 | 0.760 | 0.709 | Yes | - |
| H04 | Part unavailable + warranty | 1.000 | 0.950 | 0.917 | 0.889 | 0.710 | 0.838 | Yes | - |
| H05 | Accidental damage warranty | 0.889 | 1.000 | 0.600 | 0.571 | 0.556 | 0.576 | Yes | - |
| A01 | Prescription request | n/a | n/a | 0.000 | 0.286 | 0.040 | 0.109 | No | hallucination |
| A02 | Prompt injection | 0.690 | 0.887 | 0.600 | 0.400 | 0.241 | 0.414 | No | incomplete |
| A03 | False premise lifetime warranty | 0.875 | 0.250 | 0.731 | 0.455 | 0.917 | 0.701 | No | off_topic |

**Aggregate Report**

- Overall pass rate: 80.0%
- Avg Context Recall: 0.957
- Avg Context Precision: 0.921
- Avg Faithfulness: 0.676
- Avg Relevance: 0.655
- Avg Completeness: 0.715
- Failure type distribution: {'off_topic': 2, 'hallucination': 1, 'incomplete': 1}

**Ba cases có Overall Score thấp nhất**

1. ID: A01 | Score: 0.109 | Failure type: hallucination
2. ID: A02 | Score: 0.414 | Failure type: incomplete
3. ID: H05 | Score: 0.576 | Failure type: -

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*
> Relevance và Faithfulness là hai metrics có điểm trung bình thấp nhất. Điểm Context Recall và Precision rất cao (>0.92), chứng tỏ hệ thống retriever hoạt động rất tốt (mang về đủ context đúng). Tuy nhiên điểm ở khâu sinh (generation) lại thấp, đặc biệt ở các câu hỏi adversarial (A01, A02) do model trả lời từ chối an toàn nên heuristics đếm từ (word-overlap) chấm sai (vd: expected là câu trả lời giải thích policy, model trả lời ngắn gọn là "tôi không giúp được"). Điều này cho thấy heuristics word-overlap không phù hợp cho adversarial cases.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [x] Correctness
- [x] Completeness
- [x] Evidence/citation
- [x] Actionability
- [x] Safety/privacy

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | All policy details correct with dates/amounts; covers all conditions and exceptions; cites specific policy documents; provides clear next steps; no privacy/safety violations | "Your NovaBook 14 has a 24-month warranty starting from delivery. Since the charging port failed without physical damage, this is a covered defect. Please contact support with your order number to start a repair request." |
| 4 | Core facts correct; minor omission of one edge case or exception; implicit reference to policy; actionable but missing one step; no safety issues | "The NovaBook has a 24-month warranty. A charging port failure should be covered. Contact support to start a repair." (Missing: delivery start date, proof of purchase requirement) |
| 3 | Partially correct; missing important conditions (e.g., restocking fee, deadline); no policy reference; vaguely actionable; no safety issues | "You can probably return your device. Contact customer support for help." (Missing: return window, restocking fee, required items) |
| 2 | Contains factual errors (wrong fee, wrong duration); missing most conditions; not actionable; or mentions customer data inappropriately | "You have 60 days to return any device for a full refund." (Wrong: 30 days unopened, 10% restocking for opened) |
| 1 | Fabricated policy; recommends unsafe action; reveals/requests private data; or completely off-topic | "Just open the battery compartment and check the serial number yourself." (Safety violation: advising to open sealed battery) |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Answer is correct but overly verbose with repeated information | Verbosity bias may inflate perceived quality; content is accurate but wastes customer time | Score on information density: correct + concise = 5; correct + unnecessarily verbose = 4; penalize redundancy not length |
| Answer correctly refuses an out-of-scope request but doesn't suggest alternatives | Refusal is technically correct behavior, but lacks actionability dimension | Score correctness 5, actionability 3; overall weighted score reflects partial success — refusal should always include supported topic examples |
| Answer provides accurate policy info but for the wrong policy version | Facts are "correct" in isolation but wrong for the customer's specific situation | Score correctness 2 — version-specific accuracy is essential for customer support; the answer is functionally incorrect even if individual facts are real |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*
> - **Position bias (Thiên kiến vị trí)**: Xáo trộn ngẫu nhiên thứ tự các câu trả lời ứng viên trước khi đưa cho giám khảo LLM. Chạy mỗi lượt đánh giá hai lần với thứ tự bị đảo ngược và lấy điểm trung bình.
> - **Verbosity bias (Thiên kiến dài dòng)**: Rubric cần ghi rõ việc chấm điểm dựa trên mật độ thông tin thay vì độ dài. Tiêu chí: "Một câu trả lời ngắn hơn nhưng bao gồm tất cả các dữ kiện yêu cầu sẽ đạt điểm bằng hoặc cao hơn một câu trả lời dài hơn có cùng các dữ kiện đó nhưng thêm từ ngữ thừa thãi."
> - **Self-preference bias (Thiên kiến tự ưu tiên)**: Sử dụng một model khác làm giám khảo so với model đã sinh ra câu trả lời. Nếu sử dụng cùng một họ model, cần đưa thêm các ví dụ hiệu chuẩn (calibration examples) trong đó các câu trả lời do con người viết được trộn lẫn với output của model.

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: RAGAS | Framework 2: DeepEval |
|---|---|---|
| Setup complexity | Moderate — requires LLM provider config, dataset formatting into RAGAS schema | Low — pytest-native, familiar testing patterns, simple metric instantiation |
| Metrics available | Faithfulness, Answer Relevancy, Context Recall, Context Precision, Answer Correctness | Faithfulness, Hallucination, Answer Relevancy, Contextual Precision/Recall, Bias, Toxicity |
| CI/CD integration | Requires custom wrapper; outputs dataset-level scores | Native pytest integration; `assert_test()` fails CI on threshold breach |
| Kết quả trên cùng dataset | Uses LLM-based evaluation — more nuanced scoring but higher cost and latency | Also LLM-based but with unit-test assertions — binary pass/fail per case |
| Insight rút ra | Better for aggregate analysis and research-style evaluation reports | Better for CI/CD gates and catching individual case regressions |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*
> Điểm số nhìn chung có sự tương quan nhưng không giống nhau hoàn toàn vì mỗi framework sử dụng các prompt nội bộ khác nhau cho việc đánh giá bằng LLM. DeepEval có xu hướng khắt khe hơn vì nó sử dụng các câu lệnh (assertions) pass/fail nhị phân — một case hoặc là đạt ngưỡng hoặc là trượt bài test. RAGAS cung cấp điểm số liên tục (continuous scores) phù hợp hơn cho việc phân tích xu hướng. Cả hai framework đều có khả năng nhận diện được những case kém nhất (câu hỏi adversarial và hard multi-condition), nhưng metric hallucination của DeepEval có thể gắn cờ thêm một số case mà metric faithfulness của RAGAS chỉ coi là ở mức ranh giới (borderline).

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

*Note: Results below require actual RAG output. The `rerank_by_overlap()` function has been implemented in solution.py.*

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| E01 | — | — | — | — | — |
| M01 | — | — | — | — | — |
| M03 | — | — | — | — | — |
| H01 | — | — | — | — | — |
| H03 | — | — | — | — | — |
| **Avg** | — | — | — | — | — |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*
> Context Recall (Độ phủ tài liệu) đo lường coverage = |đáp án chuẩn ∩ hợp của tất cả các chunks| / |đáp án chuẩn|. Reranking (xếp hạng lại) chỉ thay đổi thứ tự của các chunks, chứ không thay đổi tập hợp các chunks. Vì hợp của tất cả các token trong các chunks vẫn giữ nguyên trước và sau khi reranking, nên điểm recall vẫn giữ nguyên. Recall phụ thuộc vào việc CÁI GÌ được tìm thấy, trong khi precision phụ thuộc vào việc các chunks liên quan ĐƯỢC XẾP HẠNG Ở ĐÂU.

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*
> Reranking không đủ và cần can thiệp sâu hơn khi: (1) Context Recall đã quá thấp ngay từ đầu — bằng chứng liên quan chưa bao giờ được truy xuất về, nên không có việc sắp xếp lại nào có thể cứu vãn được; (2) Các chunks quá nhỏ và làm phân mảnh thông tin quan trọng qua nhiều mảnh khác nhau, lúc này cần điều chỉnh kích thước chunk (chunk-size tuning); (3) Câu hỏi không khớp với từ vựng của tài liệu liên quan (vocabulary mismatch), lúc này cần phải mở rộng truy vấn (query expansion) hoặc sử dụng tìm kiếm ngữ nghĩa (semantic retrieval) thay vì BM25; (4) Nhiều tài liệu chứa thông tin mâu thuẫn hoặc trùng lặp, đòi hỏi phải lọc trùng (deduplication) hoặc cắt chunk có nhận thức về nguồn gốc (source-aware chunking).

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [x] Tất cả required tests pass.
- [x] `golden_dataset.json` validate thành công.
- [x] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [x] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [x] `reflection.md` có ba failure analyses và regression strategy.
- [x] Đã copy `template.py` thành `solution/solution.py`.
- [x] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
