---
layout: post
title: 'Promise all vs Promise allSettled: Xử lý performance trong JavaScript.'
date: 2020-06-04
categories: Nodejs JavaScript
---

Hế lô mọi người, chào mừng mọi người quay trở lại với blog cá nhân của mình. Tình hình là mình mới chuyển nhà sang bên Nơ Trang Long, Bình Thạnh (cách công ty tầm 2km mà sáng mở mắt ra là kẹt xe, tối từ công ty về cũng kẹt xe =.= mệt vcđ) nên thời gian có nhiều hơn đôi chút nên tính publish nhiều bài viết hơn nhưng mà dạo này toàn đánh liên minh với tụi cùng phòng thôi =)))) nên đây sẽ là bài đầu tiên trong số đó :D

## Promise All: I can help you to handle the asynchronous tasks better.

Trong công việc, mọi người chắc hẳn sẽ gặp rất nhiều trường hợp cần thực hiện CURD (Create, Update, Read, Delete) các instance trong database. Và Javascript là ngôn ngữ single thread cũng như cơ chế Blocking / Non-Blocking I/O của NodeJS mà mỗi lần thực hiện request ta sẽ (phải) chờ kết quả trả về. Điều này khiến cho performance của App trở nên cực kì tệ hại. Ví dụ như:

```javascript
const updatePerson = async () => {
  await User.find({ where: { name: 'VuDao' } }); // time cost: 5s.
  await Book.find({ where: { price: { lt: 200 } } }); // time cost: 5s.
};

await updatePerson(); // time cost: 10s.
```

Ok, nhìn vào ví dụ trên ta có thể thấy việc tìm 2 instance trong database này không hề phụ thuộc vào nhau (tức là muốn tìm sách thì không cần quan tâm việc user `VuDao` có tồn tại trong bảng **User** hay không) nhưng mỗi lần gọi tới cái hàm này ta phải chờ ~10s để nó kết thúc, điều này khiến cho performance của App giảm rất nhiều. Nodejs vốn nổi tiếng với performance khá tốt, có thể chịu vài chục nghìn request đồng thời trong 1 giây. Ấy vậy mà gọi 1 cái request đợi hết 10s thì bỏ mama mất rồi.

Và để giải quyết vấn đề trên, **Promise.all** được ra đời, hứa hẹn cho một tương lai mới cho những tasks CURD như thế này. Thay vì phải đợi từng request như ở trên, ta có thể viết lại như sau:

```javascript
const updatePerson = async () => {
  await Promise.all([
    User.find({ where: { name: 'VuDao' } }),
    Book.find({ where: { price: { lt: 200 } } }),
  ]);
};

await updatePerson(); // time cost: ~5s.
```

Bùm, thời gian chờ lúc này đã giảm xuống còn ~5s (lưu ý là thực tế sẽ không có việc mỗi task mất 5s, 2 task mất 10s thì dùng promise.all sẽ giảm còn 5s đâu nhé. Mình chỉ đang ví dụ thôi). Toẹt dời đúng không? Thay vì phải đợi 10s để xong việc thì bây giờ thời gian đã giảm đi còn một nửa. Vậy `Promise.all` là gì mà lại làm được việc này thế nhờ?

Theo như [Mozilla][promise-all] thì `Promise.all([ARRAY])` trả về một _Promise_ mới và _Promise_ này chỉ kết thúc khi tất cả các promise trong `ARRAY` kết thúc hoặc có một promise nào đó quăng ra lỗi. Kết quả mà _Promise_ này trả về sẽ là một mảng chứa kết quả của tất cả các promise theo đúng thứ tự truyền vào hoặc kết quả lỗi của promise gây ra lỗi.

Tóm gọn lại hơn: Nếu một task trong `Promise.all` bị lỗi thì tất cả sẽ dừng lại và `Promise.all` sẽ quăng lỗi ra ngay lập tức, còn không thì sẽ đợi toàn bộ các tasks hoàn thành. Điều tuyệt vời mà `Promise.all` đã thực hiện là vậy đó :D

Nhưng mọi thứ chưa bao giờ là hoàn hảo cả, `Promise.all` giúp các lập trình viên javascript tiết kiệm được thời gian, tăng performance nhưng lại gây ra một vấn đề khác: `Promise.all` sẽ bị dừng nếu như có một task bị lỗi.

Vấn đề mình đã nói ở trên, việc thực hiện tìm kiếm _User_ và _Book_ không hề liên quan tới nhau mà để giảm được thời gian chờ nhưng lại phải chịu những rủi ro không đáng có thì chúng ta lại có một method khác, cũng thực hiện những công việc giống `Promise.all` làm nhưng cải thiện được nhược điểm của `Promise.all` chính là: Không dừng lại nếu như có một task bị lỗi.

## Promise.allSettled: Anything Promise.all can do, Promise.allSettled do better.

Không nói nhiều, `Promise.allSettled` sinh ra để khắc phục những nhược điểm của `Promise.all`. Phương thức này được đem vào Nodejs ở phiên bản `12.9` thì phải (anh senior trong team mình chỉ cho mình xài cái này chứ mình có biết đâu :v). Khi sử dụng `Promise.allSettled`, nó sẽ trả về một _Promise_ mới và cái _Promise_ này sẽ chờ cho tất cả các tasks hoàn thành, mặc kệ nó có bị lỗi hay không.

Nhìn vào ví dụ dưới đây mình sẽ có cái nhìn tổng quan hơn:

```javascript
Promise.allSettled([
  Promise.reject('This failed.'),
  Promise.reject('This failed too.'),
  Promise.resolve('Ok I did it.'),
  Promise.reject('Oopps, error is coming.'),
]).then((res) => {
  console.log(`Here's the result: `, res);
});
```

**Out put**

```javascript
> Here's the result: [
  { status: 'rejected', reason: 'This failed.' },
  { status: 'rejected', reason: 'This failed too.' },
  { status: 'fulfilled', value: 'Ok I did it.' },
  { status: 'rejected', reason: 'Oopps, error is coming.' }
]
```

Thằng `Promise.allSettled` hay ở chỗ là kết quả trả về sẽ bao gồm cả status và reason. Việc xử lý dữ liệu sẽ tốn thêm một bước nhưng đảm bảo được request sẽ hoàn thành.

Trên đây là sự khác nhau và cách sử dụng của `Promise.all` và `Promise.allSettled`. Nhanh và gọn, không lói nhìu.

Cảm ơn các bạn đã đọc tới đây, có gì thắc mắc thì comment hoặc gửi tin nhắn tới [Facebook][vu-dao] của mình nhé. Thân chào!

[promise-all]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
[vu-dao]: https://www.facebook.com/ngvu.it
