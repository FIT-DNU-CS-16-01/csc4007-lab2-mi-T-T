# Lab 2 Analysis Report (IMDB)

## 1. Data audit
- Số lượng mẫu đã dùng: 50,000 mẫu
- Phân bố nhãn positive / negative: 25,000 negative / 25,000 positive
- Độ dài review điển hình (median, p95): median 173, p95 590
- Có missing / empty text không?: không
- Có duplicate không?: 834 bản (1.668%)
- 3 quan sát đáng chú ý về dữ liệu IMDB:
  - Dữ liệu hoàn toàn cân bằng
  - Độ dài review không đồng đều
  - Có xuất hiện 1.668% dữ liệu trùng lặp
## 2. Preprocessing design
- Bạn đã dùng những bước làm sạch nào?
  - Loại bỏ HTML: Xóa các thẻ như <br> bằng HTML_TAG_RE.
  - Chuẩn hóa khoảng trắng: Loại bỏ ký tự điều khiển (\p{C}+), thay thế thực thể \u00a0 và đưa các khoảng trắng thừa về một khoảng trắng duy nhất.
  - Xử lý định danh đặc biệt: Thay thế URL và Email bằng các token <URL> và <EMAIL> để giảm nhiễu nhưng vẫn giữ được cấu trúc văn bản.
- Bạn giữ lại dấu câu hay bỏ đi? Vì sao?
  - Giữ lại dấu câu (mặc định keep_punct=True) và tách chúng ra khỏi từ bằng khoảng trắng.
  - Lý do: Trong phân tích cảm xúc (IMDB), các dấu câu như ! hoặc ? mang tín hiệu cảm xúc mạnh (ví dụ: "Great!!!" khác với "Great"). Việc tách dấu giúp mô hình học được ý nghĩa của dấu câu như một token riêng biệt.
- Bạn có thay số bằng `<NUM>` không? Vì sao?
  - Không (mặc định replace_number=False).
  - Lý do: Đối với review phim, các con số thường liên quan đến điểm số (ví dụ: "9/10", "100 minutes"). Giữ lại số giúp mô hình có khả năng nhận diện các đánh giá định lượng quan trọng này.
- Có bước nào bạn cố tình **không** làm để tránh mất tín hiệu cảm xúc?
  - Không loại bỏ Stopwords: Những từ như "not", "no", "never" cực kỳ quan trọng để xác định sắc thái phủ định trong review.
  - Không Lemmatization/Stemming: Giữ nguyên dạng từ để bảo toàn sắc thái biểu cảm và thì của động từ (ví dụ: "disappointed" thường mạnh hơn "disappoint").
  - Không xóa dấu câu: Như đã nêu, việc giữ lại giúp bắt trọn sự phấn khích hoặc nghi vấn của người xem.

## 3. Experiment comparison
| Run | Text version | Vectorizer | Model | ngram | Macro-F1 | Accuracy | Ghi chú |
|---|---|---|---|---|---:|---:|---|
| 1 | cleaned | TF-IDF | LogisticRegression | 1-2 | 0.9064 | 0.9064 | baseline |
| 2 | cleaned | BoW | LogisticRegression | 1-2    | 0.8946 | 0.8946 | kém hơn 1.18% so với baseline | 
| 3 | cleaned | TF-IDF | LinearSVM | 1-2 | 0.91 | 0.91 | tốt hơn 0.36% so với baseline |

- Kết quả confusion matrix cho thấy model run 3 tốt hơn baseline một chút trong việc nhận diện ở cả 2 nhãn

baseline 

![Confusion Matrix](..\outputs\figures\confusion_matrix.png)

run 3

![Confusion Matrix](..\outputs_linearsvm\figures\confusion_matrix.png)

## 4. Error analysis (>= 10 lỗi)
- Chọn ít nhất 10 mẫu trong `outputs/error_analysis/error_analysis.csv`.
- Gom thành 2–4 nhóm lỗi.
- Gợi ý nhóm lỗi trên IMDB:
  - phủ định / tương phản (`not good`, `although ... still`)
  - cảm xúc trộn lẫn
  - mỉa mai / châm biếm
  - review quá dài, nhiều chi tiết không liên quan
  - mô hình dự đoán rất tự tin nhưng sai

