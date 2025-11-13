# Hệ Thống Truy Vấn Thông Tin Các Bài Báo Ngành AI

Hệ thống truy vấn thông tin được xây dựng để tìm kiếm và truy xuất các bài báo khoa học trong lĩnh vực Trí tuệ Nhân tạo (AI). Hệ thống sử dụng phương pháp hybrid kết hợp Word2Vec và LDA để tìm kiếm semantic trên các đoạn văn (chunks) được trích xuất từ các bài báo.

## Tổng Quan

Hệ thống bao gồm 4 thành phần chính:

1. **Crawl dữ liệu** (`crawl.ipynb`): Thu thập các bài báo từ arXiv và chuyển đổi sang định dạng TEI XML
2. **Xây dựng hệ thống** (`system.ipynb`): Xử lý dữ liệu, xây dựng mô hình và lưu trữ vào database
3. **Tạo Ground Truth** (`ground_truth.ipynb`): Tạo dữ liệu đánh giá từ metadata và arXiv
4. **Đánh giá hệ thống** (`evaluation.ipynb`): Đánh giá hiệu suất hệ thống bằng các metrics chuẩn

## Kiến Trúc Hệ Thống

### 1. Thu Thập Dữ Liệu (Crawl)

File `crawl.ipynb` thực hiện:

- **Truy vấn arXiv API**: Tìm kiếm các bài báo theo các chủ đề AI (large language model, transformer, reinforcement learning, computer vision, NLP, multimodal, diffusion model, graph neural network, federated learning, speech)
- **Tải PDF**: Tải các file PDF của bài báo từ arXiv
- **Xử lý PDF với GROBID**: Sử dụng GROBID để chuyển đổi PDF sang định dạng TEI XML
  - Chạy GROBID qua Docker: `docker run --rm -p 18070:8070 -p 18071:8071 lfoppiano/grobid:latest-crf`
  - Sử dụng API của GROBID để xử lý PDF thành TEI XML

### 2. Xây Dựng Hệ Thống (System)

File `system.ipynb` thực hiện:

#### 2.1. Cấu Trúc Database

Hệ thống sử dụng MySQL để lưu trữ dữ liệu với các bảng:

- `papers`: Thông tin cơ bản của bài báo (title, abstract, venue, pub_date, language, publisher)
- `persons`: Thông tin tác giả
- `paper_person`: Liên kết giữa bài báo và tác giả
- `identifiers`: Các định danh (DOI, arXiv ID, MD5, ...)
- `keywords`: Từ khóa
- `paper_keyword`: Liên kết giữa bài báo và từ khóa
- `sections`: Các section của bài báo
- `paper_chunks`: Các đoạn văn (chunks) được tách từ sections

#### 2.2. Xử Lý Dữ Liệu

- **Parse TEI XML**: Trích xuất thông tin từ file TEI XML (title, abstract, authors, keywords, sections, ...)
- **Chunking**: Chia các section thành các chunks có độ dài phù hợp (max_len=1200, min_len=600)
- **Tokenization**: Tokenize các chunks để chuẩn bị cho việc training mô hình

#### 2.3. Xây Dựng Mô Hình

- **Word2Vec**: 
  - Vector size: 200
  - Window: 5
  - Min count: 2
  - Sử dụng Skip-gram (sg=1)
  
- **LDA (Latent Dirichlet Allocation)**:
  - Số topics: 100
  - Passes: 5
  - Alpha: symmetric

#### 2.4. Hybrid Search

Hệ thống sử dụng phương pháp hybrid kết hợp Word2Vec và LDA:

```
score = α * score_w2v + (1 - α) * score_lda
```

Trong đó:
- `score_w2v`: Điểm tương đồng từ Word2Vec (cosine similarity)
- `score_lda`: Điểm tương đồng từ LDA (cosine similarity)
- `α`: Hệ số trọng số (mặc định: 0.5)

### 3. Tạo Ground Truth

File `ground_truth.ipynb` thực hiện quy trình tạo dữ liệu ground truth:

#### 3.1. Tạo 100 Queries từ Metadata

Queries được tạo từ các nguồn khác nhau trong database:

1. **Từ tiêu đề bài báo** (titles): Trích xuất keywords từ các tiêu đề
2. **Từ section headings**: Trích xuất keywords từ các tiêu đề section
3. **Từ keyword combinations**: Kết hợp các keywords phổ biến
4. **Từ chunk content**: Trích xuất keywords từ nội dung các chunks

#### 3.2. Kết Nối và Search trên Web arXiv

- Mỗi query được gửi đến arXiv search engine
- arXiv là nguồn đáng tin cậy cho các bài báo AI, đảm bảo tính khách quan của ground truth

#### 3.3. Lấy 20 Kết Quả Trả Về

