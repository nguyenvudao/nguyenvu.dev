---
layout: post
title: 'Firebase đã giúp mình bắn notification và quản lý việc nhắn tin của các users với nhau như thế nào? (P.1)'
date: 2020-09-26
categories: Nodejs Firebase Typescript
---

Hế lô mọi người, lâu quá không gặp :sweat_smile:. Nếu không nhầm thì cũng đã khoảng 3 tháng 16 ngày tính từ lần cuối mình publish bài viết về [Google Authenticator] ý. Vấn đề là sau hôm viết bài đấy thì mình join vào một project khác mà theo lời PM (Project Manager) thì project đang trong giai đoạn gấp rút cực kỳ và có thể bị cháy nếu như tình hình hiện tại không được cải thiện :D. Thế là mình lặn ngụp trong đấy mấy tháng trời, vừa làm vừa phải research thêm rất nhiều công nghệ khác (Firebase, Stripe, Nestjs, Odoo, Docker....) nên thời gian rảnh không có nhiều, bù lại thì mình sẽ viết thêm blog để giới thiệu với mọi người từng công nghệ mà mình đã tích hợp. Hiện tại thì project mình nhắc tới ở trên hôm nay đã là sprint cuối cùng lúc mình viết bài blog này (25-09-2020) và tuần sau sẽ bắt đầu bước vào "one month guarantee phase" nên thời gian hiện tại cũng xem như hơi hơi rảnh =)))

Có một tin không biết là vui hay buồn vì mình mới lại tham gia vào project khác, nhưng mà khó nhằn hơn vì đây là one man army project :))) Đại loại là một mình một project, mình được làm mọi thứ từ initial project, quyết định codebase, package, library, technical,... nào sẽ được sử dụng trong project này :stuck_out_tongue_closed_eyes: và cũng vì thế mà mình phải tốn nhiều thời gian hơn cho việc research, tránh việc thiếu kiến thức mà làm ảnh hưởng tới tiến độ của project. À, bà chị BA (Business Analyst) deal với khách hàng là việc initial project sẽ tốn ba ngày nhưng thực tế thì mình mất hết năm ngày cho việc này :sweat_smile: - một dấu hiệu cho thấy các bài blog sau có thể bị delay tới tận năm 2021 =))))

Thôi lan man vậy thoai, trong bài này mình sẽ nói về [Firebase] - một platform khá ngon lành được backed bởi ông khổng lồ Google.

   <center><img src="/assets/img/one-man-army.jpg"></center>
   Đây là mình trong dự án hiện tại :joy:
   
### I. Firebase là gì?

Mấy bài blog trước mình hay giải thích khái niệm của các service mà mình nhắc tới, thôi giờ lười gõ quá nên mọi người vào trang chủ của [Firebase] đọc tạm ha =)) Cái chúng ta sẽ dùng trong đây sẽ là [Firebase Cloud Function] và [Firebase Realtime Database]

### II. Gửi notification và quản lý chatting giữa user với user.

Có bao giờ mọi người sử dụng những ứng dụng trên điện thoại và thắc mắc cách mà app với server giao tiếp với nhau như thế nào để gửi và show up cái notification trên ứng dụng không nhỉ?

Lấy ví dụ cho đơn giản heee, mình đang lướt Facebook vào một ngày đẹp dời, cái tự dưng crush nhắn tin rủ đi uống trà sữa. Lúc này mình sẽ nhận một thông báo có tin nhắn mới và trong inbox của mình sẽ xuất hiện một tin nhắn mà crush vừa gửi.

Đấy là ví dụ mình assume là thằng Facebook dùng Firebase để quản lý messages và notification ha, chứ thực tế thì không phải vậy rồi =))

Quay lại nào, dưới góc nhìn của một backend developer thì có rất nhiều cách để làm việc này, từ cách tự build một service hoặc sử dụng Firebase để việc quản lý trở nên nhẹ nhàng và nhanh gọn hơn.
Ok, ở đây thì Facebook đã dùng một đoạn _token_ được gọi là _FcmToken_, các bạn có thể đọc thêm về [Firebase Cloud Messaging] để hiểu hơn về cách hoạt động của nó. Với mỗi thiết bị sẽ tương ứng với một _FcmToken_, cái _FcmToken_ này là bất biến và duy nhất sẽ được dùng để định danh thiết bị của user.

Mỗi khi userA nhắn tin tới userB, [Firebase Cloud Messaging] sẽ dựa vào cái _FcmToken_ mà bắn notification cũng như message về cho userB và ngược lại nếu như userB thực hiện việc chatting với userA.

Đó là cách mà dự án hiện tại mà mình đang sử dụng, hơi khác một chỗ là thay vì bắn notification trực tiếp từ server của mình (đương nhiên là dựa vào Firbase rồi) và lưu lại thông tin notification cũng như message thì mình đưa hết lên [Firebase Realtime Database] để quản lý luôn. Cách này giúp mình đỡ phải quản lý manual hai thứ trên, cũng như đảm bảo việc "realtime" diễn ra đúng như tên gọi của nó.

### III. Tự động gửi notification với Firebase Cloud Function.

Cloud Function là một hàm xử lý sẽ trigger các event của Realtime Database mỗi khi có sự thay đổi. Ở đây mình sẽ sử dụng event `onCreate()` để mỗi lần add một record vào realtime database thì cloud function sẽ được thực thi và gửi notification về device của user. Ngoài `onCreate()` ra thì còn các event khác như `onWrite()`, `onUpdate()` và `onDelete()`, chi tiết hơn thì các bạn có thể vào [database event] để xem thêm.

#### 1. Cài đặt Nodejs và Firebase CLI.

