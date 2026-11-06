# Kịch bản video demo (bản tiếng Việt, khoảng 9 phút)

Đề tài: Containers in Web Development and Deployment (Docker). Môn 503073 - Lập trình web và ứng dụng.

Đây là bản ngắn, chỉ giới thiệu sơ qua phần đã thực hiện rồi chuyển sang demo sản phẩm. Bản thuyết trình đầy đủ (khoảng 18 phút, có lý thuyết) nằm ở `video-script-vi.md`. Không cần đọc thuộc từng chữ: hãy nói lại bằng lời của mình, giữ đúng ý và đúng số liệu.

Tổng thời lượng dự kiến: khoảng 9 phút (trong khoảng 5-10 phút), chia đều hai người. Cả hai cùng xuất hiện trong video. Các mốc dưới đây là ước lượng, thời gian thật phụ thuộc vào tốc độ gõ lệnh và thời gian chờ container khởi động.

| Phút | Nội dung | Người nói |
|---|---|---|
| 0:00 | Giới thiệu nhanh: đã làm gì | Hiếu |
| 1:00 | Kiến trúc DockMart | Huy |
| 2:00 | Demo 1: chạy hệ thống, mua hàng | Huy |
| 4:00 | Demo 2: scale, cân bằng tải, tắt container | Hiếu |
| 5:45 | Demo 3: dữ liệu bền vững, bảo mật | Huy |
| 7:00 | Demo 4: Kubernetes | Hiếu |
| 8:30 | Tổng kết | Hai người |

## Chuẩn bị trước khi quay

1. Mở Docker Desktop. Bật Kubernetes nếu quay Demo 4 (Settings → Kubernetes, chọn kind, 3 node), và triển khai sẵn theo README mục 6.
2. Trong thư mục `source`, chạy trước để lúc quay không phải chờ tải: `docker compose build`.
3. Xóa dữ liệu cũ để đơn hàng bắt đầu từ số 1: `docker compose down -v`.
4. Mở sẵn terminal (cỡ chữ lớn), trình duyệt và slide kiến trúc. Tắt thông báo Zalo, Discord.
5. Tài khoản admin: `admin@dockmart.local` / `DockMart-Admin-2026`.
6. Quay thử 1 phút để kiểm tra tiếng và hình.

---

## 0:00 Giới thiệu nhanh (Hiếu)

**Màn hình:** slide bìa, bật camera cả hai người.

> Chúng em chào thầy và các bạn, chúng em là Trần Trung Hiếu và Nguyễn Đức Nhật Huy. Đây là video demo cho bài tiểu luận giữa kỳ môn Lập trình web và ứng dụng, do thầy Nguyễn Thái Duy hướng dẫn. Đề tài là container trong phát triển và triển khai ứng dụng web, với công cụ Docker.
>
> Để kiểm chứng lý thuyết, nhóm xây dựng DockMart, một cửa hàng trực tuyến chạy hoàn toàn bằng container. Bản thân cửa hàng khá bình thường. Điểm chính là cách nó chạy: nhóm scale nó khi đang phục vụ người dùng, cố tình làm container sập để xem nó hồi phục ra sao, và chạy cùng một bộ image trên cả Docker Compose lẫn Kubernetes ba node.
>
> Phần nội dung chi tiết về cách container hoạt động nằm trong báo cáo. Trong video này nhóm đi thẳng vào demo.

## 1:00 Kiến trúc DockMart (Huy)

**Màn hình:** slide kiến trúc.

> Đây là kiến trúc của DockMart, gồm bảy loại container. Trình duyệt chỉ đi vào qua gateway là Nginx. Gateway chuyển trang web tới container web, và chuyển các lệnh gọi API tới một trong ba replica. Đơn hàng được đưa vào hàng đợi trong Redis, rồi các container worker xử lý ở phía sau. Dữ liệu nằm trong PostgreSQL, trên một named volume.
>
> Hai quyết định thiết kế quan trọng nhất: API không giữ trạng thái nên replica nào cũng phục vụ được mọi request, và cơ sở dữ liệu nằm trong mạng nội bộ mà gateway không có đường đi tới. Bây giờ em sẽ chạy thử.

## 2:00 Demo 1: chạy hệ thống, mua hàng (Huy)

**Màn hình:** terminal và trình duyệt.

Thao tác:

1. Chạy `docker compose up -d --build --wait`.
2. Chạy `docker compose ps`.
3. Mở `http://localhost:8080`, tạo tài khoản, thêm 2 sản phẩm, đặt hàng.
4. Ở trang **My orders**, để mọi người thấy trạng thái tự chạy.

