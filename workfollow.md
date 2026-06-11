# Work Follow

Tài liệu này là playbook để đọc, phân tích, và làm việc với repo `retrieval-backend-with-rag` một cách hiệu quả trong các session sau.

Mục tiêu:
- nạp context nhanh nhưng đúng
- dùng đúng nguồn sự thật cho đúng loại câu hỏi
- tránh đọc file thủ công lan man
- khóa yêu cầu trước khi viết code
- tạo Spec gọn, có test và tiêu chí xác minh rõ ràng

## Skill Pack Mặc Định

Đọc skills từ WSL/Linux path:
- `/home/hieu0606sunny/.codex/skills/brainstorming/SKILL.md`
- `/home/hieu0606sunny/.codex/skills/rich-elicitation/SKILL.md`
- `/home/hieu0606sunny/.codex/skills/plan-writing/SKILL.md`
- `/home/hieu0606sunny/.codex/skills/idea-os/SKILL.md`

Chọn skill như sau:
- `@brainstorming`: default cho mọi task mơ hồ, feature, refactor, design, architecture, hoặc khi cần làm rõ intent trước khi code
- `@rich-elicitation`: chỉ dùng khi có từ 2 chiều mơ hồ lớn trở lên và mỗi chiều có nhiều phương án hợp lý
- `@plan-writing`: dùng sau khi Understanding Lock đã được user xác nhận, để tạo Spec/task plan trong repo hiện có
- `@idea-os`: chỉ dùng khi đây là ý tưởng sản phẩm mới/lớn, cần PRD, research, GTM, phased plan

Trong repo này, mặc định ưu tiên:
- `@brainstorming`
- `@plan-writing`

Không dùng `@idea-os` cho bugfix, refactor, feature nhỏ, hoặc task phân tích codebase hiện tại.

## Nguồn Context Và Mức Độ Tin Cậy

Thứ tự ưu tiên khi nạp context:

1. `AGENTS.md`
2. `AGENT.md`
3. CodeGraph
4. Understand-Anything
5. Các file trạng thái/task nếu user cung cấp thêm

### AGENTS.md

Đây là behavioral contract của repo:
- giao tiếp tiếng Việt
- dùng `uv` cho Python commands
- chia nhỏ thay đổi
- validate incrementally
- không đoán mò khi debug

### AGENT.md

Đây là context file đã được tổng hợp từ session trước:
- tech stack
- entry points
- main layers
- request flow
- known risks
- graph caveats

Đọc file này đầu tiên để tránh khởi động lại từ đầu.

### CodeGraph

Đây là source of truth chính cho:
- symbol
- call flow
- impact analysis
- current structure của code

Trong repo này, CodeGraph hiện usable và đáng tin hơn Understand-Anything cho current code behavior.

### Understand-Anything

Dùng cho:
- onboarding
- project layers
- tour đọc file
- business/domain framing mức cao

Không dùng làm source of truth duy nhất cho current behavior nếu `gitCommitHash` trong graph không khớp `git rev-parse HEAD`.

Hiện trạng đã biết:
- `knowledge-graph.json` tồn tại
- `meta.json` tồn tại
- `config.json` tồn tại
- `domain-graph.json` không có
- hash trong graph là `non-git-worktree`, không khớp HEAD đã phân tích trong `AGENT.md`

Kết luận:
- vẫn đọc `project`, `layers`, `tour`
- nhưng không dựa vào nó để kết luận logic hiện tại nếu CodeGraph cho thông tin khác

### Các File Trạng Thái Khác

Nếu tồn tại, phải đọc trước khi brainstorm:
- `PROJECT_STATE.md`
- `TASK_SPEC.md`
- `PROGRESS_LOG.md`
- `DECISIONS.md`
- các file report liên quan

Hiện tại repo chỉ có:
- `AGENT.md`

## Snapshot Hiện Tại Của Repo

### Tech Stack

- Python 3.12
- Flask API
- `flask-cors`
- `pydantic`
- `chromadb`
- `qdrant-client`
- `pymongo`
- `sentence-transformers`
- `transformers`
- `onnxruntime`
- `openai`
- `google-generativeai`
- `mistralai`
- evaluation stack gồm `ragas`, `sacrebleu`

