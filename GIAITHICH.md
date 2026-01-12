# 📋 Giải Thích Các Khối Trong Sơ Đồ Schematic và PCB Layout

---

## 🏗️ Tổng Quan Cấu Trúc PCB 4 Lớp

| Lớp | Tên | Chức năng |
|-----|-----|-----------|
| **Lớp 1** | Top | Lắp các linh kiện chính: Module ESP32-S3, Antenna, Mic, USB-C, ... |
| **Lớp 2** | GND Plane | Đổ full đồng làm mặt phẳng GND |
| **Lớp 3** | Power/Signal | Đi các đường nguồn (3V3, 5V, 1V8, 1V5) và tín hiệu (EN, ...) |
| **Lớp 4** | Bottom | Linh kiện phụ và linh kiện lớn: FPC Connector, LDO 2V8, LDO 1V8 |

---

## 1. 🔌 Khối USB-C (Nạp Code & Cấp Nguồn)

### Schematic

![USB-C Schematic](image/{0264918D-2589-4E4A-B3E9-FD0CA2E88D0B}.png)

### Mô tả

Em sử dụng **USB-C 16P** với các kết nối:

- **Chân CC1 và CC2**: Dùng 2 điện trở **5.1kΩ** kéo xuống GND
- **Chân SHIELD**: Nối thẳng xuống GND

### Layout

![USB-C Layout](image/{B029A4EE-4CA2-4896-96BC-E9080BFC9C97}.png)

#### Chi tiết thiết kế:

- Nối 2 chân **B7** và **A7** lại với nhau thông qua **via Through**
- Dây **D-** và **D+** đi qua:
  1. **ESD9B3.3ST5G** (ESD Protection Diode)
  2. **DLW21SN900SQ2L** (Common Mode Choke) để lọc nhiễu
  3. Cuối cùng đi tới ESP32-S3

![USB Data Lines](image/{3DF13B05-19F0-4A6C-B219-9B7BC6BB070D}.png)

- 2 pad **5V** được nối lại với nhau thông qua via through, đi qua tụ bulk **10µF (C12)** trước khi đi tới khối Buck

> **❓ Câu hỏi:**
> 1. Cách nối B7, A7 lại với nhau thông qua via Through có đúng không ạ ?
> 2. Cách dùng ESD protection diode ESD9B3.3ST5G có đúng không ạ ?
> 3. Cách đi dây đã ổn chưa ạ ?

---

## 2. ⚡ Khối Buck Converter (5V → 3V3)

### Schematic & Layout

![Buck Schematic](image/{B6C93DB4-B525-4A13-9039-084E98C0C294}.png)
![Buck Layout](image/{FF17505C-9F0E-4132-8334-1791E8986DEC}.png)

### Mô tả

- Sử dụng **2 tụ decoupling** ngay sát chân 5V để đón dòng vào Buck
- Đầu ra sử dụng **mạch chia áp** để nhận được 3V3

### LDO 1V8 (Lớp 4 - Bottom)

Ở bên dưới Buck (lớp 4), đặt **LDO 1V8** để:
- Đón dòng 3V3 và chuyển thành 1V8
- Cấp nguồn cho chân **DOVDD** của FPC Connector

![LDO 1V8 - 1](image/{5B452C1D-0F97-47A6-9F22-34FA72B4450F}.png)
![LDO 1V8 - 2](image/{2E2E306D-0CEB-44F5-B4F5-8926C6D401B2}.png)
![LDO 1V8 - 3](image/{2F1FA711-20F0-49C1-B1DB-F63F7A0F3612}.png)

### Điều khiển Power Good (PG)

Sử dụng chân **PG** của Buck để điều khiển chân **EN** của ESP32-S3:
- Mục đích: Khi dòng 3V3 ổn định thì module mới khởi động

![PG Control - 1](image/{3DBD7A48-3E60-4D12-A8F0-869103DE0D86}.png)
![PG Control - 2](image/{27E8E994-B15D-4DE7-BF53-3639AC711BB4}.png)

> *Mạch màu xanh: Chân PG kết nối vào chân EN của ESP32 để khởi động*

### Cấp nguồn cho ESP32-S3

- Cấp nguồn **3V3** từ Buck
- Sử dụng **2 tụ decoupling** sát chân 3V3 của module

![ESP32 Power](image/{077CBB7B-08DE-4D97-93D4-94A31CA7E844}.png)

---

## 3. 🔋 Khối LDO

### Bố trí

| LDO | Vị trí | Chức năng |
|-----|--------|-----------|
| **3V3 → 1V5** | Lớp 1 (Top) | Cấp nguồn DVDD cho Camera |
| **3V3 → 2V8** | Lớp 4 (Bottom) | Cấp nguồn AVDD cho Camera |

> Cả 2 khối LDO được đặt **gần FPC Connector** để giảm độ dài đường dây nguồn.

### Schematic & Layout

![LDO Overview](image/{94671F7D-09F0-443A-951C-69ADBA1CC892}.png)
![LDO Layout 1](image/{37705268-1C53-4631-BB6E-4785BCC43083}.png)
![LDO Layout 2](image/{1C606FE4-AFDC-4A93-90F1-5BA492ADD3B3}.png)
![LDO Layout 3](image/{D5477BCE-59E3-42E3-BA53-94F6E0323B90}.png)
![LDO Layout 4](image/{DF248AED-5EA1-44FC-AE74-0A8C2C96FB05}.png)

