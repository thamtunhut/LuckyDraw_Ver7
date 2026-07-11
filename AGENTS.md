# Dự án: LuckyDraw Ver7

## Tổng quan

LuckyDraw Ver7 là ứng dụng quay số may mắn dành cho sự kiện, chạy trực tiếp trên trình duyệt và không cần build hoặc backend. Toàn bộ HTML, CSS và JavaScript nghiệp vụ nằm trong `index.html` (~2.850 dòng).

Luồng sử dụng chính:

1. Upload danh sách người tham dự bằng CSV.
2. Nhập cơ cấu giải thưởng và cấu hình cách quay.
3. Quay số đơn lẻ hoặc hàng loạt.
4. Xác nhận người thắng, ghép cặp thi đấu và xem lịch sử.
5. Xuất kết quả ra CSV.

Repository đích: `https://github.com/thamtunhut/LuckyDraw_Ver7.git`.

---

## Công nghệ

| Thành phần | Chi tiết |
|---|---|
| Ngôn ngữ | HTML5, CSS3, Vanilla JavaScript ES6+ |
| CSS | Tailwind CSS v3 qua CDN + CSS custom inline |
| Font | Google Fonts: Orbitron và Inter |
| Hiệu ứng | Canvas Confetti qua CDN |
| Build tool | Không có |
| Lưu trữ | `localStorage` |
| Hosting | Phù hợp GitHub Pages hoặc static hosting |

Không có `package.json`, `src/`, `node_modules`, framework frontend hoặc bước biên dịch.

---

## Cấu trúc thư mục hiện tại

```text
LuckyDraw_Ver7/
├── index.html                 # Toàn bộ ứng dụng
├── app_icon.jpg               # Avatar mặc định trong danh sách kết quả/cặp đấu
├── images (1).jpg             # Tài nguyên ảnh trong workspace
├── danh_sach_demo_100.csv     # Danh sách demo 100 người
└── AGENTS.md                  # Tài liệu dự án và chỉ dẫn cho Codex
```

`app_icon.jpg` phải được deploy cùng cấp với `index.html`, vì giao diện tham chiếu bằng đường dẫn tương đối `src="app_icon.jpg"`.

---

## Kiến trúc `index.html`

```text
index.html
├── <head>
│   ├── Tailwind CSS CDN
│   ├── Google Fonts
│   ├── Canvas Confetti CDN
│   ├── Tailwind theme config
│   └── <style>
│       ├── Aurora/background layers
│       ├── Slot reel animation
│       ├── Glass/control panels
│       ├── Battle Stage cho chế độ cặp đôi
│       ├── Winner/match cards
│       ├── Modal/history transitions
│       └── Responsive + reduced motion
├── <body>
│   ├── #setup-view
│   ├── #main-view
│   │   ├── #lottery-display
│   │   ├── #prize-select / #start-btn
│   │   ├── #winner-card
│   │   ├── #battle-stage
│   │   └── #tab-winners / #tab-pairs
│   ├── #history-panel
│   ├── #confirmation-modal
│   ├── #restore-modal
│   └── #custom-alert-modal
└── <script>
    ├── Slot reel helpers
    ├── State và DOM references
    ├── Visual-only helpers
    ├── Session persistence
    ├── CSV parser
    ├── Lottery flows
    ├── Pairing/Battle Stage rendering
    ├── Results/history rendering
    └── Event listeners
```

---

## Luồng nghiệp vụ

### Setup

```text
CSV (ma_so, ten_kh)
  → parseCSV()
  → customers[]
  → cấu hình prizes / numDigits / mode / spinDelay
  → initializeApp()
  → Main View
```

### Single mode

```text
Chọn giải
  → chọn ngẫu nhiên trong customers chưa thắng
  → runSingleLottery()
  → stopDigitsSequentially()
  → modal xác nhận
     ├── Chấp nhận → finalizeSingleWinner() → đánh dấu isWinner → lưu session
     └── Quay lại → hủy tempWinner → reset slot
```

### Multi mode

```text
runMultiLottery()
  → lặp tối đa prize.remaining người
  → mỗi người được đánh dấu isWinner ngay trong vòng lặp
  → cập nhật danh sách + giảm remaining + saveSession
  → hỗ trợ TẠM DỪNG / TIẾP TỤC qua checkPause()
```

### Pairing mode

```text
Người thứ nhất
  → pendingPartner = winner
  → Battle Stage hiển thị bên trái + "Đang tìm đối thủ"

Người thứ hai
  → pairs.push({ a: pendingPartner, b: winner, prizeName })
  → pendingPartner = null
  → Battle Stage chạy hiệu ứng VS
  → thêm match card vào tab Cặp đôi
```

