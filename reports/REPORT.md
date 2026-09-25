# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Pham Nguyen Tuan

Công cụ gán nhãn đã dùng: CVAT



## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng
đệm ở giữa, thay vì chia ngẫu nhiên? Nếu chia ngẫu nhiên, số đo trên tập kiểm thử sẽ bị lệch theo
hướng nào, và vì sao?

Vì camera đứng yên và xe di chuyển chậm, nếu chia ngẫu nhiên, các ảnh sát nhau về mặt thời gian (chỉ cách nhau 0.4s) sẽ rơi vào cả tập train và tập test. Cùng một chiếc xe ở cùng một vị trí sẽ được học ở tập train và sau đó được kiểm tra lại ở tập test. Điều này gây rò rỉ dữ liệu (data leakage), làm số đo trên tập kiểm thử bị lệch cao hơn thực tế do mô hình chỉ đang "nhớ" chiếc xe đó thay vì thực sự học cách phát hiện. Chia theo thời gian có vùng đệm giúp tập test hoàn toàn độc lập với tập train.

## 2. Mô hình khởi đầu lạnh (cold start)

Chép dòng vòng 0 từ `rounds_table.md`. Dựa vào `outputs/compare_round0.jpg`, cho biết mô hình khởi
đầu lạnh không khớp nhãn tham chiếu ở những loại xe nào. Độ phủ (recall) theo kích thước xe cho
thấy điều gì? Một trường hợp nào cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai?

Vòng 0: | 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |

Mô hình khởi đầu lạnh thường bỏ sót (False Negative) các xe ở rất xa (xe kích thước nhỏ) hoặc các xe bị khuất, mờ, chỉ thấy ánh đèn.
Độ phủ (recall) theo kích thước: small=0.182, medium=0.547, large=0.561 cho thấy mô hình dự đoán rất tệ đối với xe nhỏ (chỉ phát hiện được 18.2%), trong khi làm tốt hơn với xe lớn.
Một trường hợp cần rà lại nhãn tham chiếu là các xe ở sát đường chân trời, cao dưới 16px, nơi mà nhãn tham chiếu cũng có thể sai hoặc thiếu sót do người/mô hình tạo nhãn tham chiếu ban đầu không nhất quán.

## 3. Chiến lược chọn mẫu

Giải thích bằng lời công thức `score = W_U·U + W_A·A + W_D·D` và vai trò của `MIN_GAP_S`.
Dẫn ba frame trong `reports/SELECTION.md` và một frame khác để chứng minh cách bạn cân nhắc
độ bất định, ảnh gần trùng và công gán nhãn. Điểm bất định có chứng minh ảnh đó sẽ cải thiện
mô hình không? Vì sao?

Công thức `score = W_U·U + W_A·A + W_D·D` tổng hợp 3 yếu tố để chọn ảnh: U (Uncertainty - độ bất định của mô hình), A (Ambiguity - số lượng box mập mờ, phản ánh công sức gán nhãn), và D (Diversity - độ đa dạng, cách xa các ảnh đã chọn). 
Vai trò của `MIN_GAP_S`: Đảm bảo các ảnh được chọn phải cách nhau ít nhất một khoảng thời gian (ví dụ 2 giây) để tránh chọn các ảnh quá giống nhau (trùng cảnh).
Ví dụ: Tôi chọn `frame_0182.jpg` vì U cao nhất (0.9182), `frame_0326.jpg` thay vì `frame_0380.jpg` để đảm bảo không trùng với `frame_0369.jpg` (cân nhắc đa dạng), và `frame_0099.jpg` để bổ sung đa dạng về thời gian dẫu điểm U không phải cao nhất.
Điểm bất định cao không hoàn toàn chứng minh ảnh đó sẽ cải thiện mô hình, vì có thể ảnh bị nhiễu, mờ hoặc chứa các vật thể không phải xe gây bối rối cho mô hình, nhưng việc học chúng lại không giúp ích cho các trường hợp chung.

## 4. Các vòng học chủ động (active learning)

Chép bảng từ `rounds_table.md`. Với mỗi vòng, trình bày:

- mức độ bạn đã sửa nhãn gợi ý (số box giữ nguyên, chỉnh sửa, xoá, thêm mới, lấy từ
  `outputs/round*_diff.md`);
