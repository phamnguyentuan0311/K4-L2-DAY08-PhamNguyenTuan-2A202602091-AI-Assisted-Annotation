# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có
ngân sách rà năm ảnh. Ghi tên, điểm, thời điểm, thứ tự và lý do; tối thiểu một quyết định phải xét
ảnh gần trùng hoặc trường hợp model không dự đoán được box: 
1. `frame_0182.jpg` (rank 1, score 0.9591, t=72.8s): Ảnh có điểm bất định (U) rất cao và có tới 18 box mập mờ (ambiguous), là ca khó nhất model gặp phải.
2. `frame_0369.jpg` (rank 2, score 0.9324, t=147.6s): Điểm bất định cao, tuy nhiên cách xa frame 182 nên mang lại bối cảnh khác biệt, đáng để chọn.
3. `frame_0326.jpg` (rank 4, score 0.9155, t=130.4s): Một frame ở thời điểm 130.4s, cách xa các frame trước, có số box mập mờ cao (15). Tôi bỏ qua frame_0380 vì nó ở 152.0s, quá gần frame_0369 (147.6s) - rà cả hai có thể lãng phí công sức do trùng cảnh.
4. `frame_0312.jpg` (rank 7, score 0.9100, t=124.8s): Có tới 18 box mập mờ, tỷ lệ box rỗng không cao, mang nhiều thông tin cho model học.
5. `frame_0099.jpg` (rank 8, score 0.9063, t=39.6s): Frame ở đoạn đầu video, bổ sung sự đa dạng về thời gian so với các frame chủ yếu ở cuối video.

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet: 
1. `frame_0182.jpg`: Model dự đoán có 28 box nhưng có tới 18 box mập mờ (ambiguous), điểm bất định U cao nhất.
2. `frame_0331.jpg`: Model dự đoán 47 box, cao nhất trong top đầu, cho thấy mật độ xe rất dày đặc.
3. `frame_0270.jpg`: Điểm U = 0.9089 rất cao, nằm ở khoảng thời gian giữa video (108.0s).

Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do: 
`frame_0380.jpg` có rank 3 và điểm rất cao (0.917) nhưng không nên chọn nếu đã chọn `frame_0369.jpg` vì chúng cách nhau chưa đầy 5 giây, cảnh quan và vị trí các xe có thể chưa thay đổi nhiều, dẫn đến việc gán nhãn bị trùng lặp, không đem lại nhiều thông tin mới cho model.

Điều phép chọn này chưa chứng minh về chất lượng mô hình: 
Phép chọn dựa trên độ bất định (uncertainty) chỉ cho thấy những ảnh mà model "bối rối" nhất, chứ không phản ánh được chất lượng tổng thể của mô hình trên toàn bộ tập dữ liệu thực tế. Model có thể chắc chắn (tự tin cao) ở nhiều ảnh khác nhưng lại đoán sai (false positives/false negatives).
