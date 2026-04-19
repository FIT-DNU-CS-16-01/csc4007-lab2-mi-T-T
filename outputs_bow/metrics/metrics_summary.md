# Metrics Summary
- dataset: imdb
- vectorizer: bow
- model: logreg
- val_accuracy: 0.8890
- val_macro_f1: 0.8890
- test_accuracy: 0.8946
- test_macro_f1: 0.8946

## How to read the result on IMDB
- IMDB khá cân bằng nên accuracy và macro-F1 có thể gần nhau.
- Dù vậy vẫn cần báo cáo macro-F1 để giữ thói quen đánh giá đúng khi sang dữ liệu lệch lớp.

## Test per-class report
- class `negative`: precision=0.8973, recall=0.8912, f1=0.8942, support=2500.0
- class `positive`: precision=0.8919, recall=0.8980, f1=0.8950, support=2500.0
- class `macro avg`: precision=0.8946, recall=0.8946, f1=0.8946, support=5000.0
- class `weighted avg`: precision=0.8946, recall=0.8946, f1=0.8946, support=5000.0
