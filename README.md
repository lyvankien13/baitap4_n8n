## Tên: Lý Văn Kiên
## Mssv: K225480106101
## Lớp: 58KTPM

**Bài tập 04:**  
# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
# 
## deadline : 23h59 ngày 25 tháng 5 năm 2026.
 #
 1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file **docker-compose.yml** chứa: 
- Mariadb: sử dụng **image: mariadb:latest** để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)
- Phpmyadmin: sử dụng **image: phpmyadmin:latest** để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1
- WordPress: sử dụng **image: wordpress:latest**, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường:  WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)
- Cloudflared: sử dụng **image: cloudflare/cloudflared:latest** , cấu hình ip tĩnh và thiết lập file config.yml để Cloudflare Tunnel biết chính xác subdomain nào cần kết nối tới dịch vụ Docker nào trên máy Ubuntu và public các sub-domain
- N8n : sử dụng **image: n8nio/n8n:latest**, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )

 + code file docker-compose.yml:
```yaml
   services:
  mariadb:
    image: mariadb:latest
    container_name: wordpress_db
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: kien
      MYSQL_PASSWORD: 123456
    volumes:
      - db_data:/var/lib/mysql
    restart: always


  phpmyadmin:
    image: phpmyadmin:latest
    container_name: wordpress_pma
    environment:
      PMA_HOST: mariadb
      PMA_USER: kien
      PMA_PASSWORD: 123456
    ports:
      - "8080:80"
    depends_on:
      - mariadb
    restart: always

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    environment:
      WORDPRESS_DB_HOST: mariadb:3306
      WORDPRESS_DB_USER: kien
      WORDPRESS_DB_PASSWORD: 123456
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - mariadb
    restart: always

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_automation
    environment:
      - TZ=Asia/Ho_Chi_Minh
      - WEBHOOK_URL=https://n8n.liviki.id.vn/
    volumes:
      - n8n_data:/home/node/.n8n
    ports:
      - "5678:5678"
    depends_on:
      - mariadb
    restart: always
    
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflare_tunnel
    volumes:
      - /home/bt4/.cloudflare:/home/nonroot/.cloudflare
    command: tunnel --config /home/nonroot/.cloudflare/config.yml run
    depends_on:
      - wordpress
      - phpmyadmin
      - n8n
    restart: always

volumes:
  db_data:
  wp_data:
  n8n_data:
```
+ code file config:
```yaml
tunnel: 2004cc7e-7020-4939-9d9f-1aa549e34966
credentials-file: /home/nonroot/.cloudflare/2004cc7e-7020-4939-9d9f-1aa549e34966.json

ingress:
  - hostname: wp.liviki.id.vn
    service: http://wordpress:80
  - hostname: pma.liviki.id.vn
    service: http://phpmyadmin:80
  - hostname: n8n.liviki.id.vn
    service: http://n8n:5678
  - service: http_status:404
```
#
#
 2. Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :
- pull các images về và chạy chúng (up -d)

 pulled các image và chạy <img width="1493" height="437" alt="image" src="https://github.com/user-attachments/assets/a09cf9da-381a-4d0d-896d-ce5eaf15a120" />
#
#
- Kiểm tra các service đã running ok (ko bị restart liên tục)
+ sử dụng lệnh docker compose ps để hiển thị các service đã running ok (up):<img width="1476" height="368" alt="image" src="https://github.com/user-attachments/assets/553e864f-14c2-4899-9579-425ae493d32a" />
#
#
- Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)
- Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)
- Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)
  <img width="1473" height="209" alt="image" src="https://github.com/user-attachments/assets/c8dd9ba9-3184-4c65-bdb3-eaeac05ffe41" />
#
#
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!
<img width="1911" height="970" alt="image" src="https://github.com/user-attachments/assets/02139761-d4d5-4511-b597-17fff7eda335" />

 #