### Bảng ghi lỗi
| ID | True | Pred | Nhóm lỗi | Giải thích ngắn |
|---|---|---|---|---|
| 31245 | negative | positive | Nhiều thực thể gây nhiễu | Review kể tên quá nhiều diễn viên giỏi và phim hay khác để so sánh, khiến BoW bắt được nhiều từ tích cực. |
| 37061 | negative | positive | Mỉa mai / Châm biếm | Người dùng dùng từ "pure genius", "magnificent" nhưng gán nhãn Negative (có thể do nhãn gốc sai hoặc mỉa mai quá sâu). |
| 8347 | negative | positive | Cảm xúc trộn lẫn | Review khen dàn cast và một số phim khác cùng thể loại nhưng chê phim hiện tại là "unremarkable". |
| 17596 | positive | negative | Cảm xúc trộn lẫn | Review dùng nhiều từ tiêu cực ("hardly a masterpiece", "not so well written") nhưng kết luận vẫn cho điểm B- và đánh giá Positive. |
| 34543 | negative | positive | Nhiều thực thể gây nhiễu | Review phân tích sâu về nội dung tôn giáo, dùng nhiều từ trừu tượng tích cực ("truth", "creative", "beautiful") dù nhãn là Negative. |
| 4648 | negative | positive | Cảm xúc trộn lẫn | Review khen diễn xuất ("acting is really good") nhưng chê nhịp phim và chỉ cho 4/10 điểm. |
| 8659 | negative | positive | Mỉa mai / Châm biếm | So sánh một bộ phim hạng B với "Taxi Driver" và "Clockwork Orange". Mô hình bị lừa bởi các danh từ kinh điển này. |
| 36707 | negative | positive | Mỉa mai / Châm biếm | Tác giả dùng những từ ngữ hoa mỹ nhất ("masterpiece", "visual flair") để nói về một bộ phim kinh dị rẻ tiền. |
| 29148 | negative | positive | Phủ định / Tương phản | Review nói về nỗi sợ (nightmare, terrifying) từ tuổi thơ. Mô hình nhầm tưởng cảm xúc mạnh này là sự hào hứng tích cực. |
| 36218 | negative | positive | Phủ định / Tương phản | Review thừa nhận diễn viên giỏi nhưng không chấp nhận được thông điệp tôn giáo. Mô hình bắt từ khóa về diễn viên. |

## 5. Reflection
- Pipeline nào tốt nhất trên IMDB? Vì sao?
  - Pipeline Run 3 (Cleaned + TF-IDF + LinearSVM + N-gram 1-2) là tốt nhất với Accuracy và Macro-F1 đạt 0.91 (91%).

  Lý do:

  - TF-IDF vs BoW: TF-IDF hiệu quả hơn Bag-of-Words (BoW) vì nó không chỉ đếm tần suất xuất hiện mà còn giảm trọng số của các từ phổ biến (như "the", "movie", "film") và tăng trọng số cho các từ mang tính đặc trưng, giúp mô hình tập trung vào các từ mang sắc thái cảm xúc mạnh.
  - LinearSVM: Trong không gian vector thưa thớt và có số chiều lớn (như N-gram 1-2), SVM thường tìm được siêu phẳng phân cách tối ưu tốt hơn so với Logistic Regression, đặc biệt là với các bài toán phân loại văn bản nhị phân.

- Trên IMDB, accuracy và macro-F1 có chênh nhau nhiều không?
  - Trên tập dữ liệu IMDB, Accuracy và Macro-F1 gần như không chênh lệch nhau (cả hai đều đạt ~0.91 ở Run 3).

  - Giải thích: Điều này cho thấy tập dữ liệu IMDB là một tập dữ liệu cân bằng (số lượng mẫu Positive và Negative xấp xỉ nhau). Khi các lớp cân bằng, Accuracy là một thước đo tin cậy và phản ánh đúng hiệu năng tổng thể của mô hình.
- Nếu chuyển sang dữ liệu lệch lớp hơn, bạn kỳ vọng metric nào sẽ phản ánh tốt hơn? Vì sao?
  - Nếu chuyển sang dữ liệu lệch lớp (ví dụ: 90% Positive, 10% Negative), tôi kỳ vọng Macro-F1 sẽ phản ánh tốt hơn hiệu năng của mô hình.

  - Vì sao? Accuracy sẽ bị "đánh lừa" trong trường hợp này; một mô hình "ngây thơ" chỉ cần dự đoán tất cả là Positive cũng đã đạt 90% Accuracy nhưng thực tế không học được gì từ lớp Negative. Ngược lại, Macro-F1 tính trung bình F1-score của từng lớp, do đó nó bắt buộc mô hình phải hoạt động tốt trên cả lớp thiểu số thì mới có điểm số cao.
- Một cải tiến bạn muốn thử ở Lab 3 là gì?
  - Sử dụng phương pháp xử lí ngôn ngữ và mô hình khác để cải thiện độ chính xác của mô hình