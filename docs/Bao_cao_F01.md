BÁO CÁO BÀI KIỂM TRA SỐ 2

Môn EEE703128: Nhập môn Thiết kế Vi mạch Tích hợp

Mã đề F01 - Nhóm F - Mạch tuần tự

| Thông tin | Nội dung |
| --- | --- |
| Sinh viên | Đặng Việt Quang |
| Mã số sinh viên | 24101108 |
| Lớp | N01 |
| Giảng viên hướng dẫn | Thầy Đinh Văn Nam |
| Ngày kiểm tra | 28/10/2026 |
| Top RTL | f01_traffic_fsm_explicit |
| Cấu hình mặc định | PHASE_TICKS = 4; NIGHT_TICKS = 2 |

## Cổng A và đường dẫn kiểm tra

Repo GitHub: Sẽ bổ sung sau.

GitHub Actions run: Sẽ bổ sung sau.

```text
Commit mã nguồn và bằng chứng:
6dca91d1a0f2429bcc04390b00eeb04c6aa68f82
```

Trạng thái: đã kiểm tra RTL, SAT và tổng hợp logic cục bộ. Cổng A hoàn tất khi repo và CI có URL thật, mở được.

Flow vật lý đã tới CTS và chưa hoàn tất. Phần 10 trình bày quy trình, tiến độ và hướng xử lý.

Hà Nội, tháng 10 năm 2026

Trang thông tin đề tài F01, sinh viên Đặng Việt Quang, MSSV 24101108, lớp N01. Repo GitHub và CI sẽ bổ sung sau.

# 1 Mục tiêu dự án


<!-- Trang 3 -->

Trong hệ thống thực, F01 nằm giữa tầng tạo tín hiệu điều khiển và tầng mạch lái đèn. Tầng trên cung cấp clock, reset, tín hiệu cho phép và lựa chọn chế độ đêm. Tầng dưới nhận các mức logic để điều khiển mạch công suất. Lõi F01 không trực tiếp cấp dòng cho đèn giao thông.

Tầng trên cần khối tạo đúng thứ tự pha, giữ đúng thời lượng, cho phép dừng cập nhật và chuyển sang nhấp nháy vàng. Mã pha được xuất ra để quan sát. Yêu cầu an toàn trọng tâm là hai hướng không đồng thời có đèn xanh ở giá trị logic ổn định.

Hình: Hình 1. Vị trí của lõi F01 trong hệ thống điều khiển đèn.

# 2 Bản chất mạch

Đây là mạch tuần tự vì kết quả phụ thuộc lịch sử lưu trong state, pc, nc và night_phase. Các thanh ghi cập nhật tại cạnh lên clk. Logic tổ hợp tính next và giải mã uo_out. Mô phỏng và tổng hợp dùng cùng module f01_traffic_fsm_explicit để tránh kiểm một module nhưng đo kết quả của module khác [2].

## Cấu hình và thời gian

Đặt P = PHASE_TICKS và N = NIGHT_TICKS. Cấu hình mặc định là P = 4, N = 2. Một tick là cạnh lên được phép cập nhật khi ena = 1. Mỗi pha ban ngày dài P tick; một vòng bốn pha dài 4P tick. Mỗi nửa chu kỳ nhấp nháy dài N tick.

P và N phải là số nguyên dương. Đây là số tick kiểm thử, chưa phải số giây vận hành thực tế. Bộ chia tần và việc đồng bộ các tín hiệu ngoài lõi thuộc tầng tích hợp.

# 2 Sơ đồ trạng thái và hoạt động


<!-- Trang 4 -->

Hình: Hình 2. Bốn pha và điều kiện chuyển trạng thái của F01.

| Trạng thái | Mã | H1 (R,Y,G) | H2 (R,Y,G) | uo_out |
| --- | --- | --- | --- | --- |
| S_NS_G | 00 | 001 | 100 | 0x21 |
| S_NS_Y | 01 | 010 | 100 | 0x62 |
| S_EW_G | 10 | 100 | 001 | 0x8C |
| S_EW_Y | 11 | 100 | 010 | 0xD4 |

Ở ban ngày, pc đếm từ 0 đến P - 1. Khi đủ ngưỡng và ena = 1, state chuyển sang pha kế tiếp, pc về 0. Với P = 4, một vòng đủ bốn pha có 16 tick. Pha vàng xuất hiện sau pha xanh của từng hướng.