-  Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)
  <img width="1427" height="943" alt="image" src="https://github.com/user-attachments/assets/cb1cf358-5a69-4ffb-87d6-21bea1ca989b" /><img width="1907" height="974" alt="image" src="https://github.com/user-attachments/assets/04c59209-f32f-44a6-8ae3-6a558e09daf2" />

 #

- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp
<img width="1919" height="956" alt="image" src="https://github.com/user-attachments/assets/15c09b54-05d8-4827-8d0a-c78fb768f6d0" />

 #
 #

- Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...
  <img width="1909" height="992" alt="image" src="https://github.com/user-attachments/assets/fe09a5a6-2b4f-44d4-96eb-cb8f43e6cf35" />
<img width="1919" height="960" alt="image" src="https://github.com/user-attachments/assets/ead2d1fe-d556-450b-a277-ce9820bb6c49" />

 #

- Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn Phát triển ứng dụng với mã nguồn mở
<img width="1904" height="966" alt="image" src="https://github.com/user-attachments/assets/d4dd7d67-40b5-446f-846f-b0d7b5a682a6" />
<img width="1919" height="955" alt="image" src="https://github.com/user-attachments/assets/19c6a1a1-57a6-436a-817c-52009a8c7de6" />

 # 
 #
 
-  Truy cập sub-domain3 để cấu hình n8n:
    + tạo tài khoản admin : nhớ điền đúng email<img width="1916" height="966" alt="image" src="https://github.com/user-attachments/assets/41f20241-1313-4432-ab9b-b9770c68ebc7" />
      #
    + Send me a Licence key<img width="1858" height="917" alt="image" src="https://github.com/user-attachments/assets/ba509760-66e3-4d2f-bd4d-dfdf5b9ff075" />
      #
    + License key được gửi về email, coppy key<img width="1920" height="978" alt="image" src="https://github.com/user-attachments/assets/56a380aa-942c-4be6-a5ee-2307aaa4b1ee" />
      #
    + Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.
    <img width="1914" height="920" alt="image" src="https://github.com/user-attachments/assets/7a26ea0f-62e7-4d6e-a9b8-dcd61c7fbaff" />
    <img width="803" height="962" alt="image" src="https://github.com/user-attachments/assets/d9d914e2-8f13-4495-8f74-a1fe21fecf40" />
    <img width="1918" height="929" alt="image" src="https://github.com/user-attachments/assets/2fb2c4bc-881f-4ebc-b7a6-210a118048b1" />
    <img width="1911" height="844" alt="image" src="https://github.com/user-attachments/assets/13e02e13-1b4b-4fe0-8ba3-4f9eabc493af" />
    <img width="1911" height="818" alt="image" src="https://github.com/user-attachments/assets/0529f082-fa5b-4a53-bcc1-db60c30b14a7" />

     + Create workflow, Add trigger node: tìm node: Telegram => OnMessage <img width="1900" height="967" alt="image" src="https://github.com/user-attachments/assets/de186fc4-83a2-4450-8b71-ecc9529d8603" />
      #
     + cấu hình Credential: Set up Credential => cần Nhập Access Token<img width="1898" height="968" alt="image" src="https://github.com/user-attachments/assets/e0907d87-ab89-4467-8789-64e19df8e1cb" />
      #
     + chát với bot @BotFather để đẻ ra bot mới của riêng mình<img width="1910" height="971" alt="image" src="https://github.com/user-attachments/assets/ce2d4eeb-6c04-4fcd-8ccf-c9d6cbfd7de8" />
     + lấy Token dán vào phần access token tại cấu hình Credential của telegram bên n8n<img width="1916" height="917" alt="image" src="https://github.com/user-attachments/assets/e9fba17e-447d-4bbb-a33b-dd1c56e8adb4" /><img width="1905" height="981" alt="image" src="https://github.com/user-attachments/assets/5feae0f6-df87-43ce-ae6d-1a630ec95f56" />
      #
     + chạy execute step và chát lần đầu với bot mới<img width="1876" height="814" alt="image" src="https://github.com/user-attachments/assets/07fca53f-d295-4869-a7c8-05268515c668" /><img width="1917" height="835" alt="image" src="https://github.com/user-attachments/assets/78fab637-9817-4a27-9a40-df6e9e09ed2a" />
      #
     + đã có output<img width="1919" height="956" alt="image" src="https://github.com/user-attachments/assets/35a73418-9b55-4e54-8255-67dc34264665" />
      #
     + Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY<img width="1894" height="966" alt="image" src="https://github.com/user-attachments/assets/c9817b67-e641-46ef-aa1d-9dc7185840a7" />
      #
     + Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys
