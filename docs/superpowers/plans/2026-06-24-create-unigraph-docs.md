# Create UniGraph Documentation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Tạo 3 tệp tài liệu kỹ thuật chất lượng cao bằng tiếng Việt trong thư mục `docs/` để làm nguồn dữ liệu chất lượng cho NotebookLM tạo slide thuyết trình.

**Architecture:** Tài liệu chia làm 3 phần độc lập tương ứng với các lớp của hệ thống: (1) Kiến trúc & DB Schema, (2) Scraper & Ingestion, (3) Hybrid Search, Prompt & Thực nghiệm.

**Tech Stack:** Markdown, Git, Mermaid.

## Global Constraints

- Toàn bộ tài liệu viết bằng Tiếng Việt chuẩn chỉnh, mạch lạc.
- Các liên kết tới các lớp/tệp tin mã nguồn phải là link tuyệt đối dạng `[Tên_File](file:///đường_dẫn_tuyệt_đối)`.
- Không sử dụng văn bản giữ chỗ (TODO, TBD, điền sau...).
- Chứa các trích đoạn code và câu Cypher chính xác từ mã nguồn thực tế của dự án.

---

### Task 1: System Overview & Architecture Document

**Files:**
- Create: `docs/SYSTEM_OVERVIEW_AND_ARCHITECTURE.md`