Battle Stage và match cards chỉ là lớp trình bày. Không được đưa logic chọn người thắng vào các hàm render giao diện.

---

## Các hàm quan trọng

| Hàm | Vai trò |
|---|---|
| `create3DCubes()` | Tạo các slot reel theo `numDigits` |
| `buildReel()` | Tạo chuỗi ký tự trước ký tự đích |
| `stopSlotSpin()` | Giảm tốc một slot và dừng đúng ký tự |
| `stopDigitsSequentially()` | Dừng lần lượt toàn bộ slot theo `spinDelay` |
| `saveSession()` | Lưu state vào `localStorage` |
| `restoreSession()` | Phục hồi phiên cũ |
| `parseCSV()` | Đọc CSV có header `ma_so,ten_kh` |
| `updatePrizeSelect()` | Đồng bộ dropdown và số lượng giải còn lại |
| `initializeApp()` | Khởi tạo Main View |
| `updateListUI()` | Thêm winner, xử lý `pendingPartner` và tạo pair |
| `createWinnerListItem()` | Dựng winner card |
| `createPairListItem()` | Dựng match card |
| `prepareBattleForSpin()` | Chuẩn bị trạng thái Battle Stage |
| `renderBattleWaiting()` | Hiển thị người chơi 1 đang chờ đối thủ |
| `renderBattleMatch()` | Hiển thị đủ hai đấu thủ và chạy hiệu ứng VS |
| `triggerConfetti()` | Bắn confetti hai bên khi hoàn thành |
| `runSingleLottery()` | Luồng quay đơn |
| `runMultiLottery()` | Luồng quay hàng loạt |
| `renderHistoryTable()` | Dựng bảng lịch sử |

---

## State và cấu trúc dữ liệu

```javascript
customers = [
  { ma_so: string, ten_kh: string, isWinner: boolean }
]

prizes = [
  { name: string, quantity: number, remaining: number }
]

winners = [
  { ma_so, ten_kh, isWinner, prizeName }
]

pairs = [
  { a: customerObj, b: customerObj, prizeName: string }
]

pendingPartner = customerObj | null
```

Session được lưu tại key `luckyDrawSession`:

```javascript
{
  customers,
  prizes,
  winners,
  pairs,
  config: {
    numDigits,
    isMultiWinnerMode,
    pairingEnabled,
    title,
    spinDelay
  }
}
```

Lưu ý: `pendingPartner` hiện không nằm trong session. Không tự ý thay đổi schema session nếu chưa đánh giá tương thích dữ liệu cũ.

---

## Quy tắc nghiệp vụ bắt buộc

1. **Không quay trùng người.** Luôn chọn từ `customers.filter(c => !c.isWinner)`.
2. **Single mode chỉ đánh dấu thắng sau khi xác nhận.** Kết quả tạm nằm trong `tempWinner` và `tempPrize`.
3. **Multi mode đánh dấu thắng ngay trong vòng lặp.** Không thêm confirmation modal cho từng lượt.
4. **Mỗi cặp gồm đúng hai người.** Người đầu nằm trong `pendingPartner`; người thứ hai mới tạo `pairs.push(...)`.
5. **Không reset `isWinner` sau khi xác nhận.**
6. **Mỗi lần thay đổi state quan trọng phải gọi `saveSession()`.**
7. **`spinDelay` là khoảng dừng giữa các slot.** Slot đầu dừng sau `spinDelay + 400ms`; các slot tiếp theo cách nhau `spinDelay`.
8. **Các hàm hiệu ứng không được thay đổi winner, prize hoặc session.**

---

## Giao diện và hiệu ứng Ver7

- Slot machine dạng reel, hỗ trợ chữ cái và chữ số.
- Motion blur khi quay, giảm tốc bằng cubic-bezier và impact khi dừng.
- Machine frame, stage halo và aurora nền nhẹ.
- Winner reveal, confetti hai phía và modal transitions.
- Battle Stage cyan/đỏ với huy hiệu `VS` cho pairing mode.
- Winner cards và match cards trong bảng kết quả.
- Avatar mặc định dùng `app_icon.jpg`.
- Tab kết quả dạng segmented control.
- Hỗ trợ `prefers-reduced-motion`.

### Lưu ý chống nhấp nháy/GPU

- Full-screen canvas particles đang **không được khởi chạy**. Không gọi lại `initParticles()` nếu chưa kiểm tra trên Chrome/Edge với `backdrop-filter`.
- `.control-panel` cố ý dùng `backdrop-filter: none` để tránh lỗi sọc ngang/compositor khi có ảnh nền.
- Không thêm pseudo-element tuyệt đối cho `.btn-neon` nếu chưa đặt đúng containing block; lỗi cũ từng làm nền nút phủ toàn màn hình.
- Không thêm animation blur/filter liên tục trên phần tử full-screen.
- Ưu tiên animation cục bộ bằng `transform` và `opacity`.