### Entry Points

- main API entry: `serve.py`
- HTTP route chính: `POST /api/search`
- benchmark:
  - `benchmark_inference_time_llms.py`
  - `benchmark_inference_time_pipeline.py`
- tests:
  - `test/unitTest/`
  - `test/integrationTest/`

### Main Layers

1. API & orchestration
   - `serve.py`
   - `semantic_router/`
2. Reflection + retrieval + rerank
   - `reflection/`
   - `rag/`
   - `re_rank/`
3. LLM adapters
   - `llms/`
4. Embeddings
   - `embeddings/`
5. Data ingestion
   - `insert_data/`
   - `hoanghamobile.csv`
6. Evaluation
   - benchmarks và tests

## Luồng Đọc Repo Hiệu Quả

Không đọc toàn bộ repo theo thứ tự alphabet. Đọc theo flow runtime.

### Pass 1: Khóa bức tranh lớn

1. Đọc `AGENTS.md`
2. Đọc `AGENT.md`
3. Kiểm tra CodeGraph status
4. Nếu cần onboarding nhanh, đọc `project`, `layers`, `tour` trong `.understand-anything/knowledge-graph.json`

Mục tiêu của pass này:
- biết repo làm gì
- biết entry point ở đâu
- biết module nào đáng tin để bắt đầu
- biết graph nào đang stale

### Pass 2: Đọc theo request flow

Thứ tự đọc khuyến nghị:

1. `serve.py`
2. `semantic_router/router.py`
3. `semantic_router/route.py`
4. `semantic_router/samples.py`
5. `reflection/core.py`
6. `rag/core.py`
7. `re_rank/core.py`
8. `llms/llms.py`
9. `llms/onlinesLlms.py`
10. `llms/localLlms.py`
11. `embeddings/sentenceTransformer.py`
12. `insert_data/build_chromadb.py`

Mục tiêu:
- hiểu request đi từ đâu tới đâu
- biết điểm nào route sang RAG, điểm nào đi thẳng LLM
- biết vector store nào được hỗ trợ
- biết data được ingest thế nào

### Pass 3: Đọc theo validation flow

Sau khi hiểu runtime flow, mới đọc:

1. `test/unitTest/test_api_response.py`
2. `test/unitTest/test_chromadb_vector_search.py`
3. `test/unitTest/test_reflection.py`
4. `test/unitTest/test_rerank_ncdg.py`
5. `test/unitTest/test_hit@k_mongodb.py`
6. `test/unitTest/test_hit@k_qdrant.py`
7. `test/unitTest/test_huggingface_backend.py`
8. `test/unitTest/test_onnx_backend.py`
9. `benchmark_inference_time_pipeline.py`
10. `benchmark_inference_time_llms.py`

Mục tiêu:
- biết repo chứng minh chất lượng hiện tại bằng gì
- biết test nào nên chạy đầu tiên khi sửa một vùng code
- biết đâu là benchmark script thay vì functional tests

## Luồng Runtime Cốt Lõi

Luồng hiện tại của `POST /api/search`:

1. client gửi một mảng chat messages
2. `handle_query()` đọc JSON body
3. `Reflection` rewrite câu hỏi thành standalone Vietnamese query
4. `SemanticRouter.guide()` phân loại `products` hoặc `chitchat`
5. nếu là `products`
   - `rag.vector_search(query)`
   - `Reranker(query, passages)`
   - ghép prompt tư vấn bán hàng
   - `rag.generate_content(data)`
6. nếu là `chitchat`
   - gọi thẳng `llm.generate_content(data)`
7. trả về:
   - `{"content": response, "role": "assistant"}`

## Luồng Ingestion

Khi chạy `serve.py` với `--db chromadb`:

1. kiểm tra collection Chroma đã tồn tại chưa
2. nếu chưa có, quét thư mục `data/` tìm CSV
3. gọi `load_csv_to_chromadb(...)`
4. tạo `combined_information`
5. embed từng row
6. insert vào Chroma persistent collection

Nếu task liên quan retrieval quality hoặc missing results, cần đọc cả:
- `rag/core.py`
- `insert_data/build_chromadb.py`
- test retrieval tương ứng