**Interfaces:**
- Consumes: [KHOA_LUAN.md](file:///home/dang-phong/Desktop/MyProject/UniGraph/docs/KHOA_LUAN.md), [Course.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/common/src/main/java/com/uni_graph/common/domain/Course.java), [RequirementRule.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/common/src/main/java/com/uni_graph/common/domain/RequirementRule.java), [V001__initialize_schema.cypher](file:///home/dang-phong/Desktop/MyProject/UniGraph/ingestion/src/main/resources/neo4j/migrations/V001__initialize_schema.cypher), [V002__update_embedding_dimension.cypher](file:///home/dang-phong/Desktop/MyProject/UniGraph/ingestion/src/main/resources/neo4j/migrations/V002__update_embedding_dimension.cypher)
- Produces: Bản mô tả chi tiết kiến trúc tổng thể và Neo4j Database Schema của UniGraph.

- [ ] **Step 1: Viết phần mở đầu, lý do chọn đề tài và mục tiêu**
  Chi tiết hóa sự khác biệt giữa Naive RAG và GraphRAG (vấn đề ảo giác, câu hỏi đa chặng). Trích dẫn mục tiêu nghiên cứu của đồ án.
- [ ] **Step 2: Vẽ sơ đồ kiến trúc hệ thống bằng Mermaid**
  Sử dụng sơ đồ Mermaid chi tiết luồng xử liệu từ scraper đến cơ sở dữ liệu và công cụ tư vấn retrieval.
- [ ] **Step 3: Mô tả chi tiết Graph Schema và mối quan hệ**
  Liệt kê toàn bộ các Node (như Course, Department, RequirementRule...) và các quan hệ (`BELONG_TO`, `REQUIRES`, `SATISFIED_BY`, `EQUIVALENT_TO`, `KNOWLEDGE_PREREQUISITE`). Giải thích ý nghĩa của từng quan hệ trong việc mô hình hóa cây logic AND/OR và các quan hệ kiến thức ẩn.
- [ ] **Step 4: Liệt kê các lệnh thiết lập Constraints và Indexes**
  Chỉ ra các câu lệnh Cypher thực tế được định nghĩa trong các tệp di cư (migrations) để tạo Unique Constraints, Indexes tìm kiếm theo tên và Vector Index 768 chiều với hàm cosine.
- [ ] **Step 5: Xác minh và Commit**
  Đảm bảo định dạng markdown chuẩn, link file hoạt động tốt.
  Chạy lệnh: `git add docs/SYSTEM_OVERVIEW_AND_ARCHITECTURE.md`
  Chạy lệnh: `git commit -m "docs: add system overview and architecture document"`

---

### Task 2: Data Pipeline & Ingestion Document

**Files:**
- Create: `docs/DATA_PIPELINE_AND_INGESTION.md`

**Interfaces:**
- Consumes: [thuthaplink.py](file:///home/dang-phong/Desktop/MyProject/UniGraph/crawl/thuthaplink.py), [monhocScrape.py](file:///home/dang-phong/Desktop/MyProject/UniGraph/crawl/monhocScrape.py), [tomtatmonhocScrape.py](file:///home/dang-phong/Desktop/MyProject/UniGraph/crawl/tomtatmonhocScrape.py), [CsvIngestionServiceImpl.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/ingestion/src/main/java/com/uni_graph/ingestion/service/impl/CsvIngestionServiceImpl.java), [LangChainEmbeddingServiceImpl.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/ingestion/src/main/java/com/uni_graph/ingestion/service/impl/LangChainEmbeddingServiceImpl.java)
- Produces: Bản mô tả chi tiết cơ chế thu thập dữ liệu và quy trình nạp dữ liệu đồ thị tri thức (Ingestion).

- [ ] **Step 1: Tài liệu hóa Module Crawl (Python)**
  Trình bày các script cào web và tệp dữ liệu CSV đầu ra (`subjects.csv`, `subject_summary.csv`). Điểm nhấn: giải thích **kỹ thuật tiền xử lý thẻ `<br>` thay bằng dấu xuống dòng `\n`** để phục vụ việc trích xuất cây logic môn tiên quyết sau này.
- [ ] **Step 2: Tài liệu hóa Ingestion - Pass 1: Tạo các Node Cơ bản**
  Chi tiết hóa logic đọc CSV, tạo các node Course và Department trong [CsvIngestionServiceImpl.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/ingestion/src/main/java/com/uni_graph/ingestion/service/impl/CsvIngestionServiceImpl.java). Giải thích cách sinh vector embedding 768 chiều từ định dạng văn bản giàu ngữ nghĩa qua [LangChainEmbeddingServiceImpl.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/ingestion/src/main/java/com/uni_graph/ingestion/service/impl/LangChainEmbeddingServiceImpl.java).
- [ ] **Step 3: Tài liệu hóa Ingestion - Pass 2: Tạo các quan hệ tường minh**
  Chi tiết hóa logic xử lý môn tương đương (`EQUIVALENT_TO`), môn tiên quyết/học trước bằng cách tạo các Node trung gian `RequirementRule` (với `LogicType.AND`/`LogicType.OR` và `RuleType`).
- [ ] **Step 4: Tài liệu hóa Ingestion - Pass 3: Nạp Tóm tắt & Trích xuất mối quan hệ ẩn bằng LLM**
  Mô tả chi tiết quá trình re-embedding khi có tóm tắt môn học. Trích dẫn prompt và giải thích cách thức LLM tự động trích xuất các mã môn liên quan làm kiến thức nền tảng (`KNOWLEDGE_PREREQUISITE`) từ nội dung tóm tắt.
- [ ] **Step 5: Xác minh và Commit**
  Đảm bảo định dạng markdown chuẩn, link file hoạt động tốt.
  Chạy lệnh: `git add docs/DATA_PIPELINE_AND_INGESTION.md`
  Chạy lệnh: `git commit -m "docs: add data pipeline and ingestion document"`

---

### Task 3: Retrieval, Reasoning & Evaluation Document

**Files:**
- Create: `docs/RETRIEVAL_AND_REASONING.md`

**Interfaces:**
- Consumes: [RetrievalController.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/retrieval/src/main/java/com/uni_graph/retrieval/controllers/RetrievalController.java), [SearchServiceImpl.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/retrieval/src/main/java/com/uni_graph/retrieval/service/impl/SearchServiceImpl.java), [CypherGeneratorImpl.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/retrieval/src/main/java/com/uni_graph/retrieval/service/impl/CypherGeneratorImpl.java), [ChatServiceImpl.java](file:///home/dang-phong/Desktop/MyProject/UniGraph/retrieval/src/main/java/com/uni_graph/retrieval/service/impl/ChatServiceImpl.java), [generate_eval_dataset.py](file:///home/dang-phong/Desktop/MyProject/UniGraph/crawl/generate_eval_dataset.py), [run_evaluation_resumable.py](file:///home/dang-phong/Desktop/MyProject/UniGraph/crawl/run_evaluation_resumable.py)
- Produces: Bản mô tả chi tiết module truy vấn, thuật toán Hybrid Search, prompt sinh câu trả lời và kết quả đánh giá thực nghiệm.

- [ ] **Step 1: Tài liệu hóa REST APIs & Thuật toán Hybrid Search**
  Trình bày các API `/api/v1/search` và `/api/v1/chat`. Giải thích thuật toán Hybrid Search 3 bước:
  - Sinh Cypher bằng LLM (trích dẫn luật nghiêm ngặt và System Prompt).
  - Hydration & Context Enrichment: Truy vấn đồ thị lấy thêm tối đa 5 môn lân cận (1-hop) làm giàu context cho LLM (giới hạn 25 node để tối ưu token).
  - Fallback: Vector search (Top 5) + Keyword search.
- [ ] **Step 2: Tài liệu hóa Chat Service Logic & Prompt Engineering**
  Trình bày cách thức cấu trúc hóa thông tin môn học đã được làm giàu ngữ cảnh thành context và cấu trúc System Prompt hướng dẫn LLM sinh câu trả lời tư vấn lộ trình học tập, cấm bịa đặt thông tin.
- [ ] **Step 3: Tài liệu hóa Thiết kế Thực nghiệm & Đánh giá**
  Mô tả bộ dữ liệu 35 câu hỏi phân chia theo 4 cấp độ (Simple, Medium, Hard, Super Hard). Giải thích cách thức sinh câu hỏi tự động dựa trên cấu trúc cây tiên quyết và tóm tắt.
- [ ] **Step 4: Tài liệu hóa Kết quả Thực nghiệm**
  Đưa ra kết quả đánh giá so sánh GraphRAG vs Naive RAG. Nêu rõ ưu điểm cải thiện độ chính xác đa chặng (Multi-hop) lên ~30% và nhược điểm về thời gian phản hồi (latency).
- [ ] **Step 5: Xác minh và Commit**
  Đảm bảo định dạng markdown chuẩn, link file hoạt động tốt.
  Chạy lệnh: `git add docs/RETRIEVAL_AND_REASONING.md`
  Chạy lệnh: `git commit -m "docs: add retrieval and reasoning document"`