> Node.js versions 8, 10, and 12 (Beta) are supported. See Set runtime options for important information regarding ongoing support for these versions of Node.js.

Chúng ta cần cài đặt [Nodejs] để viết code cũng như [Firebase CLI]. Đối với nodejs thì mình khuyến khích nên sử dụng version 10 hoặc 12, nhưng tốt nhất cứ latest mà phang, có nhiều features mới để khám phá như [Promise.allSettled] thì tội gì phải dùng mấy bản cũ cũ đúng không? Cơ mà firebase function thì phải dùng đúng bản nha :v

Sau khi cài đặt xong nodejs, mình cần phải cài thêm Firebase CLI nữa, các bạn có thể cài đặt qua NPM (Node Package Manager) bằng cách:

> npm install -g firebase-tools

#### 2. Khởi tạo project.

Đầu tiên là login vào firebase cái đã:

> firebase login

Tiếp theo gõ:

> firebase init functions

Trong quá trình init, nó sẽ hỏi mình vài câu hỏi, cứ chọn những option nào phù hợp với bản thân nhé.
Sau khi quá trình initial hoàn tất, cấu trúc của project sẽ nhìn giống như này này:

```
myproject
 +- .firebaserc    # Hidden file that helps you quickly switch between
 |                 # projects with `firebase use`
 |
 +- firebase.json  # Describes properties for your project
 |
 +- functions/     # Directory containing all your functions code
      |
      +- .eslintrc.json  # Optional file containing rules for JavaScript linting.
      |
      +- package.json  # npm package file describing your Cloud Functions code
      |
      +- index.js      # main source file for your Cloud Functions code
      |
      +- node_modules/ # directory where your dependencies (declared in
                       # package.json) are installed
```

#### 3. Viết function để tự động gửi notification mỗi khi có record mới được thêm vào nào.

Đây, mình lôi luôn code mà mình đã viết trong dự án luôn :v Đương nhiên vì tính thương mại nên mình sẽ sửa lại gần như toàn bộ nội dung :sweat_smile: nhưng cách thức hoạt động thì vẫn gần như giống như nhau nhé.

```javascript
/** ++ Imported modules ++ */
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
/** -- Imported modules -- */
admin.initializeApp();

export const TriggerOnCreateNotifications = functions.database
  .ref('notifications/{modelId}')
  .onCreate((snapshot, context) => {
    /** ++ Get the added record in Realtime Database ++ */
    const notification = snapshot.val();
    /** -- Get the added record in Realtime Database -- */
    const { message = '', fcmTokens = [], modelId = '' } = notification;

    if (!fcmTokens || !fcmTokens.length) {
      throw new Error('User does not have any tokens!');
    }

    /** ++ Prepare the message body to send to the user's device ++ */
    const payload = {
      data: {
        isRead: 'false',
        type,
        modelId,
        sender,
      },
      notification: {
        body: message,
        title,
      },
    };
    /** -- Prepare the message body to send to the user's device -- */

    // Send notification directly to the user's device.
    return admin.messaging().sendToDevice(fcmTokens, payload);
  });
```

Đoạn code phía trên chỉ đơn giản móc những record vừa được thêm vào Realtime Database, xử lý dữ liệu (kiểm tra xem fcmToken có valid không), sau đó gửi trực tiếp về cho user.

Mình có dùng thư viện [Quickstart JS] để test việc gửi nhận tin, mọi người có thể đọc chỉ dẫn ở đường link trên để test thử hen :v Hình dưới này là mình test với payload

```javascript
const payload = {
  fcmTokens: [
    'd0LHRwvckni8f1GEDpMYmU:APA91bEePS1EOp-yucw7Xg11gA1GuE46tBE-EDCMkI76sykeeNCAZwVUP0zfLTDt6ZLLUuoHaXM9AgiC5yx_X5gsAUW4pYl7W1ilNORiC3e_bUfkrka_kZWmHJSBN1-wJZwIBlf7gCvy',
  ],
  message:
    'This notification was sent from the Realtime Database by the Cloud Function ahihi',
  modelId: 'd303af19-ca90-489d-9d21-c0cffdb825aa',
  isRead: 'false',
  sender: 'Vu Dao',
};
```

<center><img src="/assets/img/firebase-notification-example.png"></center>

### IV. Kết thúc.

Phần Firebase function mình xin kết thúc tại đây, đang ngồi ngoài The Coffee House dưới Biên Hoà, đói bụng quá nên kiếm gì ăn rồi hôm sau sẽ giới thiệu về phần messasing nhé :))))

Anw, nếu có thắc mắc gì thì cứ inbox [Facebook] mình hoặc comment ở dưới nha, đừng quên là sắp tới sẽ có rấc rấc nhiều technicals mới sẽ được giới thiệu trong này, mọi người nhớ canh me vào xem nha =))))

Love you all :\*

[google authenticator]: https://nguyenvu.dev/nodejs/javascript/2020/06/11/implement-google-authenticator.html
[firebase]: https://firebase.google.com/
[firebase cloud function]: https://firebase.google.com/docs/functions
[firebase realtime database]: https://firebase.google.com/docs/database
[firebase cloud messaging]: https://firebase.google.com/docs/cloud-messaging/ios/client
[database event]: https://firebase.google.com/docs/functions/database-events#set_the_event_handler
[nodejs]: https://nodejs.org
[firebase cli]: https://github.com/firebase/firebase-tools
[promise.allsettled]: https://nguyenvu.dev/nodejs/javascript/2020/06/04/promise-all-vs-promise-allSettled.html
[quickstart js]: https://github.com/firebase/quickstart-js/tree/master/messaging
[facebook]: https://www.facebook.com/ngvu.it
