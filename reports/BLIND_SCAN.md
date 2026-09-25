# Quét độc lập trước khi xem pre-label

Frame: frame_0099.jpg

Số xe nhìn thấy bằng mắt: Khoảng 29 xe

Hai vị trí dễ bị AI bỏ sót hoặc vẽ sai, kèm mô tả xe: 
1. Góc xa sát đường chân trời phía trên: Các xe ở đây rất nhỏ, chỉ còn là 2 đốm đèn, dễ bị AI bỏ sót (FN) hoặc đoán sai kích thước bao quanh.
2. Vùng tối ở làn đường bên trái cùng: Xe bị khuất sáng, chỉ thấy một phần đèn và bóng thân xe nhòe, dễ khiến AI vẽ box quá nhỏ hoặc nhận diện nhầm lóa đèn thành xe.

Chạy `python3 tools/lock_blind.py` ngay sau khi điền. Sau đó giữ file này nguyên vẹn.
