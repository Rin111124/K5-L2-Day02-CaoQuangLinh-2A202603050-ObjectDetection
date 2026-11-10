# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Cao Quang Linh
- MSSV: 2A202603050
- Email: 26ai.linhcq2@vinuni.edu.vn
- Lớp: L2A
- Link repo bài làm: https://github.com/Rin111124/K5-L2-Day02-CaoQuangLinh-2A202603050-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1HL3n8Us9_FAcQ0MkZp5hUlBa4wtc93n2?usp=sharing

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần cần giảng viên làm rõ.

### Tóm tắt áp dụng schema:
- Bộ nhãn gồm 4 lớp phương tiện giao thông:
  - `car` (0): Toàn bộ xe con cá nhân, sedan, hatchback, SUV, crossover, coupe, taxi và xe bán tải (pickup).
  - `truck` (1): Xe tải chở hàng thương mại (thùng kín, xe ben, xe bồn, container) có cabin tách biệt rõ ràng với phần thùng chở hàng.
  - `bus` (2): Xe buýt công cộng, xe khách liên tỉnh cỡ lớn (> 16–29 chỗ), thân xe dài và cao vượt trội.
  - `van` (3): Xe chở hàng hoặc chở người dạng hình hộp liền khối (minivan gia đình, xe 9–16 chỗ như Transit/Solati).

### Tóm tắt các ca mơ hồ đã quyết định trong quá trình gán:
1. **Ảnh `drive_003`**: Phương tiện ở làn giữa $(x \approx 0.50, y \approx 0.26)$ nhìn từ xa kiểu dáng giống giữa `car` (bán tải/pickup) và `truck` nhỏ.  
   $\rightarrow$ **Quyết định:** Gán nhãn `truck` (class 1) do kết cấu thân xe có cabin tách rời với thùng hàng chuyên dụng chở hàng phía sau.
2. **Ảnh `drive_002`**: Hai xe chạy nối đuôi nhau ở làn giữa, xe phía sau bị đuôi xe phía trước che khuất một phần góc đầu xe (occlusion).  
   $\rightarrow$ **Quyết định:** Vẽ 2 bounding box riêng biệt; box của xe sau chỉ ôm sát phần thân xe thực tế nhìn thấy được (visible region), không vẽ gộp và không tự suy đoán phần bị che khuất.
3. **Ảnh `drive_001`**: Xe con ở góc mép phải dưới khung hình $(x \approx 0.93, y \approx 0.33)$ bị rìa ảnh cắt ngang nửa thân xe bên phải (truncation).  
   $\rightarrow$ **Quyết định:** Gán nhãn `car` (class 0) với cạnh phải của box chạm sát mép biên ảnh ($x_{\text{max}} = 1.0$), không ước lượng vùng nằm ngoài khung hình.
4. **Ảnh `drive_001`**: Các xe nhỏ ở làn đường trên cao phía xa chân trời ($y < 0.22$) có kích thước rất nhỏ và mờ.  
   $\rightarrow$ **Quyết định:** Không gán box vì nằm ngoài phạm vi luồng giao thông chính của camera CCTV và không đủ đặc điểm nhận dạng chắc chắn.

### Phần cần giảng viên làm rõ thêm:
- Tiêu chuẩn phân định ranh giới giữa xe minivan/MPV 7 chỗ gầm cao (`car` hay `van`) và xe 16 chỗ dáng dài (`van` hay `bus`) khi quan sát từ góc máy camera CCTV độ phân giải trung bình ở cự ly xa.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`dataset/labels/train/drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy script `python tools/check_yolo_labels.py` cho kết quả **ĐẠT cấu trúc: 50/50 file nhãn hợp lệ** (đúng basename, chuẩn 5 giá trị float chuẩn hóa trong khoảng $[0, 1]$, ID lớp hợp lệ $\{0, 1, 2, 3\}$, không có bounding box nào bị tràn viền hoặc tọa độ âm).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_004`, `drive_005`, `drive_006`, `drive_007`, `drive_008`, `drive_009`, `drive_010`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - *Lỗi bỏ sót (Completeness):* Trong ảnh `drive_002` và `drive_005`, một số xe con ở xa hoặc xe bị che khuất một phần góc đầu xe ban đầu bị bỏ sót; đã bổ sung box đầy đủ cho phần nhìn thấy.
  - *Lỗi bám sát (Tightness):* Một vài box ban đầu bị vẽ dư xuống bóng đổ trên mặt đường hoặc lấy sang làn bên cạnh; đã căn chỉnh mép đáy box bám sát điểm tiếp xúc của bánh xe với mặt đường và mép trên sát trần xe.
  - *Lỗi phân lớp (Class):* Phân biệt rõ xe bán tải (pickup) thuộc lớp `car` theo schema thay vì nhầm sang `truck`.
