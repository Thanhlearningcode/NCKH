
# 🔧 Cài đặt và sử dụng Visual Studio Code để chạy MATLAB script (.m) với GNU Octave trên Linux

## 📌 Giới thiệu

Dự án này sử dụng **GNU Octave** như một thay thế miễn phí cho **MATLAB** để chạy các file `.m`. Visual Studio Code được sử dụng làm trình soạn thảo chính.

---

## ⚙️ Cài đặt

### 1. Cài đặt GNU Octave

```bash
sudo apt update
sudo apt install octave
```

### 2. Cài đặt Visual Studio Code

**Cách nhanh nhất:**

```bash
sudo apt update
sudo apt install wget gpg -y

# Thêm key và repo
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /usr/share/keyrings/
sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/packages.microsoft.gpg] \
https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'

# Cài đặt VS Code
sudo apt update
sudo apt install code
```

---

## 🚀 Sử dụng

### 1. Mở project với VS Code

```bash
code ~/duong_dan_toi_folder_chua_file_m
```

### 2. Cài Extension cần thiết trong VS Code

- ✅ **Octave** (tên: *Octave* hoặc *language-octave* – hỗ trợ syntax highlight `.m`)
- ✅ **Code Runner** (chạy script bằng cách nhấn nút ▶️ hoặc Ctrl+Alt+N)
- ✅ **Terminal tích hợp** (có thể chạy lệnh `octave --no-gui` trong terminal bên trong VS Code)

---

## ▶️ Chạy file `.m`

### Cách 1: Trong VS Code

1. Mở file `ex1.m`
2. Nhấn `Ctrl+Alt+N` (nếu đã cài **Code Runner**)
3. Kết quả sẽ hiện ở terminal bên dưới

### Cách 2: Từ terminal

```bash
octave --no-gui ex1.m
```

---

## 📁 Ví dụ: File `ex1.m`

```matlab
% ex1.m
x = 0:0.01:2*pi;
y = sin(x);
plot(x, y)
title('Biểu đồ Sin(x)')
xlabel('x')
ylabel('sin(x)')
```

---

## 📝 Lưu ý

- `octave` không tương thích hoàn toàn với mọi hàm MATLAB nâng cao (Simulink, GUI, toolbox nâng cao).
- Để vẽ được đồ thị, hệ thống cần cài thư viện hỗ trợ hiển thị đồ họa (`gnuplot`, `qt`, hoặc `fltk`).
- Nếu bạn chạy từ WSL hoặc không thấy đồ thị hiện lên, hãy đảm bảo bạn có môi trường hiển thị X11.

---

## 📬 Liên hệ

Nếu bạn cần thêm hướng dẫn chi tiết, có thể liên hệ qua: `your_email@example.com`.
