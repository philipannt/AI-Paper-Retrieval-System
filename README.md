# Search Engine for Academic Papers

Hệ thống tìm kiếm học thuật cho các bài báo khoa học sử dụng phương pháp hybrid kết hợp Word2Vec và LDA (Latent Dirichlet Allocation).

## Tổng quan

Dự án này xây dựng một hệ thống tìm kiếm semantic cho các bài báo khoa học từ arXiv, bao gồm:
- **Crawl dữ liệu**: Thu thập papers từ arXiv API
- **Xử lý dữ liệu**: Parse TEI XML và lưu trữ vào MySQL
- **Mô hình tìm kiếm**: Hybrid approach kết hợp Word2Vec và LDA
- **Đánh giá**: Đo lường hiệu suất với ground truth queries

## Cấu trúc thư mục

```
Search-Engine/
├── code/                    # Source code chính
│   ├── crawl.ipynb         # Crawl papers từ arXiv
│   ├── system.ipynb        # Xây dựng hệ thống search engine
│   ├── evaluation.ipynb    # Đánh giá hiệu suất
│   ├── preprocessing.ipynb # Tiền xử lý dữ liệu
│   ├── BM25.ipynb          # Implementation BM25
│   ├── M3-Emb.ipynb        # M3 Embedding experiments
│   ├── ground_truth/       # Ground truth queries và labels
│   ├── models_hybrid_lda_w2v/  # Trained models
│   └── requirements.txt    # Dependencies
├── data-full/              # TEI XML files đã xử lý
└── README.md
```

## Cài đặt

1. Cài đặt dependencies:
```bash
pip install -r code/requirements.txt
```

2. Cấu hình MySQL database:
   - Tạo file `.env` trong thư mục `code/` với thông tin kết nối MySQL
   - Format:
     ```
     MYSQL_HOST=127.0.0.1
     MYSQL_PORT=3306
     MYSQL_USER=root
     MYSQL_PASSWORD=your_password
     MYSQL_DB=ai_papers
     MYSQL_CHARSET=utf8mb4
     ```

3. (Tùy chọn) Chạy GROBID server để xử lý PDF:
```bash
docker run --rm -p 18070:8070 -p 18071:8071 lfoppiano/grobid:latest-crf
```

## Sử dụng

### 1. Crawl papers từ arXiv
Chạy notebook `crawl.ipynb` để thu thập papers theo topics và categories đã định nghĩa.

### 2. Xây dựng hệ thống search
Chạy notebook `system.ipynb` để:
- Parse TEI XML files và lưu vào database
- Chunk documents thành các đoạn văn
- Train Word2Vec và LDA models
- Tạo vector embeddings cho các chunks

### 3. Đánh giá hệ thống
Chạy notebook `evaluation.ipynb` để:
- Đánh giá với ground truth queries
- So sánh các giá trị alpha khác nhau
- Tạo visualizations và báo cáo

## Công nghệ sử dụng

- **NLP/ML**: Gensim (Word2Vec, LDA), scikit-learn
- **Database**: MySQL, SQLAlchemy
- **Data Processing**: pandas, numpy
- **Visualization**: matplotlib, seaborn
- **Web Scraping**: requests, feedparser
- **XML Processing**: lxml

## Phương pháp tìm kiếm

Hệ thống sử dụng **hybrid search** kết hợp:
- **Word2Vec**: Semantic similarity dựa trên word embeddings
- **LDA**: Topic modeling để nắm bắt semantic themes

Score cuối cùng: `score = α × score_w2v + (1-α) × score_lda`

## Kết quả

Hệ thống đã được đánh giá với 100 queries và cho thấy hiệu suất tốt với α = 0.5 (cân bằng giữa Word2Vec và LDA).

## License

Academic use only.