- Commit khóa nhãn ban đầu: `55f3e6f` (Commit message: `complete annontate and mini guide`).

## 4. Gold QC và rework

Dẫn tới [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json) trong repo.

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 534 | - |
| FP | 55 | - |
| FN | 119 | - |
| Precision | 0.9066 | - |
| Recall | 0.8178 | - |
| F1 | 0.8599 | - |
| mean_iou_of_matches | 0.8622 | - |
| passed | true | - |

### Ghi chép về quá trình QC và Rework:
- **Kết quả đánh giá chất lượng:** Sau khi đối chiếu với tập chuẩn `gold_train_labels/` ở ngưỡng IoU $\ge 0.50$, kết quả đạt **F1 = 0.8599** (ngưỡng yêu cầu $\ge 0.80$) và **mean IoU of matches = 0.8622** (ngưỡng yêu cầu $\ge 0.70$). Chỉ số `passed = true` ngay từ vòng đánh giá đầu tiên.
- **Rework:** Do bộ nhãn đã vượt qua cả hai ngưỡng kiểm định của Gate (thuộc mức chất lượng Đạt, tiệm cận Xuất sắc với Mean IoU > 0.86), bộ nhãn không cần thực hiện thêm vòng rework lớn để can thiệp cấu trúc.
- **Phân tích nguyên nhân các trường hợp FN và FP:**
  - *119 False Negatives (FN):* Nguyên nhân chủ yếu xuất phát từ các phương tiện ở vị trí cực xa sát đường chân trời hoặc các xe bị che khuất rất nặng (> 70–80%) ở góc mép camera mà học viên chủ động bỏ qua theo quy tắc không đoán mò trong guideline, trong khi gold set của giảng viên vẫn gán nhãn cho các xe này.
  - *55 False Positives (FP):* Do học viên vẽ thêm một số box ở các xe làn phụ phía xa mà gold set không bao gồm, hoặc một số trường hợp mơ hồ về nhãn lớp giữa `van` và `car` ở cự ly xa. Chỉ số `mean_iou_of_matches = 0.8622` chứng minh các box ghép cặp có độ bám sát hình học rất cao và đồng nhất.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
  - Môi trường: Google Colab
  - Phần cứng: CPU (Intel Xeon CPU @ 2.20GHz, 2 vCPU)
  - Thư viện: `ultralytics 8.4.208`, `torch 2.14.1+cpu`, `Python 3.13.18`
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên toàn bộ cấu hình gốc của notebook (`model = YOLO('yolo26n.pt')`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`, `device=cpu`).
- Đường dẫn best checkpoint trong phiên chạy:
  `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

### Bảng chỉ số tổng thể trên 10 ảnh eval:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3178 (31.78%) |
| Recall | 0.3461 (34.61%) |
| mAP50 | 0.2935 (29.35%) |
| mAP50–95 | 0.1957 (19.57%) |

### Chi tiết chỉ số đánh giá theo từng lớp trên tập eval:
- `car` (94 instances): Precision = 0.568, Recall = 0.851, mAP50 = 0.803, mAP50–95 = 0.530
- `truck` (5 instances): Precision = 0.458, Recall = 0.200, mAP50 = 0.209, mAP50–95 = 0.165
- `bus` (2 instances): Precision = 0.000, Recall = 0.000, mAP50 = 0.0215, mAP50–95 = 0.0145
- `van` (12 instances): Precision = 0.245, Recall = 0.333, mAP50 = 0.140, mAP50–95 = 0.0734
- `all` (113 instances): Precision = 0.318, Recall = 0.346, mAP50 = 0.293, mAP50–95 = 0.196

### Đường dẫn thư mục assets kết quả:
- Ảnh dự đoán trực quan trên tập eval: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/)
- Đồ thị và log huấn luyện YOLO: [assets/yolo26_runs/vehicle_annotation/](assets/yolo26_runs/vehicle_annotation/) và [assets/yolo26_runs/vehicle_eval/](assets/yolo26_runs/vehicle_eval/)

### Nhận xét trên ảnh và vật thể cụ thể:
- **Ảnh `drive_055.jpg`:** Mô hình phát hiện chính xác cả 3 xe con (`car`) đang di chuyển trên đường với bounding box ôm khít và độ tin cậy cao, thể hiện năng lực tốt của mô hình đối với lớp chiếm đa số mẫu.
- **Ảnh `drive_053.jpg` và `drive_058.jpg`:** Mô hình phát hiện được xe con nhưng dự đoán nhầm một số xe lớn/van thành `bus` (2 `bus` ở `drive_053` và 4 `bus` ở `drive_058`), trong khi ground truth trên 10 ảnh eval chỉ có tổng cộng 2 cá thể `bus`.