> Chỉ một lệnh là cả hệ thống khởi động, và máy này chỉ cần cài Docker. Không cần cài Node.js, PostgreSQL hay Redis, vì tất cả đã nằm trong image. Đây chính là lợi ích đầu tiên của container: máy nào có Docker thì chạy được y hệt, hết cảnh "máy em chạy được mà".
>
> Compose khởi động các service theo đúng thứ tự dựa trên health check: PostgreSQL và Redis trước, rồi tới container migrate, API và worker, và gateway là cuối cùng. Health check nghĩa là Compose không chỉ chờ container bật lên, mà chờ đến khi nó thật sự trả lời được. Cờ `--wait` làm lệnh chỉ kết thúc khi mọi service đã khỏe.
>
> Em chạy `docker compose ps` để xem kết quả. Mọi người thấy ba replica API, hai worker, và trạng thái healthy. Riêng container migrate có trạng thái Exited 0. Đó không phải lỗi: nó là một job chạy một lần để tạo bảng và nạp sản phẩm mẫu, xong việc thì thoát, và API chỉ được chạy sau khi nó thành công.
>
> Đây là cửa hàng. Em tạo tài khoản, thêm vài sản phẩm và đặt hàng. Mọi người để ý trạng thái đơn hàng: em không hề tải lại trang. API không tự xử lý đơn mà đưa vào hàng đợi trong Redis. Một container worker nhận đơn, trừ kho, rồi từng bước được gửi tới trình duyệt theo thời gian thực. Dòng cuối cho biết worker nào đã xử lý, nên mọi người thấy rõ đơn hàng đi qua nhiều container khác nhau.

## 4:00 Demo 2: scale, cân bằng tải, tắt container (Hiếu)

**Màn hình:** đăng nhập admin, trang **Ops Center**, chia đôi màn hình với terminal.

Thao tác:

1. Bấm **Send 300 requests**.
2. Chạy `docker compose up -d --scale api=5 --no-recreate`, chờ 2 thẻ mới xuất hiện, bấm lại **Send 300 requests**.
3. Bấm **Kill** trên một thẻ API.
4. Chỉ vào thẻ đó: `Uptime` về 0 và `Restarts` tăng 1.

> Đây là Ops Center, trang dành cho quản trị viên. Mỗi thẻ là một replica đang chạy, tức là một bản sao của cùng một image. Em gửi 300 request qua gateway, và các thanh cho thấy chúng được chia đều cho ba replica. Việc chia đều này do Nginx ở gateway đảm nhận.
>
> Giờ em tăng lên năm replica chỉ bằng một lệnh, trong lúc website vẫn chạy. Hai thẻ mới xuất hiện. Em gửi lại, và lần này năm replica cùng chia tải. Em không đụng gì tới gateway và không phải sửa cấu hình. Điều này làm được vì API không giữ trạng thái trong bộ nhớ: phiên đăng nhập và dữ liệu dùng chung đều nằm ngoài, nên replica nào cũng phục vụ được bất kỳ request nào. Trong thí nghiệm của nhóm, thông lượng tăng từ 107 lên 510 request mỗi giây khi đi từ một lên sáu replica, tức là gấp khoảng 4,8 lần.
>
> Bây giờ em sẽ phá hệ thống. Em bấm Kill, tiến trình đó sập thật, và Docker đưa nó quay lại chỉ sau vài giây, nhờ chính sách tự khởi động lại của container. Mọi người nhìn vào thẻ này: Uptime về không, Restarts tăng thêm một, và nhật ký sự kiện ghi lại lần sập. Trong thí nghiệm của nhóm, 0 trên 4.001 request bị lỗi khi xảy ra sự cố, vì gateway tự thử lại ở replica khác nên người dùng không nhận ra gì. Đây là hai lớp bảo vệ: gateway che đi lúc sập, còn Docker đưa replica trở lại.

## 5:45 Demo 3: dữ liệu bền vững, bảo mật (Huy)

**Màn hình:** terminal.

Thao tác:

1. `docker compose rm -sf postgres`, rồi `docker compose up -d postgres`.
2. Tải lại trang **My orders**: đơn hàng vẫn còn.
3. `docker compose exec gateway nc -z -w2 postgres 5432`: báo `bad address`.
4. `docker compose exec api sh -c 'id; touch /app/x; grep CapEff /proc/1/status'`.

