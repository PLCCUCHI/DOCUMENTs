# EDI Sample 1 – COREOR (Giao container hàng nhập)

Ngày tạo: 13/08/2026 · Nguồn: `98. Thu thap data\EDI Collection others\02. COREOR` · 16 file / 3 hãng

## COREOR lấp đúng ô còn thiếu của ma trận COPARN

Toàn bộ 16 file đều là `BGM+12` (Gate Out) với `EQD ... +3+5` = **Import + Full**.

| Message | BGM | 8249 | 8169 | Nghiệp vụ | Thư mục mẫu |
|---|---|---|---|---|---|
| COPARN | 12 Gate Out | 2 Export | 4 Empty | Cấp rỗng đóng hàng xuất | `edi sample` |
| COPARN | 11 Gate In | 3 Import | 4 Empty | Trả rỗng sau rút hàng nhập | `edi sample` |
| COPARN | 11 Gate In | 2 Export | 5 Full | Hạ cont hàng xuất vào bãi | `edi sample` |
| **COREOR** | **12 Gate Out** | **3 Import** | **5 Full** | **Giao cont hàng nhập** | **`edi sample 1`** |

Kết luận: COPARN lo vòng đời container **rỗng**, COREOR lo lệnh giao container **hàng nhập** (bản chất là e-DO của hãng tàu gửi cảng). Hai message không thay thế nhau.

## Quy ước thư mục con

| Thư mục | BGM function 1225 | Ý nghĩa |
|---|---|---|
| `goc` | 9 | Điện gốc |
| `thaythe` | 5 | Thay thế |
| `huy` | 1 | Huỷ — **chưa có mẫu** |
| `bosung` | 2 | Bổ sung — **chưa có mẫu** |

## Cấu trúc chung của COREOR (cả 3 hãng giống nhau)

`BGM+12` → `RFF+BM` (số B/L) → `RFF+AAJ` (số lệnh giao hàng) → `TDT`+`RFF+VON` → `LOC+170` (cảng dỡ) / `LOC+176` (bến giao) / `LOC+99` (nơi trả rỗng) → `NAD+CA` (hãng) + `NAD+BJ` (bên nhận hàng) → `DTM+137` (ngày phát hành) / `DTM+200` (ngày được lấy) / `DTM+400` (hạn hiệu lực) → `EQD+CN`+số cont → `RFF+SQ` + `RFF+EP` → `FTX+AAI` → `CNT+16`

Khác COPARN ở 3 điểm cốt lõi: dùng `RFF+BM`/`RFF+AAJ` thay `RFF+BN`/`RFF+REO`; **luôn có số container**; có `LOC+99` chỉ định nơi trả rỗng ngay trong lệnh giao.

## Khác biệt giữa 3 hãng — ảnh hưởng parser

| Tiêu chí | HMM | Maersk | ONE |
|---|---|---|---|
| Syntax UNB | UNOA:1 | **UNOB:1** | UNOA:1 |
| Association | không | không | ITG14 |
| Response type BGM | không | không | **`+AB`** (chờ APERAK) |
| Mã size/type | ISO 22G1 | ISO 45G1/22G1 | **mã cũ 4500/4200** |
| EQD supplier 8077 | `2` | trống | trống |
| Số message/interchange | **1–2** | 1 | 1 |
| `LOC+99` nơi trả rỗng | tên depot | tên depot | **"KHONG XAC DINH"** |
| `NAD+BJ` | **MST + tên, ngăn bằng `#`** | chỉ tên | chỉ tên |
| Định dạng `DTM+400` | 12 số (đúng 203) | 12 số (đúng 203) | **14 số nhưng vẫn khai 203** |

## Điểm cần xử lý / xác nhận

- **Maersk dùng UNOB** thay UNOA — bộ ký tự khác, parser phải nhận cả hai.
- **ONE khai sai `DTM+400`**: giá trị `20260722235959` (14 số) nhưng format qualifier `203` = CCYYMMDDHHMM (12 số). Parser đọc cứng theo 203 sẽ lệch. Cần hỏi ONE hoặc xử lý bằng độ dài chuỗi.
- **HMM nhét mã số thuế vào `NAD+BJ`** với 2 kiểu khác nhau: `0105821250#TÊN CTY #` và `TAX ID0302654231#TÊN CTY`. Không có quy tắc ổn định — phải hỏi HMM.
- **HMM đánh số `UNH` là 1, 2** trong cùng interchange — không unique toàn cục, không dùng làm khoá chống trùng được.
- **ONE để `LOC+99` = "KHONG XAC DINH"** — nếu CP dựa vào trường này để chỉ nơi trả rỗng thì luồng ONE sẽ thiếu dữ liệu.
- **`RFF+EP:N`** xuất hiện ở cả 3 hãng, luôn giá trị `N`. Chưa rõ ngữ nghĩa (Equipment Prepaid? Empty Pickup?) — cần xác nhận.
- **Chưa có mẫu huỷ (function 1) và bổ sung (function 2)** cho COREOR ở cả 3 hãng.
- File `MAEU_VNHPHNW_COPARN_338570_...EDI` nằm trong thư mục `02. COREOR/MAE` nhưng thực chất là **COPARN** — đã bỏ qua, không copy sang đây.

## Danh sách file

### 01. HMM

| Loại | Func | Số msg | File |
|---|---|---|---|
| goc | 9 | 2 | `TVU_CRO_CRO20251104162423286KPHH_251104162423160` |
| thaythe | 5 | 2 | `TVU_CRO_CRO20251105115502363CMMF_251105115502575` |
| thaythe | 5 | 1 | `TVU_CRO_CRO20251105115550233OMOS_251105115550576` |
| thaythe | 5 | 1 | `TVU_CRO_CRO202511071503111778A1K_251107150311191` |

### 02. MAEU - Maersk

| Loại | Func | Số msg | File |
|---|---|---|---|
| goc | 9 | 1 | `MAEU.VNHPHNW.COREOR.469471.198692901343686946.edi` |
| goc | 9 | 1 | `MAEU.VNHPHNW.COREOR.469473.198692901343763947.edi` |
| thaythe | 5 | 1 | `MAEU.VNHPHNW.COREOR.469476.298692901615525921.edi` |
| thaythe | 5 | 1 | `MAEU.VNHPHNW.COREOR.469477.198692901615792275.edi` |

### 03. ONE

| Loại | Func | Số msg | File |
|---|---|---|---|
| goc | 9 | 1 | `ONE_FULREL_20260709163738.000033245186.EDI` |
| goc | 9 | 1 | `ONE_FULREL_20260709163738.000033245192.EDI` |
| goc | 9 | 1 | `ONE_FULREL_20260709163738.000033245193.EDI` |
| goc | 9 | 1 | `ONE_FULREL_20260709163738.000033245197.EDI` |
| goc | 9 | 1 | `ONE_FULREL_20260709163739.000033245199.EDI` |
| goc | 9 | 1 | `ONE_FULREL_20260709164456.000033254013.EDI` |
| goc | 9 | 1 | `ONE_FULREL_20260709165113.000033260809.EDI` |
| thaythe | 5 | 1 | `ONE_FULREL_20260709164252.000033251627.EDI` |