---

### Trả lời bốn câu hỏi cuối notebook:

#### 1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
- Lớp có Recall thấp nhất là **`bus`** với $Recall = 0.000$ ($0\%$), kế tiếp là **`truck`** với $Recall = 0.200$ ($20\%$).
- **Ảnh minh họa:** Trong ảnh `assets/eval_predictions/visuals/drive_053.jpg` và `assets/eval_predictions/visuals/drive_056.jpg`, mô hình không bắt được chính xác cá thể xe khách/xe buýt thật của ground truth hoặc dự đoán nhầm lẫn với xe khác.
- **Giải thích:** Nguyên nhân gốc rễ là sự **mất cân bằng dữ liệu nghiêm trọng (class imbalance)** trong 50 ảnh train. Trong tập huấn luyện, lớp `car` chiếm đại đa số (hàng trăm instances), trong khi `bus` chỉ xuất hiện rất hiếm hoi (1–2 lần) và `truck` cũng rất ít. Mô hình YOLO26n với dung lượng tham số nhỏ (2.39M params) được train chỉ 50 epochs không nhận đủ tín hiệu gradient cần thiết để học phân biệt đặc trưng của lớp thiểu số này.

#### 2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
- Khi $mAP_{50}$ cao nhưng $mAP_{50-95}$ thấp, cần kiểm tra **độ bám sát hình học (tightness) và độ chính xác tọa độ viền biên** của bounding box.
- **Giải thích:**
  - $mAP_{50}$ chỉ yêu cầu tỷ lệ chồng lấn $IoU \ge 0.50$ giữa box dự đoán và ground truth. Ngưỡng này tương đối dễ dãi, chỉ cần box định vị được vùng trọng tâm của vật thể là đã được tính là True Positive.
  - Ngược lại, $mAP_{50-95}$ tính trung bình mAP trên 10 ngưỡng IoU liên tiếp từ $0.50$ đến $0.95$ với bước nhảy $0.05$. Tại các ngưỡng IoU khắt khe ($0.75, 0.85, 0.95$), nếu bounding box vẽ bị lỏng (lấn ra mặt đường, khoảng trời, xe bên cạnh) hoặc vẽ thiếu mép xe, chỉ số IoU sẽ không vượt qua được ngưỡng yêu cầu. Điều này dẫn tới điểm AP tại các ngưỡng cao bị giảm mạnh, kéo tụt điểm trung bình $mAP_{50-95}$.

#### 3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
- Lỗi phân loại nhầm giữa **`car` vs `van`** và giữa **`van` vs `bus`**.
- **Dữ kiện hỗ trợ:**
  - Trên tập eval, lớp `van` có độ chính xác rất thấp ($Precision = 0.245$, $mAP_{50} = 0.140$); lớp `bus` có $Precision = 0.0$ và $Recall = 0.0$. Tuy nhiên, khi dự đoán trên ảnh eval (`drive_053`, `drive_056`, `drive_058`), mô hình lại gắn nhãn dự đoán là `bus` cho nhiều phương tiện.
  - Từ góc chụp camera giám sát giao thông trên cao nhìn xuống ở khoảng cách xa, hình dáng của các xe minivan gia đình 7–9 chỗ (dễ nhầm với SUV thuộc `car`) và xe khách nhỏ 16 chỗ dạng Transit/Solati (dễ phân vân giữa `van` và `bus`) có đặc trưng thị giác rất tương đồng. Nếu guideline không quy định cụ thể về kích thước chiều dài thân xe, số hàng cửa sổ hoặc kết cấu cửa trượt thì người gán nhãn rất dễ dao động, gây nhiễu nhãn đào tạo cho mô hình.

#### 4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?
- **Lý do không dùng để train:**
  - Trong học máy và thị giác máy tính, việc giữ tập đánh giá tách biệt hoàn toàn khỏi tập huấn luyện là nguyên tắc cốt lõi nhằm ngăn chặn hiện tượng **rò rỉ dữ liệu (data leakage)** và **học vẹt (overfitting)**.
  - Nếu đưa 10 ảnh eval vào tập train, mô hình sẽ học thuộc lòng các hình ảnh này. Khi đó, các chỉ số đo lường sẽ bị thổi phồng giả tạo và mất đi giá trị đo lường khả năng **tổng quát hóa (generalization)** trên dữ liệu thực tế trong tương lai.