> Container vốn là thứ tạm thời: xóa đi là mất mọi thứ bên trong. Vì vậy em làm một phép thử. Em vừa xóa hẳn container cơ sở dữ liệu, rồi tạo lại container mới. Tải lại trang, đơn hàng vẫn còn, vì dữ liệu nằm trong volume chứ không nằm trong container. Container mới chỉ việc gắn lại volume cũ và đọc tiếp. Nguyên tắc ở đây là code nằm trong image, còn dữ liệu nằm trong volume.
>
> Giờ sang bảo mật. Hệ thống chia mạng thành nhiều vùng. Từ gateway em thử kết nối PostgreSQL thì cái tên đó còn không phân giải được, vì gateway và cơ sở dữ liệu không chung mạng. Mạng chứa cơ sở dữ liệu còn được đặt là internal nên không có đường ra Internet.
>
> Tiếp theo là bên trong container API. User không phải root, hệ thống file chỉ đọc nên không ghi được vào thư mục ứng dụng, và tập capability bằng không, tức là tiến trình không có đặc quyền Linux nào. Mật khẩu cũng không nằm trong biến môi trường mà được đưa vào bằng Docker secret, dưới dạng file. Nên kể cả khi kẻ tấn công lọt vào một container, họ cũng làm được rất ít.

## 7:00 Demo 4: Kubernetes (Hiếu)

**Màn hình:** terminal, chạy trước `kubectl -n dockmart get pods -o wide`.

Thao tác:

1. `kubectl -n dockmart get pods -o wide`, rồi dùng chuột khoanh vào cột `NODE`.
2. `kubectl -n dockmart set image deploy/api api=dockmart/backend:1.1.0`, rồi `kubectl -n dockmart get pods -w`.
3. `kubectl -n dockmart rollout undo deploy/api`.
4. Nếu đủ thời gian: `kubectl -n dockmart get hpa`.

> Compose rất tiện trên một máy, nhưng khi cần chạy trên nhiều máy thì người ta dùng Kubernetes. Những image đó giờ chạy trên một cluster Kubernetes ba node mà không sửa một dòng code nào, chỉ thay đổi phần mô tả cách chạy. Mọi người nhìn vào cột NODE, cột này cho biết mỗi pod đang chạy trên máy nào. Các pod được chia cho hai worker node là desktop-worker và desktop-worker2, và các pod API có mặt ở cả hai. Node thứ ba là control-plane, chỉ dùng để điều khiển cluster nên không chạy pod ứng dụng. Ý tưởng của Kubernetes là mình khai báo trạng thái mong muốn, ví dụ ba replica, và hệ thống luôn tự kéo thực tế về đúng trạng thái đó.
>
> Giờ em cập nhật lên phiên bản 1.1. Kubernetes khởi động pod mới, chờ nó sẵn sàng rồi mới gỡ pod cũ, từng pod một. Mọi người nhìn cột trạng thái: pod mới lên trước, pod cũ mới biến mất sau, nên lúc nào cũng có pod phục vụ. Nhóm đã chạy việc này với 100 request mỗi giây: trong 7.500 request có 0 request lỗi, và quá trình cập nhật mất 14,2 giây.
>
> Nếu phiên bản mới có lỗi, em chỉ cần chạy `rollout undo` là Kubernetes quay lại bản cũ, cũng theo cách từng pod một nên không gián đoạn.
>
> Khi tải nặng, Kubernetes còn tự tăng từ ba lên tám replica trong khoảng 60 giây, rồi trở về ba sau khi hết tải. Đây là điều Compose không tự làm được, và là lý do chính để chọn Kubernetes khi hệ thống lớn lên.

## 8:30 Tổng kết (Hiếu, rồi Huy)

**Màn hình:** slide kết quả đo hoặc slide kết luận, bật camera cả hai người.

**Hiếu:**

> Tóm lại, qua demo có thể thấy container giúp khởi động cả hệ thống bằng một lệnh, scale và tự hồi phục khi có sự cố, và chạy được trên cả Compose lẫn Kubernetes. Thông lượng tăng 4,8 lần khi đi từ một lên sáu replica, và image nhỏ đi 6 lần so với cách viết Dockerfile đơn giản.

**Huy:**

> Compose là đủ khi chạy trên một máy, còn Kubernetes đáng dùng khi cần cập nhật không gián đoạn và tự động scale. Mã nguồn, báo cáo và hướng dẫn chạy đều có trong repository. Chúng em xin cảm ơn thầy và các bạn đã theo dõi.

---

## Sau khi quay

Kiểm tra lại: có đủ hai người, tiếng rõ, thời lượng trong khoảng 5-10 phút, đọc được chữ trong terminal. Xuất file MP4.