Chế độ đêm là lớp phủ ngõ ra, không phải trạng thái pha thứ năm. Khi night_mode = 1, state giữ nguyên; pc về 0 tại cạnh được phép. Hai đèn vàng cùng sáng khi night_phase = 1 và cùng tắt khi bằng 0. Hai bit uo_out[7:6] vẫn phản ánh pha đang giữ.

Khi trở lại ban ngày, mạch tiếp tục pha cũ và đếm lại thời lượng từ đầu. Phần thời gian trước khi vào đêm không được cộng tiếp. Khi ena = 0, mọi thanh ghi giữ nguyên, nhưng đổi night_mode vẫn có thể đổi ngõ ra vì có đường logic tổ hợp trực tiếp.

# 3 Rủi ro kỹ thuật và kiểm soát


<!-- Trang 5 -->

| Rủi ro | Cách kiểm tra |
| --- | --- |
| Sai thời điểm chuyển pha | Đối chiếu state và pc ở ngưỡng P - 1; thử P = 1, 3, 5. |
| Reset hoặc enable sai ưu tiên | Reset đồng thời ena = 0; kiểm cả bốn thanh ghi. |
| Sai khi vào hoặc rời đêm | Vào đêm giữa pha; kiểm giữ state, xóa pc và đếm lại khi rời đêm. |
| Chỉ khóa một phần mạch | Khi ena = 0, kiểm state, pc, nc và night_phase ở cả hai chế độ. |
| Kết luận sai từ hình hoặc log | Đối chiếu lệnh, top, tham số và file gốc của từng kết quả. |

Reset rst_n tích cực thấp và đồng bộ vì always_ff chỉ nhạy với posedge clk. Reset được xét trước ena nên vẫn có tác dụng khi ena = 0. Mã RTL dùng đủ bốn mã nhị phân cho bốn pha; một lỗi lật bit có thể chuyển sang pha hợp lệ khác, nên nhánh default không tạo bảo vệ lỗi phần cứng.

# 4 Trọng tâm phân tích

## Số bit tối thiểu và đối chiếu flip flop

Bốn trạng thái cần tối thiểu ceil(log2(4)) = 2 bit để mã hóa. Tổng số phần tử nhớ còn bao gồm các bộ đếm và bit nhấp nháy, không chỉ thanh ghi pha.

| Thành phần | Độ rộng mặc định | Chức năng |
| --- | --- | --- |
| state | 2 bit | Lưu một trong bốn pha. |
| pc | 2 bit | Đếm 0 đến 3. |
| nc | 1 bit | Đếm 0 đến 1. |
| night_phase | 1 bit | Nhớ nửa chu kỳ nhấp nháy. |
| Tổng | 6 bit | Đối chiếu 6 FF trong ảnh tổng hợp. |

Ảnh ở trang 9 ghi 6 cell $_SDFFE_PN0P_, phù hợp tổng dự đoán. Netlist tham chiếu trong 06_GDSII/synthesis/netlist.json có hai bit state nối tới Q của hai FF. Log FSM_DETECT của bộ tham chiếu ghi state nối ra cổng và không thấy lợi ích đổi mã [3].

Với mã hóa one-hot, riêng pha cần bốn FF thay vì hai; tổng dự tính là tám bit ở cùng bộ đếm. Vì vậy, 6 FF toàn lõi không có nghĩa FSM dùng sáu bit pha. Số cell vật lý và lợi ích timing của cách mã hóa cần đọc ở báo cáo tổng hợp đúng thư viện.

# 5 Giao diện và sơ đồ chân


<!-- Trang 6 -->

Wrapper tt_um_f01 nối night_mode với uio_in[0] theo charter. Sáu đèn nằm ở uo_out[5:0], mã pha nằm ở uo_out[7:6]. Mạch sử dụng một chân dữ liệu vào và tám chân ra, tổng 9 trên 24 chân dữ liệu. Clock, reset và ena là các chân điều khiển riêng [1, 2].