- **Vai trò của 10 ảnh eval trong notebook:**
  - Đóng vai trò là **Validation Set**: Dùng để đo lường loss và metrics (mAP, Precision, Recall) sau mỗi epoch trong quá trình huấn luyện mà không can thiệp vào việc cập nhật trọng số, từ đó hỗ trợ thuật toán lưu lại checkpoint có kết quả tốt nhất (`best.pt`).
  - Đóng vai trò là công cụ chẩn đoán (Error Analysis): Giúp kỹ sư AI nhìn thấy điểm yếu của mô hình (lớp nào yếu, box có ôm sát không) để truy ngược về chất lượng của bộ dữ liệu và sự cân bằng giữa các lớp.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| **CP2 — Gán nhãn** | Nhiều xe con ở làn đường xa bị che khuất một phần (occlusion) hoặc cắt mép ảnh (truncation) rất khó xác định viền ngoài. | Phóng to ảnh (zoom 200–300%), chỉ vẽ box ôm sát phần thân xe thực tế nhìn thấy được; cho cạnh box chạm đúng biên ảnh ($x, y \in \{0, 1\}$). | Đạt độ bám sát rất cao, Mean IoU đối chiếu với gold set đạt tới 0.8622. |
| **CP2 — Phân loại lớp** | Dễ nhầm lẫn giữa xe bán tải (pickup) với xe tải (`truck`), và xe minivan với xe khách (`bus`). | Bám sát định nghĩa trong guideline: pickup vẫn là `car`; chỉ gán `truck` khi thùng hàng tách rời cabin; xe hình hộp liền khối xếp vào `van`. | Precision đạt 0.9066, hạn chế tối đa nhầm lẫn class. |
| **CP6 — Train Colab** | Phiên chạy Google Colab chạy trên CPU dẫn đến thời gian huấn luyện 50 epochs kéo dài (~39 phút). | Giữ phiên kết nối Colab ổn định, theo dõi loss giảm đều qua từng epoch và xác nhận checkpoint `best.pt` được tạo thành công. | Quá trình train hoàn tất an toàn, tạo đầy đủ file ZIP log và kết quả đánh giá. |

## 7. Tự đánh giá theo rubric

Đối chiếu theo [RUBRIC.md](../../RUBRIC.md) (Thang điểm 100):

| Tiêu chí | Kết quả bài làm | Tự đánh giá điểm |
|---|---|---:|
| **Hoàn thành và định dạng YOLO** | Có đủ 50 file `.txt` cùng basename với ảnh train, định dạng 5 cột normalized chuẩn, 4 class ID hợp lệ, vượt qua script kiểm tra `check_yolo_labels.py` (50/50). | 10 / 10 |
| **Đầy đủ vật thể** | Bắt được hầu hết các xe nhìn thấy rõ và xe bị che/cắt khung; Recall đạt 0.8178 trên tập gold train. | 19 / 20 |
| **Độ bám sát và hình học box** | Mean IoU of matches đạt 0.8622 (vượt mức Xuất sắc $\ge 0.80$). Box ôm sát thân xe, không lấn nền hay gộp vật thể. | 20 / 20 |
| **Phân lớp nhất quán** | Precision đạt 0.9066; phân biệt rành mạch 4 lớp theo guideline; không gán các vật thể ngoài schema (xe máy, người đi bộ). | 15 / 15 |
| **Mini guideline và ca mơ hồ** | Hoàn thành `GUIDELINE_MINI.md` với định nghĩa 4 lớp, quy tắc box và 4 ca mơ hồ có phân tích lý do cụ thể. | 10 / 10 |
| **Gold-set annotation QC** | Đạt ngay vòng đầu với $F1 = 0.8599 \ge 0.80$ và $mean\_iou = 0.8622 \ge 0.70$. Có ghi chép phân tích nguyên nhân FN/FP. | 15 / 15 |
| **Pipeline YOLO và phân tích** | Colab chạy đầy đủ 50 epochs; giữ nguyên split eval làm validation; trích xuất đủ chỉ số, link assets và trả lời sâu sắc 4 câu hỏi phân tích. | 10 / 10 |
| **Tổng điểm tự đánh giá** | **Xuất sắc** | **99 / 100** |

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI đã sử dụng:**
  - Sử dụng trợ lý AI trong Antigravity IDE để hỗ trợ trích xuất cấu trúc dữ liệu JSON từ file log Colab, đối chiếu chỉ số kiểm định và định dạng cấu trúc báo cáo Markdown. Toàn bộ quyết định gán nhãn trên 50 ảnh train và việc phân tích kết quả chuyên môn đều do học viên trực tiếp thực hiện và kiểm chứng.
- **Tài liệu tham khảo:**
  - Tài liệu chính thức Ultralytics YOLO: https://docs.ultralytics.com/
  - Tài liệu hướng dẫn bài tập nội bộ: `GUIDE.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md`.
  - Không sử dụng các tài liệu ngoài luồng, không chia sẻ nhãn gold train và không lưu trữ token/mật khẩu trong bài làm.
