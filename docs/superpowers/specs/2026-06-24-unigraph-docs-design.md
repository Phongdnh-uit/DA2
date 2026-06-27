# Spec Thiết kế: Tài liệu Tóm tắt Dự án UniGraph phục vụ Slide Thuyết trình

**Ngày thực hiện:** 2026-06-24  
**Tác giả:** Antigravity  
**Mục tiêu:** Xây dựng bộ tài liệu tóm tắt kỹ thuật chất lượng cao cho dự án UniGraph trong thư mục `docs/`. Bộ tài liệu này được cấu trúc hóa để mô hình ngôn ngữ như NotebookLM có thể đọc, hiểu sâu sắc kiến trúc, mã nguồn và kết quả thực nghiệm của hệ thống để làm slide thuyết trình chính xác nhất.

---

## 1. Các tài liệu đầu ra (Deliverables)

Hệ thống sẽ được đúc kết vào 3 tệp tài liệu chính nằm trong thư mục `docs/`:

1. **`docs/SYSTEM_OVERVIEW_AND_ARCHITECTURE.md`**
   - *Mục tiêu:* Giới thiệu tổng quan đề tài, cơ sở lý thuyết, kiến trúc tổng thể của hệ thống và thiết kế chi tiết cơ sở dữ liệu đồ thị Neo4j.
   - *Nội dung chính:* Lý do chọn đề tài (Naive RAG vs GraphRAG); Sơ đồ kiến trúc Mermaid; Schema đồ thị chi tiết (Nodes, Relationships); Các ràng buộc (Constraints) và Chỉ mục tìm kiếm (Indexes) trong Neo4j.

2. **`docs/DATA_PIPELINE_AND_INGESTION.md`**
   - *Mục tiêu:* Tài liệu hóa quy trình thu thập dữ liệu (cào web) và nhập liệu đồ thị tri thức (Ingestion).
   - *Nội dung chính:* Các script scraper bằng Python (xử lý thẻ `<br>` cho điều kiện tiên quyết); Quy trình nạp Course/Department từ CSV bằng Java Spring Boot + LangChain4j; Cơ chế tạo vector embeddings (768 chiều, `embeddinggemma`); Quy trình trích xuất mối quan hệ kiến thức nền tảng ẩn `KNOWLEDGE_PREREQUISITE` từ mô tả môn học bằng LLM.

3. **`docs/RETRIEVAL_AND_REASONING.md`**
   - *Mục tiêu:* Tài liệu hóa module truy vấn, cơ chế Hybrid Search, prompt sinh câu trả lời và quy trình đánh giá thực nghiệm.
   - *Nội dung chính:* REST API endpoints; Thuật toán Hybrid Search (Sinh Cypher bằng LLM -> Thực thi -> Hydration & Context Enrichment lấy các node lân cận -> Fallback Vector/Keyword search); Prompt Engineering cho tư vấn học tập; Dataset đánh giá (35 câu hỏi, 4 mức độ); Quy trình đánh giá tự động và kết quả so sánh hiệu năng.

---

## 2. Thiết kế chi tiết từng tài liệu

### 2.1. SYSTEM_OVERVIEW_AND_ARCHITECTURE.md
* **Đề tài:** Xây dựng và khai thác Biểu đồ tri thức (Knowledge Graph) cho hệ thống Retrieval-Augmented Generation (RAG) trong lĩnh vực học thuật.
* **Lý do chọn đề tài:** Naive RAG thường gặp hiện tượng ảo giác (hallucination) và không có khả năng suy luận đa chặng (multi-hop). GraphRAG giải quyết bằng cách biểu diễn tri thức dưới dạng thực thể và quan hệ logic chặt chẽ.
* **Kiến trúc hệ thống:** 
  ```mermaid
  graph TD
      Crawl[Module Python Crawler] -->|subjects.csv / summaries.csv| Ingest[Module Ingestion Spring Boot]
      Ingest -->|Embeddings / LLM Extraction| Neo4j[(Neo4j Graph Database)]
      User[Người dùng] -->|Câu hỏi| UI[React Frontend]
      UI -->|Query| Retrieval[Module Retrieval Spring Boot]
      Retrieval -->|1. Generate Cypher| LLM[LLM Ollama / Groq]
      Retrieval -->|2. Run Cypher / Fallback Vector| Neo4j
      Retrieval -->|3. Enrich Context| Neo4j
      Retrieval -->|4. Generate Response| LLM
      Retrieval -->|Trả lời| UI
  ```
* **Neo4j Graph Schema:**
  - Nodes: `Course`, `Department`, `RequirementRule`, `Section`, `Teacher`, `Classroom`, `TimeSlot`, `Semester`, `Student`, `Group`.
  - Relationships chính: 
    - `BELONG_TO` (Course -> Department)
    - `REQUIRES` (Course -> RequirementRule)
    - `SATISFIED_BY` (RequirementRule -> Course hoặc RequirementRule: Phục vụ logic cây điều kiện AND/OR/NOT)
    - `EQUIVALENT_TO` (Course -> Course)
    - `KNOWLEDGE_PREREQUISITE` (Course -> Course - Trích xuất từ tóm tắt bằng LLM)

