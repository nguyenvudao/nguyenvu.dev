---
layout: post
title: 'Tích hợp Google Authenticator vào trang web.'
date: 2020-06-11
categories: Nodejs JavaScript
---

Chào mọi người, tuần trước Client có gửi cho mình một cái requirement yêu cầu tích hợp Google Authenticator để xác minh hai bước (Two-Factor Authentication) vào trang quản trị `Admin` của họ. Client của mình là một công ty hoạt động trong lĩnh vực finance nên yêu cầu bảo mật cũng phải tốt một chút. Sau một tuần ăn nằm với nó thì mình cùng với team cũng implement xong vụ này nên tiện share lên cho mọi người cùng đọc vì trong quá trình research mình cũng thấy resource nói về việc 2FA integration cũng mù mờ và khá là hạn chế. Chính vì thế mà hôm nay mình sẽ hướng dẫn các bạn cùng integrate cái chức năng này vào trang web héng.

À lưu ý là backend sẽ dùng [Nodejs][nodejs-link] còn frontend sẽ là [Reactjs][reactjs-link] nhé. Thực ra khi đã hiểu được flow của nó thì platform hay ngôn ngữ nào cũng không phải là vấn đề nữa :D

### What is it?

Theo [Authy][authy-link] thì Two-Factor Authentication là một lớp bảo mật bổ sung để xác thực rằng người dùng đang login vào tài khoản mà họ có quyền truy cập. Sau khi đăng nhập bằng email / password, user sẽ phải cung cấp thêm một (số) thông tin bảo mật để xác thực. Thông tin này có thể là:

- Something you know: Nó có thể là một cái mã pin, một cái mã giống như OTP để chuyển khoản ngân hàng hoặc có thể là câu hỏi bảo mật,...
- Something you have: Thường thường thì nó là những gì mà user đang sở hữu, như là CMND hoặc hardware token (cái này ở VN thấy mỗi ngân hàng dùng).
- Something you are: Vân tay, vống mắt,... cái này thì quen quá rồi. Nếu chưa biết thì mình có mò được cái gif trên [Tenor][tentor-link] nè :v
    <!-- ![Jack-Authentication](/assets/img/jackjack-authentication.gif) -->
  <center><img src="/assets/img/jackjack-authentication.gif"></center>

Okay, có rất nhiều cách để có thể xác thực hai bước, nhưng trong bài này mình sẽ hướng dẫn các bạn xác thực với Google Authenticator. Để sử dụng được phương thức xác thực này, mình sẽ cần một cái smart phone và ứng dụng Google Authenticator có thể tải về trên [IOS][ios-link] và [ANDROID][android-link].

À mà cái hay của nó là user có thể thực hiện được việc authentication mà không cần điện thoại phải kết nối tới mạng, hồi trước mình nghĩ là Google generate ra sẵn một đống mã cơ chứ, nhưng sự thật thì không phải là như thế , cùng xem nào :)).

### How does it work?

Mấy cái cách xác thực kia mình không bàn, ở đây mình sẽ nói phương thức xác thực hai lớp bằng Google Authenticator thôi ha.

- Khi mà user bật tính năng 2FA, thì mình sẽ tạo một cái secret key, dựa trên secret key mà sinh ra một cái QRCode để user quét bằng ứng dụng Google Authenticator.
- User's smart phone sau đó sẽ generate ra một cái OTP code và sẽ thay đổi mỗi 30 giây một lần. Code này sẽ dùng để xác thực khi user đăng nhập.
  <!-- <center>![GG-Auth](/assets/img/ggAuth-dashboard.png)</center> -->
   <center><img src="/assets/img/ggAuth-dashboard.png"></center>

Đấy là góc nhìn của enduser, nghe đơn giản lắm đúng không? Nhưng còn cái business khi integrate ở phía backend thì như thế nào??

### How the backend handles the Authentication?