| Tín hiệu | Chiều | Rộng | Mức tích cực và giả định |
| --- | --- | --- | --- |
| clk | Vào | 1 | Cập nhật tại cạnh lên. |
| rst_n | Vào | 1 | Thấp, reset đồng bộ, ưu tiên hơn ena. |
| ena | Vào | 1 | Cao, cho phép cập nhật thanh ghi. |
| uio_in[0] | Vào | 1 | Cao: chế độ đêm; thấp: ban ngày. |
| uo_out[2:0] | Ra | 3 | Đèn H1 theo thứ tự (R,Y,G), tích cực cao. |
| uo_out[5:3] | Ra | 3 | Đèn H2 theo thứ tự (R,Y,G), tích cực cao. |
| uo_out[7:6] | Ra | 2 | Mã pha 00, 01, 10, 11. |
| ui_in[7:0] | Vào | 8 | Không sử dụng. |
| uio_in[7:1] | Vào | 7 | Không sử dụng. |
| uio_out[7:0] | Ra | 8 | Gán 0; không lái ra chân khi oe = 0. |
| uio_oe[7:0] | Ra | 8 | Gán 0, đặt toàn bộ uio ở chiều vào. |

## Ánh xạ sáu đèn

| Bit | 0 | 1 | 2 | 3 | 4 | 5 |
| --- | --- | --- | --- | --- | --- | --- |
| Chức năng | H1 xanh | H1 vàng | H1 đỏ | H2 xanh | H2 vàng | H2 đỏ |

## Phụ thuộc Moore và Mealy

Ở ban ngày, các đèn được giải mã từ state nên có đặc điểm Moore. Khi xét toàn lõi, uo_out[5:0] còn phụ thuộc trực tiếp night_mode trong logic tổ hợp nên có đặc điểm Mealy. Riêng uo_out[7:6] chỉ phản ánh state. Vì vậy, ena = 0 không đồng nghĩa mọi ngõ ra bị khóa.

Đầu vào phải có giá trị 0 hoặc 1 ổn định khi lấy mẫu. Tín hiệu chế độ đêm từ công tắc hoặc cảm biến cần được đồng bộ trước khi nối vào lõi. RTL chưa mô tả bộ đồng bộ, mạch lọc nhiễu hoặc mạch công suất.

# 6 Ca kiểm thử và kết quả


<!-- Trang 7 -->

Testbench điều khiển tín hiệu tại cạnh xuống, kiểm ngõ ra trước cạnh lên và kiểm thanh ghi sau cạnh lên. Cách này tránh tranh chấp với cập nhật thanh ghi. Các giá trị mong đợi được sinh bởi mô hình chuẩn Python; wrapper được đối chiếu với lõi tại mỗi vector.

| Ca kiểm thử | Nội dung và lỗi được phát hiện |
| --- | --- |
| Bắt buộc 1 | Chạy đủ các pha, phát hiện sai thứ tự và sai thời lượng. |
| Bắt buộc 2 | Vào đêm giữa pha rồi trở lại ngày, phát hiện mất pha hoặc cộng tiếp sai pc. |
| Bắt buộc 3 | Kiểm hai đèn xanh không cùng bằng 1. |
| Bổ sung 1, ca biên | P = N = 1 và P = 3, 5: bắt lỗi độ rộng đếm, ngưỡng và tràn tự nhiên. |
| Bổ sung 2 | ena = 0 ở ngày và đêm: bắt lỗi bộ đếm vẫn chạy khi state bị giữ. |
| Bổ sung 3 | Reset khi ena = 0 và khi đang đêm: bắt lỗi ưu tiên hoặc giữ sót thanh ghi. |
| Bổ sung 4 | Đổi reset giữa hai cạnh: phân biệt reset đồng bộ và bất đồng bộ. |
| Bổ sung 5 | 600 vector giả ngẫu nhiên mỗi cấu hình, seed cố định: kiểm tương tác giữa các điều khiển. |

## Cấu hình của bộ kiểm thử

| P | N | Số vector của bộ kiểm thử |
| --- | --- | --- |
| 1 | 1 | 640 |
| 2 | 2 | 659 |
| 3 | 3 | 678 |
| 4 | 2 | 2862 |
| 5 | 3 | 705 |
| 8 | 4 | 752 |

Bộ kiểm thử có tổng 6296 vector. Với P = 4, N = 2, mô hình có 28 trạng thái mở rộng (state, pc, nc, night_phase) và 224 cạnh khi xét các tổ hợp reset, enable, chế độ. Đây không phải 28 pha đèn. Các ca phát lại từ reset dùng để kiểm đủ các cạnh này.