## Cách Dùng CodeGraph Trong Repo Này

### Khi nào dùng gì

- `codegraph_status`
  - kiểm tra graph usable không
- `codegraph_files`
  - lấy cây thư mục đã index
- `codegraph_context`
  - khi cần map nhanh kiến trúc hoặc flow của một task
- `codegraph_search`
  - tìm symbol
- `codegraph_node`
  - lấy body của một symbol cụ thể
- `codegraph_trace`
  - lần theo flow từ entry point đến destination
- `codegraph_impact`
  - dùng sau khi sửa code để xem bán kính ảnh hưởng
- `codegraph_explore`
  - dùng khi cần đọc source của một nhóm symbol liên quan

### Quy tắc thực dụng

- hỏi về architecture hoặc flow: bắt đầu bằng `codegraph_context`
- hỏi về một hàm/class cụ thể: bắt đầu bằng `codegraph_search` rồi `codegraph_node`
- hỏi “từ request đến response đi qua đâu”: dùng `codegraph_trace`
- sau khi sửa code nhiều module: dùng `codegraph_impact` trước khi đọc tay thêm

### Không nên làm

- không quét cả repo bằng `sed` hoặc `cat` trước khi dùng CodeGraph
- không đọc hàng loạt file nếu symbol-level context là đủ
- không tin Understand-Anything hơn CodeGraph cho logic hiện tại

## Cách Dùng Understand-Anything Trong Repo Này

### Dùng cho

- onboarding
- xác định layers
- tour đọc file
- business framing mức cao

### Đọc phần nào

- `project`
- `layers`
- `tour`
- node/component liên quan nếu thật sự cần

### Không làm

- không chạy lại `/understand` chỉ vì muốn refresh nhẹ
- không coi graph này là current-truth nếu hash vẫn mismatch

## Known Hotspots Cần Nhớ

Đây là các vùng đáng chú ý đã xác nhận trong context trước:

1. `rag/core.py`
   - `_collection_exists` đang có dấu hiệu bị dùng như property thay vì method
2. `insert_data/build_chromadb.py`
   - có dấu hiệu typo giữa `combined_infomation` và `combined_information`
3. `serve.py`
   - `Reflection` nằm trên hot path của mọi request
   - `process_query()` có vẻ không được dùng
   - Flask đang chạy `debug=True`
4. docs/manifests
   - `pyproject.toml` khai báo `README.md`, nhưng file thực tế là `README_old.md`

Khi nhận task liên quan:
- retrieval bug
- startup behavior
- latency
- ingestion mismatch
- packaging/docs inconsistency

thì các hotspot trên là nơi đọc đầu tiên.

## Quy Trình Làm Việc Cho Mỗi Task

### Phase 1: Reload Context

Luôn bắt đầu bằng:

1. đọc `AGENTS.md`
2. đọc `AGENT.md`
3. kiểm tra CodeGraph status
4. nếu thiếu context nghiệp vụ, đọc `.understand-anything/knowledge-graph.json`
5. nếu có thêm file trạng thái/task/report, đọc hết trước khi hỏi

Sau đó phải tóm tắt ngắn 4 mục:
- trạng thái hiện tại
- nhiệm vụ/tính năng đang nhắm tới
- các ràng buộc đã biết
- những điểm còn thiếu thông tin

### Phase 2: Brainstorming

Dùng `@brainstorming` làm khung chính.

Mục tiêu là làm rõ:
- user/audience là ai
- mục tiêu sản phẩm/tính năng là gì
- MVP scope là gì
- non-goals là gì
- constraints là gì
- success criteria là gì
- người học sẽ học được gì từ task này
- outcome portfolio hoặc running product cần đạt là gì

Quy tắc:
- không viết code
- không nhảy sang implementation
- không chốt scope bằng silent assumptions nếu assumption đó có risk

### Phase 3: Rich Elicitation Khi Cần

Chỉ bật `@rich-elicitation` nếu:
- có ít nhất 2 chiều mơ hồ lớn
- mỗi chiều có nhiều phương án hợp lý
- câu trả lời sẽ đổi scope, design, test plan, hoặc implementation plan

