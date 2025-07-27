# VNPT GW020H Reverse Engineering & Rooting Project

## 📌 Mục tiêu dự án

Dự án nhằm mục đích nghiên cứu và khai thác modem **GW020H** và **các dòng modem liên quan (GW040H, NS, ...)** của VNPT, cụ thể:

- Truy cập **root shell** thông qua mạng nội bộ, sửa đổi file config **và** UART.
- Phân tích firmware gốc và các cơ chế bảo vệ.
- Hỗ trợ **debricking** thiết bị trong quá trình vọc vạch.

> ⚠️ **Miễn trừ trách nhiệm**: Dự án này chỉ phục vụ mục đích **nghiên cứu, học tập** và không khuyến khích sử dụng vào các hoạt động vi phạm pháp luật, quyền riêng tư hay điều khoản sử dụng của nhà mạng. Bạn hoàn toàn chịu trách nhiệm nếu sử dụng sai mục đích.

---

## 📂 Nội dung repo

- `flashdump/mtd0.bin` đến `mtd10.bin`: Dump đầy đủ từ modem chạy **firmware gốc ver 1** – có thể phân tích bằng binwalk và dùng để debrinking trong trươngf hợp cần thiết.
- Một bản **initramfs OpenWrt** tương thích với SoC của modem: dùng để **debrick** hoặc mở shell tạm thời.
- Một **tài liệu nội bộ kỹ thuật** của VNPT (tham khảo).
- Các **script** và **notes** cùng với **phần mềm kèm theo** phục vụ việc root và truy cập hệ thống.
- Một bản dump firmware đã được strip, trong **squashfs-modified** , gồm 2 file **boa-dump.bin** là file nguyên gốc trong quá trình update firmware qua web UI và file **squashfs.image** là file đã được trích xuất phần squashfs ( có thể dùng tool únquashfs để unpack )

---

## 🔧 Hướng dẫn sơ bộ

1. **Kết nối UART**:
   *với một số dòng modem như trong tài liệu nội bộ của vnpt, có thể làm theo hướng dẫn để mở telnet mà không cần làm theo các bước dưới.
   - Yêu cầu mở nắp thiết bị, dùng bộ chuyển đổi UART-to-USB (nên dùng CH340), dây jumper.
   - Xác định vị trí chân cắm gần khu vực đèn LED, gồm 3 khe: `RX`, `TX`, `GND`.
   - ⚠️ Cắm đúng để tránh hư thiết bị.

2. **Bật nguồn** và chờ thiết bị khởi động đến khi xuất hiện dòng:

Please press Enter to activate this console.

3. **Đăng nhập**:
- Nhấn Enter để thấy prompt `tc login:`
- Có 3 tài khoản có thể đăng nhập:
  - `admin / VnT3ch@dm1n`
  - `operator / VnT3ch0per@tor`
  - `customer / customer` (quyền hạn thấp)
- Đăng nhập thành công sẽ vào shell. Gõ `uname -a` để xác nhận.

4. **(Tùy chọn)** Mở Telnet tạm thời (reboot sẽ mất):
```sh
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
```
Địa chỉ Telnet/SSH mặc định: 192.168.1.1

5.**(Tùy chọn)** Boot vào OpenWrt initramfs: xem mục "BOOT WRT" trong thư mục doc

---

##🛠️ Ghi chú kỹ thuật

-SoC: MediaTek EN751221
-Flash NAND: 128MB SPI NAND (F50L1G41LB hoặc một số dòng chip khác)
-Dump được thực hiện bằng cat /dev/mtdX hoặc nanddump từ BusyBox trong OpenWrt initramfs.
-Tài liệu nội bộ đã có sẵn trên Scribd (xem trong doc).
-Credentials và thông tin đăng nhập có trong doc/credentials.txt.
-Hiện chưa timf được phương pháp giải mã firmware mà không thông qua dump file giải mã sẵn khi upload

---

##🛠️ Patch romfile.cfg

Giới thiệu:
-romfile.cfg là file cấu hình backup của modem, tải được từ 192.168.1.1 → Maintenance → Backup/Restore.


-File chứa:
+ LOID & mật khẩu LOID
+ SSID, mật khẩu Wi-Fi
+ Cấu hình mạng, firewall, cron task,...
  