Log tham chiếu ngày 28/10 ghi sáu cấu hình PASS. Các ảnh thao tác ngày 30/10 minh chứng việc quan sát dạng sóng và chạy SAT trên Ubuntu. Kết quả toàn bộ ca kiểm thử của lần chạy mới cần được đối chiếu thêm với thư mục logs trên máy trước khi chốt hồ sơ.

# 6 Minh chứng dạng sóng mô phỏng


<!-- Trang 8 -->

Hình: Hình 3. Ảnh GTKWave trên Ubuntu, vùng 0-700 ns.

Các tín hiệu quan sát gồm clk, ena, nc, night_mode, night_phase, pc, rst_n, state và uo_out. Trong đoạn ban ngày, state lần lượt có 00, 01, 10, 11; uo_out có các mã 0x21, 0x62, 0x8C, 0xD4. Bộ đếm pc được đưa về 0 tại thời điểm chuyển pha.

Ở vùng khoảng 490-570 ns, night_mode lên 1, state giữ tại 01, pc về 0. Hai đèn vàng sáng và tắt theo night_phase. Mã 0x52 là hai đèn vàng sáng, mã 0x40 là đèn tắt; cả hai vẫn có mã pha 01 ở hai bit cao.

Khi night_mode trở về 0, uo_out trở lại mẫu ban ngày của pha đang giữ. pc tiếp tục đếm từ đầu trước khi chuyển sang pha kế tiếp. Ảnh này giúp đối chiếu việc giữ pha và bộ đếm; kết quả đúng của toàn bộ ca kiểm thử cần đọc cùng log và VCD.

## File và lệnh quan sát

```text
bash 04_IMPLEMENTATION/scripts/01_run_sim.sh
gtkwave 05_VERIFICATION/waveforms/p4_n2.vcd
```

Cấu hình của hình là P = 4, N = 2. File p4_n2.vcd lưu đầy đủ diễn biến tín hiệu; trace CSV tương ứng nằm trong 05_VERIFICATION/traces. Ảnh minh chứng gốc được lưu trong docs/minh_chung của bộ mã nguồn.

# 6 Minh chứng kiểm thuộc tính an toàn


<!-- Trang 9 -->

Ảnh dưới đây ghi lệnh kiểm SAT trên Ubuntu, kết quả SUCCESS và một phần bảng tổng hợp có 6 FF. Đây là minh chứng trực tiếp của lần thao tác ngày 30/10/2026.

Hình: Hình 4. Kết quả SAT SUCCESS và dòng thống kê 6 flip-flop.

Đặt G1 = uo_out[0], G2 = uo_out[3], M = night_mode, s1 = state[1], s0 = state[0]. Từ bảng giải mã: G1 = (1-M)(1-s1)(1-s0), G2 = (1-M)s1(1-s0). Tích G1*G2 bằng 0 vì (1-s1)*s1 = 0. SAT kiểm cùng thuộc tính safe = !(G1 && G2).

Kết luận áp dụng cho giá trị logic nhị phân ổn định. Phép kiểm này không đánh giá trễ cổng, glitch, metastability hoặc an toàn toàn bộ tủ điều khiển thực tế.

# 7 Phạm vi dự án


<!-- Trang 10 -->

Trong phạm vi gồm đặc tả, RTL, kiểm thử, dạng sóng, tổng hợp và thử flow vật lý. Các nội dung cảm biến xe, đèn bộ hành, ưu tiên xe khẩn cấp, phối hợp nhiều nút giao, chế tạo chip và đánh giá tủ đèn thực tế nằm ngoài phạm vi bài này.

# 8 Kế hoạch thực hiện

| Mốc | Nội dung | Trạng thái |
| --- | --- | --- |
| M1 | Chốt giao diện, giả định và FSM. | Đã trình bày. |
| M2 | RTL, ca kiểm thử và dạng sóng. | Có mã, bộ kiểm thử và ảnh GTKWave. |
| M3 | Tổng hợp, thử vật lý và GitHub CI. | Có tổng hợp; flow dừng ở CTS; CI chờ bổ sung. |
| M4 | Đọc báo cáo và giải thích kết quả. | Đã đối chiếu FF; có chẩn đoán lỗi vật lý. |
| M5 | Hoàn thiện báo cáo và vấn đáp. | Báo cáo 13 trang; chờ đường dẫn GitHub và CI. |