Quy tắc hỏi:
- tối đa 3 câu mỗi lượt
- ưu tiên multiple-choice có recommended option
- không hỏi lan man
- sau tối đa 3 rounds thì nêu assumptions còn lại và proceed

### Phase 4: Understanding Lock

Trước khi tạo Spec hoặc viết code, phải khóa lại understanding bằng một summary có các mục:

- What is being built
- Why it exists
- Who it is for
- MVP scope
- Non-goals
- Assumptions
- Open questions
- Skill pack đã chọn và vì sao chọn
- Learning / portfolio / production criteria nếu có

Sau đó phải dừng và chờ user xác nhận hoặc chỉnh sửa.

Không được tạo Spec trước bước này.

### Phase 5: Tạo Spec

Chỉ làm sau khi user đã confirm Understanding Lock.

Chọn skill:
- task trong repo hiện có: `@plan-writing`
- ý tưởng sản phẩm mới/lớn: `@idea-os`

Spec phải có:
- Goal
- Requirements
- Non-goals
- Technical approach
- Affected files/modules dự kiến
- Acceptance criteria
- Unit tests / integration tests cần có
- Evaluation plan nếu liên quan LLM/ML/Agent
- Experiment plan nếu cần so sánh phương án
- Risks
- Report outline để học lại sau khi hoàn thành

Tên file:
- lưu ở root repo
- format: `{task-slug}-spec.md`
- ví dụ: `retrieval-latency-spec.md`

## Template Tóm Tắt Sau Khi Reload Context

Dùng mẫu ngắn này trước khi brainstorm:

```md
Current state:
- ...

Target task:
- ...

Known constraints:
- ...

Missing information:
- ...
```

## Template Understanding Lock

```md
Understanding Lock

- What is being built: ...
- Why it exists: ...
- Who it is for: ...
- MVP scope: ...
- Non-goals: ...
- Assumptions: ...
- Open questions: ...
- Skill pack: ...
- Learning / portfolio / production criteria: ...
```

Sau block này phải chờ user confirm.

## Template Spec Cho Repo Hiện Có

```md
# <task-name>

## Goal
...

## Requirements
- ...

## Non-goals
- ...

## Technical Approach
- ...

## Affected Files / Modules
- ...

## Acceptance Criteria
- ...

## Tests
- Unit tests: ...
- Integration tests: ...

## Evaluation Plan
- ...

## Experiment Plan
- ...

## Risks
- ...

## Report Outline
- ...
```

## Quy Tắc Đọc Và Hỏi Hiệu Quả

- đọc flow trước, đọc implementation detail sau
- đọc entry point trước dependency leaf
- dùng CodeGraph trước file scanning thủ công
- đọc test sau khi đã hiểu runtime flow
- nếu context còn thiếu, đọc graph trước khi hỏi
- nếu hỏi user, hỏi câu làm thay đổi quyết định kỹ thuật
- không viết code trong pha brainstorming
- không tạo Spec khi chưa có Understanding Lock confirmed

## Khi Nào Nên Cập Nhật `AGENT.md`

Cập nhật `AGENT.md` khi có một trong các thay đổi sau:
- kiến trúc thay đổi đáng kể
- entry point mới
- request flow đổi
- thêm module lớn
- thay đổi source of truth hoặc graph health
- phát hiện hotspot/risk mới có tính hệ thống

Không cần cập nhật `AGENT.md` cho thay đổi nhỏ, cục bộ, hoặc bugfix hẹp nếu không làm đổi mental model của repo.

## Mental Model Ngắn Gọn

Repo này là một Flask backend nhỏ cho Vietnamese RAG product Q&A.

Luồng chính là:
- rewrite câu hỏi từ chat history
- semantic route
- nếu là product question thì retrieve + rerank + generate
- nếu là chitchat thì generate trực tiếp

Vì vậy khi phân tích bất kỳ task nào, luôn tự hỏi:
- task này đụng vào route decision?
- task này đụng vào reflection?
- task này đụng vào retrieval?
- task này đụng vào rerank?
- task này đụng vào LLM adapter?
- task này cần test ở unit, integration, hay benchmark level?