### 2.2. DATA_PIPELINE_AND_INGESTION.md
* **Scraper:** 
  - `thuthaplink.py`: Thu thập link chương trình đào tạo các khóa >= 2023 từ studentUIT.
  - `monhocScrape.py`: Bóc tách bảng danh mục môn học. **Kỹ thuật quan trọng:** thay thế thẻ `<br>` bằng `\n` trong các ô dữ liệu môn tiên quyết/môn học trước nhằm giữ nguyên cấu trúc phân dòng giúp thuật toán ánh xạ điều kiện logic chính xác.
  - `tomtatmonhocScrape.py`: Thu thập tóm tắt nội dung đề cương môn học.
* **Ingestion (Spring Boot):**
  - **Pass 1:** Tạo node `Course`, `Department`. Gọi `OllamaEmbeddingModel` tạo vector embeddings 768 chiều (mô hình `embeddinggemma`) dựa trên văn bản định dạng chi tiết môn học.
  - **Pass 2:** Đọc dữ liệu môn tương đương và tiên quyết. Phân tích các mã môn, tạo node trung gian `RequirementRule` với `LogicType` (AND/OR) và `RuleType` (PREREQUISITE/PREVIOUS).
  - **Pass 3:** Đọc CSV tóm tắt môn học. Cập nhật `summary` cho Course, sinh lại embeddings tích hợp summary. Gọi LLM (`llama3-70b-8192` qua Groq) để phân tích tóm tắt môn học, tự động tìm và trích xuất các mã môn học nền tảng cần thiết trong danh sách để tạo liên kết `KNOWLEDGE_PREREQUISITE`.

### 2.3. RETRIEVAL_AND_REASONING.md
* **Hybrid Search Algorithm:**
  - Nhận câu hỏi người dùng.
  - **Bước 1: Cypher Search:** LLM nhận Schema đồ thị và câu hỏi để dịch sang câu lệnh Cypher (Rule: Trả về full node Course, trả về cả node đầu và node cuối).
  - **Bước 2: Hydration & Context Enrichment:** Lấy kết quả từ Cypher hoặc RegEx mã môn, nạp thông tin đầy đủ của Course từ Neo4j. Với mỗi môn học tìm thấy, thực hiện truy vấn đồ thị lấy thêm tối đa 5 môn lân cận (tương đương, tiên quyết, học trước, kiến thức nền tảng cả 2 chiều) để đưa thêm vào ngữ cảnh (giới hạn tối đa 25 node lân cận cho toàn bộ context để tránh bùng nổ token).
  - **Bước 3: Fallback:** Nếu Cypher không hoạt động hoặc không có kết quả, thực hiện Vector Search (Top 5 cosine similarity bằng embedding của câu hỏi) + Keyword Search (tìm kiếm văn bản).
* **Chat Service Prompt:** Hướng dẫn LLM làm nhiệm vụ tư vấn học tập. Context truyền vào được cấu trúc hóa rõ ràng từ thông tin môn học đã làm giàu ngữ cảnh. Yêu cầu LLM suy luận trên quan hệ đồ thị của các node thực tế và tuyệt đối không tự tạo thông tin môn học.
* **Evaluation:**
  - Bộ câu hỏi 35 câu với 4 cấp độ (Simple, Medium, Hard, Super Hard).
  - So sánh thực nghiệm giữa GraphRAG và Naive RAG: GraphRAG có ưu thế vượt trội ở các câu hỏi đa chặng (Multi-hop) như lộ trình học tập, tìm môn học liên quan, đạt độ chính xác cao hơn khoảng 30%. Tuy nhiên, điểm hạn chế là thời gian xử lý (latency) lớn hơn do thời gian sinh Cypher bằng LLM.

---

## 3. Kế hoạch thực hiện (Execution Plan)

1. **Khởi tạo Spec:** Tạo tệp Spec thiết kế này và trình người dùng xem duyệt. (Đang thực hiện)
2. **Triển khai viết 3 tài liệu:** 
   - Tạo file và viết nội dung cho `docs/SYSTEM_OVERVIEW_AND_ARCHITECTURE.md`.
   - Tạo file và viết nội dung cho `docs/DATA_PIPELINE_AND_INGESTION.md`.
   - Tạo file và viết nội dung cho `docs/RETRIEVAL_AND_REASONING.md`.
3. **Kiểm tra tự đánh giá (Self-review):** Rà soát các tệp tài liệu để đảm bảo không có phần bỏ trống (TBD/TODO), định dạng Markdown trực quan, các đường link file là chính xác và nội dung logic đồng nhất.
4. **Báo cáo kết quả:** Gửi liên kết các tệp tài liệu cho người dùng duyệt và kết thúc phiên làm việc.
