---
layout: post
title: 'Cộng trừ trong Javascript. What the heck?'
date: 2020-05-23
categories: Nodejs JavaScript
---

Haiiia chào cả nhà iu của kem. Tuần này mình sẽ bắt đầu public posts về technical nhé.
AND HERE WE GOOOOO.

## Ủa, cộng trừ trong javascipt cũng là toán tử bình thường thôi mà?

Yeah đó là những gì mình nghĩ trong những ngày đầu làm quen với Javascript và nó thật sự đã hack bay mất não của mình. Cùng xem nó là gì nhé.

Ví dụ 1.
A normal plus operator in javascipt.

```javascript
const add = 1 + 2;
console.log(add); // Output: 3

const addString = 'Vu' + 'Dao';
console.log(addString); // Output: VuDao
```

Ví dụ 2.
Another normal subtract operator in javascript.

```javascript
const sub = 3 - 1;
console.log(sub); // Output: 2
```

Trời má, dễ vậy thôi mà có khác gì ngôn ngữ khác đâu đúng không?? Mình làm tiếp vài ví dụ nữa nhé.

```javascript
const add = 1 + '2';
console.log(add); // Output: ??

const addMore = '1' + '2';
console.log(add); // Output: ??

const sub = 3 - '1';
console.log(sub); // Output: ??

const reverseSub = 'mottram' - 1;
console.log(reverseSub); // Output: ??

const stringSubString = 'Vu' - 'Dao';
console.log(stringSubString); // Output: ??
```

Đù, thằng này có cộng trừ với string (chuỗi) nữa hả? Ai đời lại làm cái phép toán này bao giờ? Kết quả chắc chắn là `error` rồi.

À, mà đoán thử, cái đầu tiên `const add = 1 + '2';` và thứ hai `const add = '1' + '2';` kết quả chắc là bằng `3` giống như `const add = 1 + 2;` nhỉ??

Nghe có vẻ vô lý, nhưng là vô lý thật :)))

## Thật ra kết quả không phải là `error` và không chỉ là 3 không đâu.

Well, trong javascript khi mà mình thực hiện một phép cộng giữa số và chuỗi thì nó sẽ convert số thành chuỗi sau đó thực hiện phép cộng như chuỗi cộng chuỗi.

Ngược lại, khi thực hiện phép trừ thì javascript lại convert tất cả về số rồi mới thực hiện phép tính.
Nếu trong quá trình convert từ string về số mà không được thì kết quả trả về sẽ luôn là `NaN` (Not a Number).

Nói tóm lại, các bạn có thể nhìn vào kết quả của các ví dụ trên như sau:

```javascript
const add = 1 + '2';
console.log(add); // Output: '12'

const addMore = '1' + '2';
console.log(addMore); // Output: '12'

const addMoreMore = 1 + 2;
console.log(addMoreMore); // Output: 3

const addMoreMoreEver = 'Vu' + 100 + 'dodeptrai';
console.log(addMoreMoreEver); // Output: 'Vu100dodeptrai'

const sub = 3 - '1';
console.log(sub); // Output: 2

const reverseSub = 'mottram' - 1;
console.log(reverseSub); // Output: NaN

const stringSubString = 'Vu' - 'Dao';
console.log(stringSubString); // Output: NaN
```

Vấn còn mơ hồ đúng không? Vậy thì các bạn nhìn vào công thức ở dưới đây nhé.

```javascript
number + number = number;
number + string = string;
string + string = string;

number - number = number;
// (Nếu string không convert được về number thì kết quả sẽ là NaN)
number - string = number;
string - number = number;
```

Như vậy thì đây cũng là 1 mẹo nhỏ để các bạn có thể ép kiểu của biến trong quá trình làm việc mà không cần phải dùng tới các hàm có sẵn của Javascript. Ví dụ như:

```javascript
let aString = '10';
aString = aString - 0;

console.log(aString); // 10
console.log(typeof aString); // number
```

Nhìn lại thì javascript cũng không có gì gọi là phức tạp quá đúng không nhỉ? Chỉ giống như cô gái hơi đỏng đảnh chút xíu thôi, việc của mình là nắm bắt được tâm lý của cô gái ấy thì mọi thứ ắt sẽ thành công. Cứ cố gắng đi, biết đâu được đúng hông nào, không thành công cũng thành thụ :D

Trong bài mình có sử dụng khá nhiều các từ khóa như là _let_ _const_, bài viết sau mình sẽ chia sẻ cho các bạn biết nó là gì, và ES6 đã thay đổi toàn bộ cách khai báo biến của javascript như thế nào.

Nếu có thắc mắc gì các bạn hãy comment ngay ở bên dưới nhé.
Xin chào và hẹn gặp lại.
