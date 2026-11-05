# PV Dust Economics — Ngưỡng kinh tế vệ sinh pin mặt trời

**Website công khai:** https://nguyenthihongtham01012005-code.github.io/pv-dust-economics/

Website hỗ trợ **đánh giá kịch bản hòa vốn** khi vệ sinh pin mặt trời, dựa trên suy giảm công suất đo ở tải cố định và sản lượng mô phỏng PVGIS/PVsyst. **Không** xác định chu kỳ vệ sinh tối ưu hay tuyên bố đã đo tổn thất năng lượng trên hệ PV có MPPT.

## 1. Dữ liệu đang có và mức độ tin cậy

| Thành phần | Thông tin | Phạm vi diễn giải |
|---|---|---|
| Thực nghiệm Data(8).xlsx | 3 mức bức xạ: 300, 670, 900 W/m²; 7 mức bụi: 0, 1, 2, 3, 4, 5, 10 g/m² | 21 nhóm, mỗi nhóm 40 lượt đọc U×I liên tiếp; không phải 40 lần thí nghiệm độc lập |
| Công suất thực nghiệm | P = U × I tại tải cố định | **Không được gọi là Pmp**, không áp dụng trực tiếp tỷ lệ suy hao này cho năng lượng MPPT mà không hiệu chuẩn |
| PVGIS giờ mặc định | Năm 2023, mô phỏng hệ 100 kWp | Mô hình giờ: P (W) và G trên mặt phẳng tấm pin; không phải số đo vận hành |
| PVGIS tháng mặc định | Sản lượng theo tháng từ một bộ thống kê nhiều năm | Khác phạm vi với dữ liệu giờ năm 2023; không ghép hai bộ thành một chuỗi |
| Chi phí vệ sinh, giá trị điện, η | Tham số tương tác người sử dụng | **Kịch bản**, không phải báo giá và hiệu quả vệ sinh đã được chứng nhận |

Hai bảng CSV có thể tải trực tiếp từ repo:
- [840 giá trị công suất U×I lưu trong website](data/measurements_840_readings.csv) — các giá trị công suất đã tổng hợp, **không chứa cặp U và I gốc**.
- [Bảng 21 nhóm: n, trung bình, độ lệch chuẩn và L(%) so với 0 g](data/loss_summary_21_groups.csv).

## 2. Mô hình được sử dụng

Tại mỗi mức bức xạ đo G, so sánh công suất trung bình có bụi với công suất trung bình sạch cùng mức:

```text
L(d,G) = 100 × [1 − P_mean(d,G) / P_mean(0,G)]  (%)
```

Chỉ **nội suy tuyến tính theo G trong [300, 900] W/m²** tại các mức bụi đã đo. Với dữ liệu PVGIS theo giờ, chỉ những giờ nằm trong miền đó được tính:

```text
G_t = Gb(i)_t + Gd(i)_t + Gr(i)_t
E_loss(d) = Σ_t∈miền [E_clean,t × max(0,L(d,G_t))/100]    (kWh)
E_restored(d) = η × E_loss(d)                            (kWh)
B(d) = E_restored(d) × p                                (VNĐ)
C = đơn_giá_vệ_sinh × công_suất_hệ                      (VNĐ)
C_max(d) = B(d) / công_suất_hệ                          (VNĐ/kWp/lần)
L* = 100 × C / [E_trong_miền × p × η]                   (%)
```

Ở đây `η` là **phần tổn thất do bụi được giả định khôi phục**; `p` là giá trị kinh tế của 1 kWh, không nhất thiết là biểu giá bán điện. Công thức được triển khai như **một kịch bản quy đổi**: suy hao công suất ở tải cố định chưa được xác minh bằng suy hao sản lượng MPPT.

Trường hợp chỉ nhập 12 tháng sản lượng, web có **chế độ bức xạ đại diện** G = 300 / 670 / 900 W/m². Đây là giả định áp một tỷ lệ suy hao lên toàn kỳ, không thay thế phân bố G theo giờ. Dữ liệu ngoài miền G=300–900 W/m² không được ngoại suy.

**Tương thích công suất hệ:** Nếu công suất hệ nhập khác công suất mà tệp PVGIS/PVsyst dùng để mô phỏng (khi metadata có giá trị), giao diện cảnh báo và không coi phép tính kinh tế là hợp lệ. Không tự nhân sản lượng lên một quy mô khác mà chưa hiệu chuẩn.

## 3. Kiểm tra và giới hạn khoa học

Trang có bảng kiểm tra tự động: đủ 21 nhóm; có chuỗi công suất để đối chiếu trung bình/SD hay không; công suất hệ và đầu vào; cân bằng điện năng trong/ngoài miền; nhất quán công thức giá trị điện và chi phí.