Charter chưa ghi hạn cụ thể cho từng mốc. Trình tự công việc phụ thuộc từ đặc tả tới kiểm thử, chạy flow, phân tích và hoàn thiện báo cáo. Các kết quả đạt hoặc chưa đạt phải được ghi cùng điều kiện tạo ra chúng.

# 9 Bàn giao và tiêu chí nghiệm thu

| Sản phẩm | Nội dung bàn giao |
| --- | --- |
| Repo cá nhân và CI | Mã nguồn, lịch sử commit và link một lần chạy Actions cụ thể. |
| RTL và kiểm thử | Lõi FSM, wrapper, testbench, mô hình, vector và mô tả giao diện. |
| Kết quả flow hoặc nhật ký lỗi | Log, VCD, netlist; ảnh lỗi môi trường và lỗi CTS có chẩn đoán. |
| Báo cáo và tái lập | Bản Word, PDF; top, tham số, công cụ, PDK và ngày chạy. |

Cổng A của charter yêu cầu repo cá nhân và link CI mở được. Ảnh chụp mã nguồn hoặc file workflow không thay cho một CI run thực. Nếu flow vật lý chưa hoàn tất, cần nộp nguyên log và giải thích lỗi thay vì ghi kết quả vật lý đạt.

# 10 Kết quả flow RTL to GDSII


<!-- Trang 11 -->

Flow RTL to GDSII chuyển mô tả phần cứng thành bố trí vật lý theo các bước trong Hình 5. Với F01, mô phỏng và tổng hợp logic đã có kết quả; lần thử LibreLane đã tới Clock Tree Synthesis, chưa xác nhận hoàn tất các bước sau.

Hình: Hình 5. Sơ đồ minh họa quy trình RTL to GDSII và tiến độ hiện có.

| Giai đoạn | Đầu ra và trạng thái |
| --- | --- |
| Kiểm RTL | Dạng sóng, log kiểm thử, kết quả SAT; có bằng chứng. |
| Tổng hợp logic | Netlist và thống kê FF; có bộ kết quả tham chiếu. |
| Floorplan và placement | Flow đã tiến qua để tới bước CTS; cần giữ log chi tiết. |
| CTS | Lần chạy dừng do detailed placement không hợp lệ. |
| Routing và signoff | Chưa xác nhận hoàn tất; chưa có GDS được kiểm tra. |

Ảnh log gốc và thông tin lần chạy được giữ tại docs/minh_chung trong bộ mã nguồn để tra cứu. Cần đối chiếu log khi đánh giá từng bước của flow.

# 10 Đánh giá kết quả và hướng hoàn thiện


<!-- Trang 12 -->

## Tiến độ flow vật lý

Lần thử đầu chưa tìm thấy module librelane nên dừng trước khi đọc thiết kế. Sau khi mở môi trường công cụ, flow đã tiến tới CTS. Hai lỗi thuộc hai giai đoạn khác nhau: lỗi đầu là thiếu môi trường, lỗi sau xuất hiện trong quá trình thiết kế vật lý.

Một khả năng cần kiểm tra là floorplan của lõi nhỏ chưa thuận lợi cho việc đặt hợp lệ các cell sau CTS. Phương án thử là dùng FP_SIZING = relative, giảm FP_CORE_UTIL xuống 20 và đặt PL_TARGET_DENSITY_PCT = 50. Thay đổi này giữ nguyên RTL và clock 10 ns. Chưa có kết quả xác nhận lần chạy lại đã qua CTS.

## Giới hạn của số liệu hiện có

| Nhóm kết quả | Tình trạng và cách đọc |
| --- | --- |
| Synthesis | Ảnh có 6 FF. Bộ tham chiếu có 53 cell tổng quát; không dùng như số standard cell SKY130. |
| STA | Chưa có báo cáo đầy đủ để chốt startpoint, endpoint, setup, hold, slack, WNS hoặc TNS. |
| Power | Chưa có báo cáo kèm góc PVT và giả định hoạt động để kết luận công suất. |
| CTS | Flow đã tới CTS rồi lỗi. Chưa xác nhận số buffer, skew và insertion delay của kết quả hoàn tất. |
| DRC và LVS | Chưa có báo cáo cuối để kết luận số lỗi hoặc độ khớp kết nối. |
| Layout và GDS | Chưa có kết quả xác nhận hoàn tất routing và tạo GDS. |