---

## Theme

```javascript
primary:   '#00f2ff' // cyan
secondary: '#7000ff' // purple
accent:    '#ff0055' // pink/red
dark:      '#050510' // near black
```

Trong pairing mode:

- Người chơi A: cyan.
- Người chơi B: pink/red.
- `VS`: trắng, glow hai màu.

---

## CSV

Format bắt buộc:

```csv
ma_so,ten_kh
LD0001,Nguyễn Văn An
LD0002,Trần Thị Bình
```

Quy tắc:

- Header phải đúng `ma_so` và `ten_kh`.
- Phân tách bằng dấu phẩy.
- UTF-8.
- Parser hiện tại là parser đơn giản theo `split(',')`; tránh dấu phẩy trong tên.
- `danh_sach_demo_100.csv` có 100 mã duy nhất từ `LD0001` đến `LD0100`.

---

## Chạy local và deploy

Ứng dụng có thể mở trực tiếp bằng trình duyệt. Để kiểm tra file upload và hành vi gần với GitHub Pages hơn, dùng static server:

```bash
python -m http.server 8080
```

Sau đó mở `http://localhost:8080`.

Deploy GitHub Pages:

1. Push `index.html`, `app_icon.jpg` và các asset cần thiết lên branch `main`.
2. Trong GitHub repository, bật Pages từ branch `main`/root.
3. Không cần build command.

---

## Quy tắc phát triển

### Kiến trúc

- Không tách logic ứng dụng khỏi `index.html`.
- Không thêm React, Vue, Angular hoặc build tool.
- Không thêm `import`/`require` cho logic browser.
- Asset ảnh và CSV demo được phép tồn tại riêng cạnh `index.html`.

### Convention

- State nằm trong scope của `DOMContentLoaded`.
- DOM element được cache vào `const` khi dùng nhiều lần.
- Dùng async/await cho animation sequences.
- UI sử dụng tiếng Việt.
- Dùng `showAlert()` thay cho `window.alert()`.
- Khi dựng nội dung từ CSV bằng `innerHTML`, phải qua `escapeHTML()`.
- Mọi class hiệu ứng mới phải có fallback trong `prefers-reduced-motion` nếu chuyển động đáng kể.

### Khi thay đổi tính năng

1. Xác định state và luồng single/multi/pairing bị ảnh hưởng.
2. Sửa trực tiếp `index.html`.
3. Nếu thêm state cần persist, cập nhật đồng thời `saveSession()` và `restoreSession()`.
4. Kiểm tra cú pháp JavaScript.
5. Kiểm tra các ID DOM không thiếu hoặc trùng.
6. Kiểm tra các bất biến: không trùng winner, giảm đúng prize, pairing đúng hai người.
7. Kiểm tra desktop và breakpoint 768px.

---

## Trạng thái hiện tại

Đang hoạt động:

- Single mode với xác nhận/quay lại.
- Multi mode với tạm dừng/tiếp tục.
- Pairing mode với Battle Stage và match cards.
- Upload/validate CSV.
- Upload ảnh nền.
- Cấu hình tiêu đề, số ký tự, opacity, blur và tốc độ.
- Session restore.
- Lịch sử bằng F1.
- Export CSV cho winner và pair.
- Keyboard Space để quay/dừng khi hợp lệ.
- Responsive mobile.

Không có build step và không có test framework tự động. Kiểm tra tối thiểu hiện tại là parse JavaScript, kiểm tra ID DOM và chạy thử trên trình duyệt.

---

## TODO tiềm năng

- Cho phép chỉnh sửa giải sau khi bắt đầu.
- Preview danh sách CSV trước khi chạy.
- Thêm âm thanh có công tắc bật/tắt.
- Hỗ trợ `.xlsx`.
- Persist `pendingPartner` có versioning để tương thích session cũ.
- Thêm chế độ trình chiếu toàn màn hình.

---

## Những điều không được làm

- Không thay đổi thuật toán random khi yêu cầu chỉ liên quan giao diện.
- Không đánh dấu winner trước confirmation trong single mode.
- Không reset `customers[].isWinner` ngoài luồng reset toàn bộ.
- Không thêm animation full-screen liên tục gây GPU flicker.
- Không bỏ `app_icon.jpg` khỏi deployment khi danh sách kết quả còn tham chiếu file này.
- Không commit file cấu hình cá nhân như `.claude/settings.local.json`.
