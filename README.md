# DCT ECC Erasure Defense

Bài thực hành Labtainer về **phòng thủ tấn công hình học vào kỹ thuật giấu tin trong ảnh dựa trên biến đổi miền tần số DCT**.

Chủ đề chính:

- Giấu tin trong ảnh bằng DCT block 8x8.
- Phân tích dung lượng nhúng dựa trên DCT energy.
- Thiết kế vùng nhúng bằng block mask.
- Tăng khả năng phục hồi bằng ECC và interleaving.
- Phát hiện block lỗi bằng erasure detection.
- Khôi phục watermark bằng adaptive tile recovery.
- Quan sát trực quan bản đồ lỗi, bản đồ erasure và tile survival.

---

# Mục tiêu

Sau khi hoàn thành bài lab, sinh viên có thể:

- Hiểu cách nhúng watermark vào ảnh bằng DCT.
- Hiểu ảnh hưởng của crop, shift, rotation và scaling tới block DCT.
- Phân tích vùng ảnh phù hợp để nhúng watermark dựa trên DCT energy.
- Thiết kế block mask để tránh vùng trơn và vùng sát biên.
- Sử dụng ECC để sửa lỗi bit sau tấn công hình học.
- Sử dụng interleaving để phân tán lỗi cụm thành lỗi rải rác.
- Phát hiện block không đáng tin cậy bằng confidence score.
- Loại bỏ tile bị hỏng nặng trong quá trình khôi phục.
- Đánh giá kết quả bằng BER, erasure rate và tile survival rate.
- Viết memo phân tích nguyên nhân lỗi và hiệu quả phòng thủ.

---

# Kiến thức chính

## 1. DCT Watermarking

Trong bài lab, ảnh được chia thành các block 8x8. Watermark được nhúng vào miền tần số bằng DCT.

Các vùng ảnh có texture và năng lượng DCT cao thường phù hợp để nhúng hơn vùng quá trơn, vì watermark ít gây méo ảnh và dễ khôi phục hơn sau tấn công.

---

## 2. ECC và Interleaving

ECC giúp sửa lỗi bit sau khi watermark bị tấn công.

Interleaving giúp phân tán các bit liên tiếp ra nhiều vùng khác nhau trên ảnh. Khi một vùng ảnh bị crop hoặc biến dạng, lỗi không tập trung vào một đoạn thông điệp mà được trải đều hơn, giúp ECC dễ khôi phục.

---

## 3. Erasure Detection

Không phải block nào sau tấn công cũng đáng tin cậy. Nếu block có confidence thấp hoặc hệ số DCT quá yếu, hệ thống sẽ đánh dấu block đó là erasure thay vì đoán bit.

Cách này giúp giảm số bit sai đưa vào bộ giải mã ECC.

---

## 4. Adaptive Tile Recovery

Ảnh được chia thành nhiều tile. Sau tấn công hình học, một số tile có thể bị crop, padding hoặc biến dạng mạnh.

Adaptive tile recovery sẽ ưu tiên tile còn sống tốt và loại bỏ tile xấu để tăng khả năng khôi phục watermark.

---

# Tải bài lab

```bash
imodule https://github.com/KhoaPTIT/dct-ecc-erasure-defense/raw/master/dct-ecc-erasure-defense.tar
```

---

# Khởi động bài lab

```bash
labtainer dct-ecc-erasure-defense
```

Nếu đang phát triển hoặc chỉnh sửa lab, có thể rebuild:

```bash
rebuild dct-ecc-erasure-defense
```

---

# Nội dung thực hành

## Task 1 — Tạo ảnh cover scene

Task này tạo ảnh cover để sử dụng trong toàn bộ bài lab.

Chạy:

```bash
python3 generate_scene.py
eog cover_scene.png &
cat scene_report.txt
```

Sinh viên cần quan sát:

- `cover_scene.png`: ảnh gốc.
- Các vùng ảnh có texture khác nhau.
- Nội dung trong `scene_report.txt`.

Kết quả checkwork:

```text
Y - task1_scene
```

---

## Task 2 — Phân tích dung lượng DCT

Task này phân tích năng lượng DCT của các block ảnh.

Chạy:

```bash
python3 analyze_dct_capacity.py
eog dct_energy_map.png &
cat dct_capacity_report.txt
```