- **Đã đo:** quan hệ bụi → P=U×I ở ba mức G trong thí nghiệm.
- **Đã mô phỏng:** sản lượng PVGIS/PVsyst. PVGIS có thể được cấu hình tổn thất hệ thống tổng hợp 14%; phải xác minh cách hiểu phần bụi trong cấu hình để tránh cộng trùng.
- **Giả định:** tỷ lệ hồi phục sau vệ sinh η, giá trị điện, chi phí vệ sinh, duy trì mức suy hao suốt kỳ H.
- **Chưa xác định:** tốc độ tích bụi g/m²/ngày, số ngày để đạt ngưỡng, mất điện năng đo trên hệ MPPT, thời gian vệ sinh tối ưu.
- Độ lệch chuẩn của 40 lần đọc liên tiếp **không** phải độ không chắc chắn của 40 thí nghiệm độc lập. Nếu muốn ước lượng sai số giữa lần lặp, cần các lần thí nghiệm độc lập hơn.

## 4. Kiểm soát nền sản lượng sạch và độ biến động thí nghiệm (bản bổ sung)

- Website có mục **Kiểm tra nền sản lượng và điều kiện quy đổi** với ba lựa chọn: *chưa xác minh nguồn sạch* (mặc định), *người dùng khai báo đã có căn cứ riêng xem nguồn là sạch*, và *nguồn đã gồm tổn thất bụi*. Trường hợp cuối **khóa ngưỡng hòa vốn và báo giá tối đa**, vì không được cộng trùng suy hao bụi. Trường hợp chưa xác minh vẫn hiển thị kết quả để khảo sát **kịch bản toán học**, nhưng không xác nhận lợi ích phục hồi thực tế.
- PVGIS/JRC xác nhận đầu ra hệ thống đã có tổn thất tổng hợp và 14% là giá trị mặc định, **có thể gồm bụi**. Không được mặc nhiên khấu trừ toàn bộ 14% hay lấy toàn bộ 14% là soiling loss. Phải đối chiếu cấu hình sản lượng sạch của từng tệp: [PVGIS user manual](https://joint-research-centre.ec.europa.eu/photovoltaic-geographical-information-system-pvgis/using-pvgis-5/pvgis-5-user-manual_en).
- Đồ thị **Trung bình ± 1 SD** tại G = 300 / 670 / 900 W/m² vẽ trên đúng 7 mức bụi với P = U×I và SD tương ứng. Đây là **dao động mô tả trong chuỗi 40 lượt đọc liên tiếp**, không phải 95% CI hoặc 40 phép lặp độc lập.
- Ngưỡng hòa vốn L* là **điều kiện kinh tế cho kỳ H** được chọn, với giả định suy hao và tỷ lệ khôi phục không đổi trong kỳ. Phân tích H = 7 / 15 / 30 / 60 / 90 ngày chỉ là thử độ nhạy, **không** dự báo bụi tích tụ theo ngày hay chu kỳ vệ sinh.
- Khi chuyển giữa nguồn sản lượng giờ/tháng hoặc nạp tệp mới, trạng thái kiểm chứng nền sạch tự quay về *chưa xác minh*. Trạng thái đầu vào được lưu cùng dự án JSON/CSV báo cáo. Dù nền sạch được người dùng xác nhận, dữ liệu U×I tải cố định **chưa chứng minh** suy hao Pmp hay sản lượng AC MPPT.
- Căn cứ soiling: [IEC 61724-1:2021 (IEC)](https://webstore.iec.ch/en/publication/65561), [Monitoring photovoltaic soiling: assessment, challenges and perspectives](https://pmc.ncbi.nlm.nih.gov/articles/PMC7960939/).

## 5. Tài liệu đối chiếu

1. **IEC 61724-1:2021 — Photovoltaic system performance, monitoring** (khái niệm soiling ratio/soiling level). [Bản thông tin tiêu chuẩn](https://ecommerce.sist.si/catalog/standards/sist/c167b9d7-14bf-4ff0-b716-ef6f4aebd61a/sist-en-iec-61724-1-2021). Viện dẫn khái niệm, **không tuyên bố thí nghiệm tải cố định là phép đo đạt IEC**.
2. **Monitoring photovoltaic soiling: assessment, challenges, and perspectives** (2021). [Bài nghiên cứu open access](https://pmc.ncbi.nlm.nih.gov/articles/PMC7960939/).
3. **European Commission JRC — PVGIS hourly radiation** ([mô tả trường dữ liệu](https://joint-research-centre.ec.europa.eu/photovoltaic-geographical-information-system-pvgis/using-pvgis-5/pvgis-5-tools/hourly-radiation_en)).
4. **JRC — PVGIS 5 user manual** ([tổn thất hệ thống PV](https://joint-research-centre.ec.europa.eu/photovoltaic-geographical-information-system-pvgis/using-pvgis-5/pvgis-5-user-manual_en)).
5. **Economics of seasonal photovoltaic soiling and cleaning optimization scenarios**, *Energy* 215 (2021), 119018. [DOI](https://doi.org/10.1016/j.energy.2020.119018). Nghiên cứu lịch vệ sinh cần thông tin biến thiên bám bụi theo thời gian; web hiện không cung cấp phần này.

## 6. Cách xem và tải

Website có biểu đồ suy hao theo G, bảng hòa vốn VNĐ/kWp, phân tích kịch bản trước/sau vệ sinh, độ nhạy giá điện/chi phí/η, bảng kiểm tra dữ liệu, tải/nhập CSV/JSON, và chức năng in/lưu PDF.

Nội dung website là minh họa có điều kiện phục vụ đồ án; không tự động là khuyến nghị vận hành thực địa.