cần tạo project mới, sẽ lấy được API KEY<img width="1914" height="939" alt="image" src="https://github.com/user-attachments/assets/aaf504ea-e7f9-408f-9cfe-34b29cc613ed" /><img width="911" height="629" alt="image" src="https://github.com/user-attachments/assets/b801673a-0211-4b3d-a0de-d18350230f82" /><img width="648" height="495" alt="image" src="https://github.com/user-attachments/assets/4f07e9a9-94b5-4a46-9dc8-bc149dc4cfa9" /><img width="693" height="368" alt="image" src="https://github.com/user-attachments/assets/85769de4-3a50-42df-8952-22b6bbca58c4" /><img width="955" height="643" alt="image" src="https://github.com/user-attachments/assets/e087b3a6-e0d6-4c57-a94d-902e79b2bf7d" />
      #
     + quay lại giao diện n8n tại mục set up Set up Credential của gg gemini và dán key API vào<img width="1906" height="955" alt="image" src="https://github.com/user-attachments/assets/8c12926c-e6c8-464a-a448-bec1bf550f43" />
      #
     + kéo thả nội dung đã chát với bot của telegram (phía bên trái) vào nội dung phần PROMPT,kết quả được {{ $json.message.text }}<img width="1864" height="820" alt="image" src="https://github.com/user-attachments/assets/41f74bec-192d-49bc-840f-94f26480d5fc" /><img width="674" height="417" alt="image" src="https://github.com/user-attachments/assets/965cd535-75ae-4c40-83c6-2e293850e2ec" />
      #
     + Turn on Output Content as JSON : để kết quả trả về dạng json<img width="582" height="454" alt="image" src="https://github.com/user-attachments/assets/ddf9ba21-6b3b-4824-9806-199c9e61a2dd" />
      #
     + thử nghiệm phần System message để AI thiết kế bài viết với vai trò là 1 nhà sáng tạo bài viết chuyên nghiệp<img width="1831" height="777" alt="image" src="https://github.com/user-attachments/assets/4cad4f67-3b12-4740-adfa-8bb59a583e4c" /><img width="534" height="88" alt="image" src="https://github.com/user-attachments/assets/4efa2295-f207-4cbb-8daa-c80eba8bfd84" />
      #
     + execute step để chạy ra output<img width="1882" height="825" alt="image" src="https://github.com/user-attachments/assets/4895b834-be51-4ec1-a2b0-9ea36cb193cd" />
      #
     + Add (nối tiếp vào sau node Message a model) node: Code in JavaScript.
     + code javascript:
```
$input.item.json.content.parts[0].text;

try {
  // Thực hiện ép kiểu chuỗi String thành Object JSON thực sự
  const parsedData = JSON.parse(rawText);
  
  // Trả về kết quả sạch gồm tiêu đề và nội dung HTML bài viết
  return {
    json: {
      status: parsedData.status,
      title: parsedData.data.title,
      html_content: parsedData.data.content
    }
  };
} catch (error) {
  // Trường hợp AI sinh chuỗi lỗi không đúng định dạng JSON
  return {
    json: {
      status: "error",
      message: "Không thể parse JSON từ AI",
      raw_text: rawText
    }
  };
}
```

  #
  
  #
  + sau đó execute step<img width="1881" height="634" alt="image" src="https://github.com/user-attachments/assets/cba2ec89-3684-498c-928b-439628529df8" />
      #
     + Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post<img width="1881" height="806" alt="image" src="https://github.com/user-attachments/assets/d86d5061-0854-405e-a811-c991046bf6e8" />
     + Set up Credential<img width="1874" height="816" alt="image" src="https://github.com/user-attachments/assets/8679354e-f2d0-48ab-b763-1ce16dc346af" /><img width="1832" height="816" alt="image" src="https://github.com/user-attachments/assets/07f46203-f7d8-4fe1-82c6-560f3d001798" />
      #
     + vào wp tại url: https://sub-domain1/wp-admin => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential<img width="1898" height="949" alt="image" src="https://github.com/user-attachments/assets/f4055321-6a33-4320-815c-25ce0b55403f" /><img width="1920" height="916" alt="image" src="https://github.com/user-attachments/assets/48d16060-d399-4950-bbca-172a6d822884" /><img width="1911" height="919" alt="image" src="https://github.com/user-attachments/assets/8b121391-e091-4131-a829-41597ab6f6bd" /><img width="1911" height="874" alt="image" src="https://github.com/user-attachments/assets/d713fc80-4893-43b0-80a4-b8c456a150a4" />
      #
     + điền user name là user name của tài khoản word press<img width="1346" height="697" alt="image" src="https://github.com/user-attachments/assets/706733ec-b25e-4b6a-825c-67d273ce8dfa" /><img width="1901" height="990" alt="image" src="https://github.com/user-attachments/assets/53c70d95-a024-4ed3-8ba8-06816f1449c9" />
      #
     + Wordpress URL: điền giá trị https://sub-domain1/ (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)
Ignore SSL Issues (Insecure): TURN ON<img width="1920" height="737" alt="image" src="https://github.com/user-attachments/assets/f2aef75e-279d-4443-ac2b-cea00ad957c5" /><img width="1243" height="494" alt="image" src="https://github.com/user-attachments/assets/e6443fdb-3066-49b5-a0d3-11b5a607386d" /><img width="1346" height="704" alt="image" src="https://github.com/user-attachments/assets/25b5797f-8d04-4c45-9ff4-98faf674f0d9" />
      #
     + Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content
Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp) <img width="1874" height="820" alt="image" src="https://github.com/user-attachments/assets/e24b2342-b53a-441a-be94-dc30f39422c6" />
      #
     + output hiển thị bài viết đã tự động đăng<img width="1892" height="830" alt="image" src="https://github.com/user-attachments/assets/f0e9b8b0-c159-443b-8762-b6cdf510c76c" />
      #
     + publish dự án<img width="1897" height="1040" alt="image" src="https://github.com/user-attachments/assets/4f9ca6cc-8fb6-4ee6-bbae-63675c45293e" /><img width="1404" height="691" alt="image" src="https://github.com/user-attachments/assets/759ce491-ebc8-4e1b-b947-cd8e1dd3e1b0" /><img width="1867" height="957" alt="image" src="https://github.com/user-attachments/assets/a1804821-ea37-49c2-9b40-8be9be6e1ae3" />
      #
     + chat với bot để ai tự tạo bài viết<img width="1901" height="853" alt="image" src="https://github.com/user-attachments/assets/e9280c34-7447-42f7-8494-19a5ed810025" />
      #
     + bài viết mới tự động được đăng<img width="1897" height="990" alt="image" src="https://github.com/user-attachments/assets/b04e9486-cd23-438a-a0fb-0efdb0d6d8dd" /><img width="1917" height="963" alt="image" src="https://github.com/user-attachments/assets/ef70a4e2-3639-4c6a-b95c-a0448fdad06e" />
#
#
==> Nhận xét thành quả đạt được: Thành quả lớn nhất mà em đạt được là làm chủ quy trình triển khai hạ tầng Web và Tự động hóa (DevOps) thực tế: từ việc đóng gói 5 dịch vụ bằng Docker, làm chủ bản đồ định tuyến mạng bằng file Cloudflare config.yml, đưa hệ thống nội bộ ra Internet qua tên miền cá nhân, cho đến việc sẵn sàng kích hoạt luồng AI tự động hóa với n8n và Gemini.












































    