Sinh viên cần quan sát:

- `dct_energy_map.png`: bản đồ năng lượng DCT.
- Vùng nào có năng lượng cao.
- Vùng nào quá trơn, không phù hợp để nhúng.
- Nội dung trong `dct_capacity_report.txt`.

Kết quả checkwork:

```text
Y - task2_dct_capacity
```

---

## Task 3 — Thiết kế block mask

Task này yêu cầu sinh viên chỉnh cấu hình chọn block nhúng watermark.

Mở file:

```bash
nano block_config.py
```

Sửa thành:

```python
MIN_DCT_ENERGY = 22.0
MAX_SMOOTHNESS = 65.0
EXCLUDE_BORDER_BLOCKS = 2
USE_TEXTURE_ONLY = True
```

Chạy:

```bash
python3 design_block_mask.py
eog selected_blocks_mask.png &
eog tile_layout.png &
cat block_mask_report.txt
```

Sinh viên cần quan sát:

- `selected_blocks_mask.png`: các block được chọn để nhúng.
- `tile_layout.png`: bố cục tile trên ảnh.
- Block sát biên đã được hạn chế.
- Vùng quá trơn không được ưu tiên nhúng.

Kết quả checkwork:

```text
Y - task3_block_mask
```

---

## Task 4 — Thiết kế ECC

Task này cấu hình mã sửa lỗi cho watermark.

Mở file:

```bash
nano ecc_config.py
```

Sửa thành:

```python
ECC_MODE = "hamming_parity"
PARITY_BYTES = 12
REPEAT_FACTOR = 3
LANE_COUNT = 6
```

Chạy:

```bash
python3 ecc_encode_message.py
cat ecc_report.txt
eog ecc_syndrome_table.png &
```

Sinh viên cần quan sát:

- `ecc_report.txt`: thông tin mã hóa ECC.
- `ecc_syndrome_table.png`: bảng minh họa kiểm tra và sửa lỗi.
- Số parity bytes.
- Repeat factor.
- Số lane được dùng để phân tán dữ liệu.

Kết quả checkwork:

```text
Y - task4_ecc_design
```

---

## Task 5 — Xây dựng interleaver map

Task này phân tán các bit watermark ra nhiều vùng ảnh khác nhau.

Mở file:

```bash
nano interleaver_config.py
```

Sửa thành:

```python
INTERLEAVER_MODE = "tile_spread"
STRIDE_X = 7
STRIDE_Y = 11
SHUFFLE_SEED = 164
MIN_TILE_DISTANCE = 3
```

Chạy:

```bash
python3 build_interleaver.py
eog interleaver_map.png &
cat interleaver_report.txt
```

Sinh viên cần quan sát:

- `interleaver_map.png`: bản đồ phân tán bit.
- Bit liên tiếp không nằm sát nhau.
- Các bản sao dữ liệu được trải qua nhiều tile.
- Nội dung trong `interleaver_report.txt`.

Kết quả checkwork:

```text
Y - task5_interleaver
```

---

## Task 6 — Nhúng ECC watermark

Task này nhúng watermark đã mã hóa ECC vào ảnh bằng DCT.

Chạy:

```bash
python3 embed_ecc_watermark.py --alpha 34
eog stego_ecc.png &
cat embed_ecc_report.txt
```

Sinh viên cần quan sát:

- `stego_ecc.png`: ảnh sau khi nhúng watermark.
- PSNR sau khi nhúng.
- Clean BER.
- Số block được sử dụng.

Kết quả checkwork:

```text
Y - task6_embed_ecc
```

---

## Task 7 — Tạo tấn công hình học

Task này tạo hai dạng tấn công hình học: crop-shift và rotation-scale.

Chạy crop-shift attack:

```bash
python3 simulate_geometric_damage.py --mode crop_shift --crop 18 --dx 9 --dy -6
eog attack_crop_shift.png &
cat crop_shift_attack_report.txt
```

Chạy rotation-scale attack:

```bash
python3 simulate_geometric_damage.py --mode rotate_scale --angle 4 --scale 0.94
eog attack_rotation_scale.png &
cat rotation_scale_attack_report.txt
```

Sinh viên cần quan sát:

- `attack_crop_shift.png`.
- `attack_rotation_scale.png`.
- Ảnh vẫn có thể nhìn bình thường nhưng watermark đã bị ảnh hưởng.
- Nội dung trong các file report.