- ⚠️ Không nên chia sẻ public vì chứa nhiều thông tin nhạy cảm.
  
-Giải mã & chỉnh sửa
+ File được mã hóa bởi chương trình cfg_manager trong firmware.
+ Đã reverse thành công IV và key để giải mã và mã hóa lại.
+ Tất cả các modem GW dùng chung key → có thể chuyển romfile.cfg giữa các thiết bị.

 - Sử dụng công cụ
+ Dùng script tools/romfileedit.py để giải mã và chỉnh sửa.
+ Yêu cầu: Python + một số thư viện hỗ trợ (xem trong tool).
+ Sau khi chỉnh sửa, cần mã hóa lại và upload qua web UI để modem chấp nhận.


--- 

##Debricking (OPEN WRT)
- Trong một số trường hợp, thiết bị có thể bị brick do một số chỉnh sửa gây tác động và làm hệ thống không thể khởi động, việc này gây bootloop hay web UI không load được ( có thể thử restart boa nếu vẫn còn truy cập được shell ), trong trường hợp không truy cập được shell, thử tắt nguồn bằng nút vật lý và mở lại để reboot thiết bị, nếu vãn không giải quyết được, hãy dùng phương pháp load initramfs tạm thời:
- Tham khảo [OpenWrt Wiki] TP-Link Archer VR1200v (v2).pdf hoặc truy cập theo link https://openwrt.org/inbox/toh/tp-link/archer_vr1200v?s[]=tp%2A&s[]=link%2A mục debricking
- Sau khi vào được shell root của openwrt, có thể flash lại firmware ( các bản mtdX.bin ) , reboot thiết bị để kiểm tra ( vui lòng giữ file backup romfile.cfg của modem bạn để tải lên restore sau khi debricking )
---
##Phương thức giải mã firmware

hãy nhập lệnh sau vào shell của router

sed -i '1,$d' /tmp/auto_dump_boatemp.sh
cat >> /tmp/auto_dump_boatemp.sh <<'EOF'
#!/bin/sh
out="/tmp/yaffs/boa-dump.bin"
mkdir -p /tmp/yaffs

echo "[*] Waiting for /tmp/boa-temp to complete upload..."
last_size=0
stable_count=0

while true; do
    if [ -f /tmp/boa-temp ]; then
        set -- $(ls -l /tmp/boa-temp 2>/dev/null)
        size=$5

        if [ "$size" -gt 100000 ]; then
            if [ "$size" -eq "$last_size" ]; then
                stable_count=`expr $stable_count + 1`
            else
                stable_count=0
            fi
            last_size=$size

            # Nếu không đổi 2 lần liên tiếp (2 giây) => upload xong
            if [ "$stable_count" -ge 2 ]; then
                cp /tmp/boa-temp "$out"
                echo "[+] Dumped boa-temp ($size bytes) to $out"
                break
            fi
        fi
    fi
    sleep 1
done
EOF

chmod +x /tmp/auto_dump_boatemp.sh



- command trên sẽ tạo file auto_dump_boatemp.sh là file lệnh để dump được bản firmware đã giải mã, sau khi xong, hãy dùng lệnh
- sh /tmp/auto_dump_boatemp.sh để chạy
- lưu ý, ko tắt shell trong quá trình thực hiện dump, như thế sẽ làm code dừng và ko dump được,
- bước tiếp theo, hãy đăng nhập tài khoản web, upload file firmware mà bạn muốn dump và bấm upgrade ( việc này cũng sẽ update hệ thống, hãy cẩn thận)
sau đó, có thể truy cập lại shell, vào /tmp/yaffs để có file dump ( có thể dùng tftp để tải về )
- sau đó có thể dùng tool binwalk hoặc únsquashfs để xem nội dung
- có thể chỉnh sửa và repack lại, sau đó can thiệp vào quá trình update thông qua sửa đổi file boa-temp để ép router update firmware đã sửa ( lưu ý, ruit ro brick rất cao nếu ko timming chuẩn nhưng đây là phương pháp duy nhất để vượt qua mode ro của squáshfs, trong tương lai, admin sẽ cố tung ra 1 bản đã sửa sẵn để tắt ro cho mn xài)