Greate, tới phần mà mình thích nhất rồi đây :) Cách mà backend handle việc authentication này cũng khá là đơn giản, và nó chính xác là những gì mà mình đang làm ở phía server của mình.
`Google Authenticator` là ứng dụng hiện thực hóa thuật toán **Time-Based One-Time Password (TOTP)**. Cục này gồm những thứ tạm gọi như sau:

- Một cái token (secret code).
- Một cái param truyền vào là thời gian hiện tại.
- Một cái function để mã hóa.

#### Token (Secret code)

Token thường sẽ có dạng là:

```javascript
'CZKUCYAXJFAFWLR4';
```

Cái này chính là token dùng để sinh ra cái QrCode cho user quét, user cũng có thể add manual cái token này vào ứng dụng `Google Authenticator` để sử dụng :D Cái QrCode sẽ chứa cái token tương tự URL như sau:

```javascript
'otpauth://totp/{USER_EMAIL}?secret={USER_TOKEN}&issuer={SERVICE_NAME}';
```

Hai cái format này mình đang sử dụng luôn trong project và cái token ở trên là của default admin luôn =))

#### Current Time input

Cái giá trị thời gian này sẽ được dùng để mã hóa với cái token ở trên rồi sinh ra một cái passcode gồm 6 ký tự. Server và điện thoại sẽ dựa trên giá trị này để thực hiện việc encryption rồi so sánh cái passcode. Nếu passcode mình nhập và passcode trên điện thoại là giống nhau thì có nghĩ là mình được quyền đi tiếp :D

#### Signing Function

Signing function được sử dụng ở đây là **HMAC-SHA1**. HMAC có nghĩa là [Hash-based message authentication code][hmac-link] là một thuật toán sử dụng hàm mã hóa một chiều(ở đây là SHA1) để mã hóa giá trị.

Cơ bản là backend sẽ cần những thứ như trên để decode, encode rồi bla bla bla... và thay vì phải handle một đống thứ ở trên thì mình tìm được một cái package khá ổn để handle việc này thay cho mình. Nó là [OTPLib][otplink], package được release lần cuối vào khoảng năm tháng trước tính từ khi bài blog này được viết (6/2020) và số lượng download ~60K cũng khá ổn. Thế nên mình quyết định dùng package này vào dự án luôn :D

### Okay, let's integrate Google Authentication to our website ;)

#### 0. Generate the QrCode.

Đầu tiên mình cần làm là generate ra một cái QrCode để user quét. Token cũng sẽ được generate tại thời điểm này, lưu ý là token chỉ nên được generate ra một lần duy nhất mà thôi.

Okay, để generate cái token, và QrCode. Ta cần làm như sau:

```javascript
const QRCode = require('qrcode');
const { authenticator } = require('otplib');

// Generate Token.
const secretCode = authenticator.generateSecret();
// Prepare the qrcode
const qrCodeStr = `otpauth://totp/admin@dirox.net?secret=${secretCode}&issuer=Atira`;

// Generate the QrCode.
// Mình cũng không hiểu tại sao nó cần await chỗ này, vì trong docs nó hỗ trợ callback, với promise nên nhét await vào, để hôm khác test thử xem bỏ luôn cái await như nào :D
const qrcode = await QRCode.toDataURL(qrcodeStr);
```

Thế là xong phần generate token & qrcode.

#### 1. Verify the passcode.

[authy-link]: https://authy.com/what-is-2fa/
[tentor-link]: https://tenor.com/view/jack-jack-edna-mode-incredibles2-lollipop-cute-gif-14544010
[ios-link]: https://apps.apple.com/vn/app/google-authenticator/id388497605?l=vi
[android-link]: https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=vi
[nodejs-link]: https://nodejs.org/en/
[reactjs-link]: https://reactjs.org/
[hmac-link]: https://en.wikipedia.org/wiki/HMAC
[otplink]: https://www.npmjs.com/package/otplib