Kết quả checkwork:

```text
Y - task7_geometric_damage
```

---

## Task 8 — Extract raw bits

Task này trích xuất bit thô từ ảnh đã bị tấn công, chưa dùng ECC recovery.

Chạy với ảnh crop-shift:

```bash
python3 extract_raw_bits.py --image attack_crop_shift.png --out raw_crop_report.txt
eog raw_bit_error_map.png &
cat raw_crop_report.txt
```

Chạy với ảnh rotation-scale:

```bash
python3 extract_raw_bits.py --image attack_rotation_scale.png --out raw_rot_report.txt
cat raw_rot_report.txt
```

Sinh viên cần quan sát:

- Raw BER sau tấn công.
- `raw_bit_error_map.png`: bản đồ lỗi bit.
- Lỗi thường tập trung theo vùng.
- Một số tile bị lỗi nặng hơn các tile khác.

Kết quả checkwork:

```text
Y - task8_raw_extract
```

---

## Task 9 — Phát hiện erasure blocks

Task này phát hiện block không đáng tin cậy bằng confidence score và coefficient margin.

Mở file:

```bash
nano recovery_config.py
```

Sửa thành:

```python
ERASURE_MODE = "coeff_margin"
CONFIDENCE_THRESHOLD = 0.58
MIN_COEFF_MARGIN = 6.5
TILE_WEIGHT_MODE = "survival_confidence"
DROP_WORST_TILES = 3
LANE_VOTING = "weighted"
MIN_TILE_SURVIVAL = 0.42
```

Chạy:

```bash
python3 detect_erasure_blocks.py --image attack_crop_shift.png
eog erasure_confidence_map.png &
eog tile_survival_map.png &
cat erasure_report.txt
```

Sinh viên cần quan sát:

- `erasure_confidence_map.png`: bản đồ độ tin cậy.
- `tile_survival_map.png`: bản đồ tile còn sống.
- Block yếu được đánh dấu là erasure.
- Tile nào bị hỏng nặng.

Kết quả checkwork:

```text
Y - task9_erasure_detection
```

---

## Task 10 — Decode bằng ECC

Task này dùng ECC để khôi phục thông điệp từ raw bits và erasure blocks.

Chạy:

```bash
python3 decode_with_ecc.py --raw raw_crop_report.txt --erasure erasure_report.txt
cat ecc_decode_report.txt
eog ecc_syndrome_table.png &
```

Sinh viên cần quan sát:

- Corrected BER.
- Số lỗi được ECC sửa.
- Thông điệp khôi phục.
- So sánh raw BER và corrected BER.

Kết quả checkwork:

```text
Y - task10_ecc_decode
```

---

## Task 11 — Adaptive tile recovery

Task này khôi phục watermark từ ảnh rotation-scale bằng cách ưu tiên tile tốt và loại bỏ tile xấu.

Chạy:

```bash
python3 adaptive_tile_recovery.py --image attack_rotation_scale.png
eog tile_survival_map.png &
cat adaptive_recovery_report.txt
```

Sinh viên cần quan sát:

- Tile survival rate.
- Tile nào được giữ lại.
- Tile nào bị loại bỏ.
- Recovered BER trong `adaptive_recovery_report.txt`.

Kết quả checkwork:

```text
Y - task11_adaptive_recovery
```

---

## Task 12 — Challenge recovery

Task này tạo ảnh challenge với dạng phá hoại hình học chưa biết trước, sau đó chạy pipeline khôi phục.

Tạo challenge:

```bash
python3 simulate_geometric_damage.py --mode challenge
eog challenge_damage.png &
cat challenge_public_report.txt
```

Chạy pipeline recovery:

```bash
python3 extract_raw_bits.py --image challenge_damage.png --out challenge_raw_report.txt
python3 detect_erasure_blocks.py --image challenge_damage.png --out challenge_erasure_report.txt
python3 adaptive_tile_recovery.py --image challenge_damage.png --out challenge_recovery_report.txt
cat challenge_recovery_report.txt
```

Mở ảnh quan sát:

```bash
eog raw_bit_error_map.png &
eog erasure_confidence_map.png &
eog tile_survival_map.png &
```

Sinh viên cần quan sát:

- `challenge_damage.png`: ảnh challenge.
- Raw bit error map.
- Erasure confidence map.
- Tile survival map.
- Kết quả trong `challenge_recovery_report.txt`.

Kết quả checkwork:

```text
Y - task12_challenge_recovery
```

---

## Task 13 — Tạo bằng chứng trực quan

Task này tạo contact sheet tổng hợp các ảnh quan trọng.

Chạy:

```bash
python3 visualize_damage_maps.py
eog recovery_contact_sheet.png &
cat visual_report.txt
```

Sinh viên cần quan sát:

- `recovery_contact_sheet.png`.
- Ảnh gốc, ảnh stego, ảnh bị attack.
- Bản đồ DCT energy.
- Bản đồ block mask.
- Bản đồ lỗi raw bit.
- Bản đồ erasure và tile survival.

Kết quả checkwork:

```text
Y - task13_visual_evidence
```

---

## Task 14 — Viết Recovery Memo

Task này yêu cầu sinh viên viết memo phân tích kết quả.

Mở file:

```bash
nano RECOVERY_MEMO.md
```

Điền nội dung tối thiểu:

```markdown
# Recovery Memo

## Why raw extraction failed

Raw BER increased because crop-shift and rotation-scale attacks damaged groups of DCT blocks. The raw extractor still tried to read every block, so many damaged blocks became wrong bits.

## How interleaving reduced burst errors

Interleaving spread adjacent encoded bits and repeated copies across distant tiles. This changed burst damage into scattered errors that ECC and voting could recover.

## How erasure detection improved recovery

The erasure detector used coefficient margin and confidence score to reject unreliable blocks. This reduced Corrected BER because weak blocks were treated as erasures instead of guessed bits.

## Why some tiles were dropped

Tiles with low Tile survival rate were likely cropped, padded, or heavily distorted. Dropping bad tile regions prevented them from dominating weighted recovery.

## Remaining weaknesses

The method is still weak against very large crop, severe rotation, perspective warp, or attacks that damage most tiles. It does not perform registration or geometric alignment.
```

Validate:

```bash
python3 write_recovery_memo.py
cat memo_report.txt
```

Sinh viên cần đảm bảo:

- Không còn TODO.
- Có đủ các mục phân tích.
- Có giải thích vì sao raw extraction thất bại.
- Có giải thích vai trò của interleaving, erasure detection và tile recovery.
- Có nêu điểm yếu còn lại.

Kết quả checkwork:

```text
Y - task14_memo
```

---

## Task 15 — Tổng kết và checkwork

Task cuối tạo báo cáo tổng hợp và chạy checkwork.

Chạy:

```bash
python3 summary.py
cat summary_report.txt
checkwork
```

`summary_report.txt` sẽ tổng hợp:

- PSNR.
- Clean BER.
- Raw BER.
- Corrected BER.
- Erasure rate.
- Tile survival rate.
- Challenge recovery result.
- Trạng thái memo.
- Trạng thái visual evidence.

Kết quả checkwork:

```text
Y - task15_summary
```

---

# Checkwork

Sau khi hoàn thành đầy đủ, chạy:

```bash
checkwork
```

Kết quả mong đợi:

```text
Labname dct-ecc-erasure-defense

Y - task1_scene
Y - task2_dct_capacity
Y - task3_block_mask
Y - task4_ecc_design
Y - task5_interleaver
Y - task6_embed_ecc
Y - task7_geometric_damage
Y - task8_raw_extract
Y - task9_erasure_detection
Y - task10_ecc_decode
Y - task11_adaptive_recovery
Y - task12_challenge_recovery
Y - task13_visual_evidence
Y - task14_memo
Y - task15_summary
```

---

# Dừng lab

Sau khi hoàn thành:

```bash
stoplab dct-ecc-erasure-defense
```

Kết quả checkwork sẽ được lưu trong:

```text
~/labtainer_xfer/dct-ecc-erasure-defense
```

---

# Thông tin học phần

Hoàng Anh Khoa — B22DCAT164

Lớp: D22CQAT04-B

Học phần: Kỹ thuật giấu tin (INT14102)

Học viện Công nghệ Bưu chính Viễn thông (PTIT)

Giảng viên hướng dẫn: PGS.TS. Đỗ Xuân Chợ