- AP50 thay đổi bao nhiêu so với khởi đầu lạnh và so với vòng trước;
- nhóm xe nào tốt lên hoặc xấu đi theo số đo trên cùng tập test.

Dựa vào các ảnh `compare_round*.jpg`, chỉ ra một ca kết quả đổi sau fine-tune (tốt hơn hoặc xấu
đi), cùng lý do có thể kiểm. Dùng `BLIND_SCAN.md`, `REVIEW_LOG.csv` và `round1_diff.md` phân biệt
quan sát độc lập, lỗi pre-label đã sửa và kết quả mô hình sau train. Mô tả một ca khó theo guideline.

Bảng từ `rounds_table.md`:
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vong 1..1 | 12 | 169 | 0.378 | -0.393 | 1.000 | 0.030 | 0.058 | 0.000 | 0.020 | 0.146 |

- Mức độ sửa nhãn gợi ý (vòng 1): Giữ nguyên 169 box, chỉnh sửa 0, xóa 0, thêm mới 0 (Accept rate 100%).
- Thay đổi AP50: Ở vòng 1, AP50 giảm mạnh từ 0.771 xuống 0.378 (giảm 0.393 so với cold start).
- Nhóm xe: Khả năng phát hiện tụt giảm nghiêm trọng ở tất cả các kích thước (small từ 0.182 xuống 0.000, medium từ 0.547 xuống 0.020). Mô hình sau fine-tune dường như đã bị "catastrophic forgetting" (quên kiến thức cũ) hoặc quá mức tự tin khắt khe (precision lên 1.0 nhưng recall rớt thê thảm xuống 0.030).
- Dựa trên `compare_round1.jpg`, mô hình sau fine-tune gần như không dự đoán ra các box (chữ FN - false negative dày đặc đỏ rực), cho thấy mô hình không còn phát hiện ra các xe ở xa hay thậm chí cả xe ở gần, kết quả xấu đi rất nhiều so với vòng 0.
- Ca khó theo guideline: Những xe ở rất xa, chỉ còn hai chấm đèn. Ở `REVIEW_LOG.csv`, tôi đã accept nhãn AI cho "xe ở xa tối màu" với lý do "Chỉ thấy đèn nhưng vẫn đoán được ranh giới thân xe".

## 5. Kết luận và giới hạn

Kết quả vòng này so với cold start ra sao? Vì sao bạn dừng hoặc tiếp tục? Đề xuất hai ca còn yếu
hoặc bất định cho vòng sau, kèm chi phí rà nhãn và nguy cơ ảnh gần trùng. Tập kiểm thử chỉ 20 ảnh,
có luật bỏ qua xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được rà thủ công; các giới hạn đó
ảnh hưởng thế nào đến kết luận? Nếu AP50 giảm, bạn sẽ kiểm tra điều gì trước khi train thêm?

Kết quả vòng 1 giảm mạnh so với cold start (AP50 giảm 0.393). Việc chỉ dùng 12 ảnh để fine-tune trực tiếp mô hình YOLOv8 pretrained đã khiến nó mất đi khả năng tổng quát hóa vốn có.
Tôi quyết định dừng ở đây để phân tích tại sao việc học thêm 12 ảnh lại phá hỏng mô hình (precision 1.0 nhưng recall 0.03).
Đề xuất ca còn yếu: Các xe nhỏ ở xa và các xe bị lấp ló ánh đèn. Tuy nhiên chi phí rà nhãn cho các xe nhỏ rất tốn thời gian.
Giới hạn của tập test: Việc test set chỉ có 20 ảnh và nhãn do mô hình tạo chưa được người rà thủ công có thể làm sai lệch AP50 thực tế, tuy nhiên sự sụt giảm ở vòng 1 là quá lớn không thể giải thích bằng nhãn test sai.
Khi AP50 giảm, trước khi train thêm, tôi sẽ kiểm tra lại: (1) Cách export nhãn YOLO đã chuẩn chưa (tọa độ normalize đúng chưa), (2) Việc không điều chỉnh nhãn (accept 100%) có vô tình củng cố lỗi của AI không, và (3) Hyperparameters của fine-tuning (số epoch, learning rate) có đang phá vỡ tệp trọng số pre-trained không.