---

## 4. 📷 Khối FPC Connector (Camera)

### Schematic

![FPC Schematic 1](image/{F922C975-C0F9-4BEF-95DC-37F3276B077C}.png)
![FPC Schematic 2](image/{87E3CD52-B977-4E11-9777-9048316D481A}.png)
![FPC Schematic 3](image/{51326DFF-1582-480A-A61C-F34BBB468396}.png)

### Mô tả

Khối FPC Connector kết nối **camera OV2640** với ESP32-S3:
- Đặt **gần ESP32-S3** để giảm nhiễu và giảm độ dài dây kết nối

![FPC Layout](image/{08924485-C63C-499F-80E8-F835DC0FE6E9}.png)

### Thiết kế chi tiết

| Thành phần | Thiết kế |
|------------|----------|
| **Ferrite Bead** | Giảm nhiễu cho các chân nguồn của FPC |
| **AVDD (2V8)** | Ferrite Bead BEADC1608X95N 600R@100MHz + 2 tụ decoupling (0.1µF, 1µF) |
| **DOVDD (1V8)** | Tương tự AVDD |
| **DVDD (1V5)** | Tương tự AVDD |
| **MCLK** | GPIO 10 của ESP32-S3 cấp clock |
| **SDA/SCL** | Mạch pull-up kết hợp GPIO 47, GPIO 48 |
| **RESET** | Mạch RC để reset sạch, ổn định |
| **PWDN** | Mạch pull-down |

![Reset Circuit](image/{29A361AC-B092-4241-9216-938F7F87BFCC}.png)
![PWDN Circuit](image/{E7CFFB56-DBAF-4104-BD0C-1DD5BB34848C}.png)

---

## 5. 🔊 Khối Audio Amplifier (AMP)

### Bố trí

- Đặt **gần mép board**, cách xa LDO và FPC để tránh nhiễu
- 3 dây **DIN, BCLK, LRCLK** đi song song, thẳng

![AMP Layout](image/{F9515507-90E2-4E05-A330-9B8FE58AC4CF}.png)

### Thiết kế chi tiết

| Chân | Thiết kế |
|------|----------|
| **SD_MODE** | Mạch pull-down + GPIO38 điều khiển |
| **VDD** | Ferrite Bead giảm nhiễu + 2 tụ decoupling |
| **OUTP/OUTN** | 2 tụ trên đường dây tín hiệu xuất ra → Header Pin → Loa |

![AMP VDD](image/{486EDF7F-9E92-483F-9417-612CD2D846EA}.png)
![AMP Schematic](image/{321EFAFC-7E9C-43E7-824A-7D59357CD721}.png)
![AMP Output](image/{210607F8-D007-475A-942F-F896E61B1E17}.png)

> ⚠️ *Em đã sửa lại mạch: Xóa các Ferrite Bead sau chân OUTP/OUTN, chỉ dùng tụ*

![AMP Output Updated](image/{5B20EF41-2770-469D-9EE6-6F3D7F099B66}.png)

> **❓ Câu hỏi:**
> Với các chân OUTP và OUTN nếu chỉ dùng tụ không thì âm thanh đầu ra có bị dè hay nhiễu không? Nếu có thì em nên khắc phục như thế nào?

---

## 6. 🎤 Khối Mic PDM

### Bố trí

- Đặt **tránh xa LDO, Buck, FPC Connector và AMP** để tránh nhiễu
- Đặt **gần ESP32-S3** để dây DATA và CLK đi ngắn nhất, thẳng nhất

### Thiết kế chi tiết

| Chân | Thiết kế |
|------|----------|
| **VDD** | Ferrite Bead giảm nhiễu + 2 tụ decoupling |
| **SELECT** | Nối thẳng xuống GND |
| **Nguồn** | Cấp 1V8 cho Mic |

![Mic Schematic](image/{1B4B83B9-5CE6-4967-ADE1-EC5C5B12CD9C}.png)
![Mic Layout](image/{4A89F00D-9DDF-4FDF-891D-A74DADA999D6}.png)
![Mic SELECT](image/{B3940E35-2591-4962-A763-A01DC32EB8AD}.png)
![Mic Power](image/{13139196-C700-4AD9-8BD2-5B2168B69850}.png)

---

## 7. 📐 Tổng Quan Layout PCB

### Lớp Top (Lớp 1)

![PCB Top](image/{1A0D9F78-EDDF-48B3-B9B6-E256F9B80871}.png)

### Lớp Bottom (Lớp 4)

![PCB Bottom](image/{14FF0D45-3F29-49D0-9D67-85524CE2E7F6}.png)

### Các thành phần khác

| Thành phần | Layout |
|------------|--------|
| **Nút Reset** | ![Reset Button](image/{E8D0ABD4-06C2-41FD-B80B-B9BA86F1F8B7}.png) |
| **Nút Boot** | ![Boot Button](image/{F1BFADB4-7F0D-4BDD-8403-476B70D7E7AE}.png) |

### Routing dây Data FPC

![FPC Data Lines](image/{2956DE65-22AE-460F-A220-88328EFBCF71}.png)

> **Lưu ý:** Các dây data của FPC đi ở **Lớp 4**, còn 2 dây D- và D+ đi ở **Lớp 1**

### Hình ảnh tổng quan

![Full PCB](image/{5B6DCA3D-6460-4694-9895-1A2B506C7431}.png)

---