- Với mỗi query, hệ thống thu thập top 20 kết quả từ arXiv
- Các kết quả này đại diện cho các bài báo được arXiv đánh giá là liên quan

#### 3.4. Match với Database để Được Data GT

- Các kết quả từ arXiv được match với database local sử dụng các identifiers (DOI, title, authors)
- Với mỗi bài báo được match, các chunks liên quan được xác định và thêm vào ground truth
- Ground truth được lưu dưới dạng CSV và JSON

### 4. Đánh Giá Hệ Thống (Evaluation)

File `evaluation.ipynb` thực hiện đánh giá hệ thống:

#### 4.1. Metrics

Hệ thống được đánh giá bằng các metrics chuẩn:

- **Precision@k (P@k)**: Tỷ lệ các kết quả liên quan trong top k kết quả
- **Recall@k (R@k)**: Tỷ lệ các kết quả liên quan được tìm thấy trong top k
- **F1@k**: Harmonic mean của Precision@k và Recall@k

#### 4.2. So Sánh Các Giá Trị Alpha

Hệ thống so sánh hiệu suất với các giá trị α khác nhau:
- α = 0.0: Chỉ sử dụng LDA
- α = 0.3: Ưu tiên LDA
- α = 0.5: Cân bằng (mặc định)
- α = 0.7: Ưu tiên Word2Vec
- α = 1.0: Chỉ sử dụng Word2Vec

#### 4.3. Visualizations

- Biểu đồ so sánh Precision@k, Recall@k, F1@k theo các giá trị k
- Biểu đồ so sánh metrics theo các giá trị α
- Phân tích metrics theo nguồn query (title, section_head, keyword_combination, chunk_content)

## Cài Đặt

### Yêu Cầu

- Python 3.8+
- MySQL 8.0+
- Docker (để chạy GROBID)

### Cài Đặt Dependencies

```bash
pip install -r requirements.txt
```

### Cấu Hình Database

1. Tạo file `.env` trong thư mục `code/`:

2. Chạy các cells trong `system.ipynb` để tạo database schema

## Sử Dụng

### 1. Thu Thập Dữ Liệu

Chạy các cells trong `crawl.ipynb`:

1. Cấu hình các chủ đề và categories cần crawl
2. Chạy crawl để tải PDF từ arXiv
3. Chạy GROBID để chuyển đổi PDF sang TEI XML

### 2. Xây Dựng Hệ Thống

Chạy các cells trong `system.ipynb` theo thứ tự:

1. **Cell 1-2**: Kết nối database và tạo schema
2. **Cell 3**: Định nghĩa cấu trúc database
3. **Cell 4-5**: Parse TEI XML và import vào database
4. **Cell 6**: Import dữ liệu từ thư mục TEI XML
5. **Cell 7**: Tạo chunks từ sections
6. **Cell 8**: Upload chunks vào database
7. **Cell 9**: Tokenize chunks
8. **Cell 10**: Training Word2Vec model
9. **Cell 11**: Training LDA model
10. **Cell 12**: Định nghĩa hàm search
11. **Cell 13-16**: Test hệ thống search

### 3. Tạo Ground Truth

Chạy các cells trong `ground_truth.ipynb`:

1. **Cell 1-2**: Load models và định nghĩa hàm search
2. **Cell 3**: Tạo 100 queries từ metadata
3. **Cell 4**: Tạo ground truth bằng cách:
   - Search trên arXiv web cho mỗi query
   - Lấy top 20 kết quả
   - Match với database
   - Xác định các chunks liên quan
4. **Cell 5**: Lưu ground truth dưới dạng CSV và JSON

### 4. Đánh Giá Hệ Thống

Chạy các cells trong `evaluation.ipynb`:

1. **Cell 1**: Load models
2. **Cell 2**: Load ground truth
3. **Cell 3**: Định nghĩa các hàm metrics và search
4. **Cell 4**: Định nghĩa các hàm tính toán metrics
5. **Cell 5**: Đánh giá với các giá trị α khác nhau
6. **Cell 6**: Lưu kết quả đánh giá
7. **Cell 7-8**: Tạo visualizations
8. **Cell 9**: Lưu kết quả tốt nhất
9. **Cell 10**: Phân tích chi tiết từng query
11. **Cell 12**: Phân tích metrics theo nguồn query

## Kết Quả

Hệ thống đã được đánh giá với 100 queries và cho kết quả:

- **Tổng số queries**: 100
- **Tổng số chunks liên quan**: ~2,000 (trung bình 20 chunks/query)
- **Metrics**: Precision@10, Recall@10, F1@10 được tính toán và so sánh với các giá trị α khác nhau

Kết quả chi tiết được lưu trong thư mục `evaluation_results/`.

## Tác Giả

Nguyễn Thiên Ân

Hệ thống được phát triển cho bài toán truy vấn thông tin các bài báo ngành AI.

## License