Các đường có thể cần phân tích gồm FF bộ đếm qua so sánh ngưỡng tới FF pha, FF pha qua giải mã tới đèn, và night_mode qua lớp phủ tới ngõ ra. Đường tới hạn thực phải lấy từ STA của lần chạy cụ thể. CLOCK_PERIOD là giả định phân tích, không tự xác nhận giới hạn tốc độ của mạch.

Số cell sau layout có thể tăng do buffer, cây clock, tap và endcap. Diện tích standard cell khác diện tích core hoặc die. Khi có báo cáo cần ghi rõ bước đo, thư viện và ràng buộc để việc so sánh có ý nghĩa.

## Lệnh và vị trí bằng chứng

```text
bash 04_IMPLEMENTATION/scripts/02_run_synth.sh
bash 04_IMPLEMENTATION/scripts/03_check_safety.sh
bash 04_IMPLEMENTATION/scripts/04_run_physical.sh
```

Kết quả logic nằm trong 06_GDSII/synthesis. Log thử vật lý nằm trong 06_GDSII/physical; các file chi tiết của lần chạy mới ở thư mục runs dưới 04_IMPLEMENTATION/scripts trên máy Ubuntu. Ảnh chụp không thay thế toàn bộ các file raw của lần chạy đó.

# 11 Kết luận


<!-- Trang 13 -->

F01 mô tả được bốn pha có đèn vàng và chế độ đêm nhấp nháy vàng. Ảnh GTKWave thể hiện chuỗi pha và việc giữ pha khi vào đêm. Ảnh tổng hợp có 6 FF, phù hợp hai bit pha và bốn bit cho các bộ đếm, bit nháy. SAT đạt thuộc tính hai hướng không cùng xanh ở logic ổn định.

Flow vật lý đã tiến tới CTS rồi dừng ở DPL-0036. Chưa xác nhận kết quả chạy lại, timing vật lý, công suất, DRC/LVS hoặc GDS. Hồ sơ cần bổ sung repo, CI và các file kết quả mới để khép thông tin tái lập.

# 12 Danh mục tự kiểm

| Nội dung | Tình trạng |
| --- | --- |
| Giao diện, FSM và câu phân tích FF | Đã trình bày. |
| Ca bắt buộc và ca thêm có ca biên | Đã mô tả, kèm bộ kiểm thử. |
| Dạng sóng, SAT và tiến độ flow | Có ảnh thực tế và sơ đồ quy trình. |
| Repo, CI run và log mới | Chờ bổ sung và đối chiếu trước khi nộp. |

## Phụ lục thông tin tái lập

| Hạng mục | Nội dung |
| --- | --- |
| Top và tham số | f01_traffic_fsm_explicit; P = 4, N = 2. |
| Clock và IO delay | Clock 10 ns; input delay và output delay 1 ns. |
| Môi trường ngày 30/10 | Ubuntu WSL 2; cần bổ sung phiên bản công cụ trong log môi trường. |
| LibreLane | Cần bổ sung kết quả lệnh librelane --version. |
| PDK trong ảnh log | sky130A; hash ghi ở đoạn dưới. |
| Repo GitHub | Sẽ bổ sung sau. |
| CI run | Sẽ bổ sung sau. |

```text
PDK: 8afc8346a57fe1ab7934ba5a6056ea8b43078e71
Mã tham chiếu: 6dca91d1a0f2429bcc04390b00eeb04c6aa68f82
```

## Tài liệu tham khảo

[1] Charter F01, EEE703128, bản HTML giảng viên cung cấp.

[2] RTL, testbench và mô hình trong bộ F01; nguồn gốc ở 08_ORIGINAL_SOURCE.

[3] synth_stat.txt, netlist.json và yosys.log của bộ tham chiếu ngày 28/10/2026.

[4] Ảnh GTKWave, SAT và lỗi CTS trên Ubuntu ngày 30/10/2026, docs/minh_chung.
