---
layout: post
title: 'Cách mà ES6 hủy hoại cuộc đời của "var"'
date: 2020-05-30
categories: Nodejs JavaScript
---

Chào mọi người, lại là mình đây. Hôm nay viết blog sớm vì laptop cá nhân của mình đang bị em gái trưng dụng nên đành tranh thủ ít phút trên công ty để viết blog :D. Ok, chúng ta sẽ bàn về những cái mới của ES6 và cách mà nó tiêu diệt **var** lạnh lùng như cách mà crush bơ mình vậy :)

## ES6 là gì vậy nhỉ?

ES6 (ECMAScript 6) hay còn được gọi là ES2015 (vì ra đời vào năm 2015) là một phiên bản của chuẩn ECMAScript. ECMAScript chính là tiêu chuẩn do hiệp hội các nhà sản xuất máy tính Châu Âu đề xuất làm tiêu chuẩn của ngôn ngữ JavaScript.
Theo [Wikipedia][wiki] thì phiên bản mới nhất hiện tại đang là ES10 (ECMAScript 10), được published vào tháng 6 năm 2019 đem đến cho các lập trình viên nhiều tính năng mới hơn như là _.flat_ dành cho _array_.

## Được rồi, ES6 là một phiên bản của JavaScript, vậy thì liên quan gì tới việc "var" bị khai tử chứ?

Đầu tiên chúng ta điểm qua vài tính năng đã được giới thiệu trong ES6:

0. Arrow function.
1. Rest parameter.
2. Destructuring assignment.
3. Default parameters.
4. Template literals.
5. Multi-line string.
6. Enhanced object literals.
7. Promises.
8. Classes.
9. Block - Scroped construct "let" and "const" (A var killer).

Cái ta quan tâm nhất là dòng số 10 (_let_ và _const_).

Nếu như trước khi ES6 được published chúng ta chỉ có duy nhất 1 từ khóa _var_ để khai báo biến thì bây giờ đã có thêm _let_ và _const_. Nhưng vấn đề là tại sao hai thằng này xuất hiện lại khiến cho việc sử dụng _var_ chở nên lỗi thời nhỉ?

Để giải thích cho việc này, chúng ta cần phải tìm hiểu sự khác nhau giữa chúng nó là gì.

## Var, let and const. Sự khác nhau là gì?

Mình cùng nhìn vào cái ví dụ kinh điển nhất:

```javascript
{
  let variable = 16;
}
console.log(variable); // ReferenceError: variable is not defined

{
  var variable = 10;
}
console.log(variable); // 10

const object = { hello: 'vudao' };
console.log(object); // Object { hello: "vudao" }
object.hello = 'nguyenvu';
console.log(object); // Object { hello: "nguyenvu" }
object = { type: 'object' }; // TypeError: invalid assignment to const `object'
```

### var == function scope.

**Function scope** nghĩa là những gì mà nó làm trong một function.

Okay, phải nói thêm một điều nữa là nếu như **var** được dùng ở ngoài function thì scope của **var** sẽ là **global**.
Cái ví dụ thực tiễn mà bất kì một lập trình viên Javascript nào cũng gặp trước khi có ES6 chính là việc lặp qua một mảng:

```javascript
for (var i = 0; i < 5; i++) {
  // do stuff
}
console.log('Outside a loop: ', i); // i = 5
```

Vấn đề là thằng biến **i** chưa bao giờ được khai báo bên ngoài và chỉ dùng trong vòng lặp, thế nhưng khi kết thúc vòng lặp thì biến **i** bị gán giá trị bằng 5. Điều này gây ra khá nhiều bất tiện bởi vì mỗi lần dùng vòng lặp là ta lại override đi override lại cùng một biến, khiến cho việc quản lý gặp rất nhiều khó khăn.

### let == block scope.

**Block scope** nghĩa là những gì mà nó xảy ra trong một khối mã _{}_.
Khi mà một biến được khai báo với từ khóa **let**, phạm vi hoạt động của biến sẽ bị giới hạn lại trong cái scope gần nhất đó, những thằng bên ngoài sẽ không thể truy cập tới nó.

Và cách mà ES6 cứu rỗi việc sử dụng vòng lặp với **let**:

```javascript
for (let i = 0; i < 5; i++) {
  //do stuff
}
console.log('Outside a loop: ', i); // ReferenceError: i is not defined
```

Thật toẹt dời, biến _i_ được khởi tạo và sử dụng cho duy nhất một vòng lặp, khi vòng lặp kết thúc thì _i_ cũng quyên sinh cùng với vòng lặp luôn, điều này giúp cho lập trình viên Javascript quản lý các biến của mình tốt hơn, không sợ bị thằng khác override lại giá trị của nó :D

### const == block scope but immutable.

**const** thật ra cũng giống như **let** nhưng khác ở chỗ là khi khai báo biến với _let_, chúng ta có thể cập nhật giá trị cho nó. Nhưng một khi biến được khai báo bằng _const_ thì giá trị của nó sẽ là bất di bất dịch.

```javascript
let name = 'VuDaodeptraikhoaito';
name = 'Nguyen Vu';
console.log(name); // Nguyen Vu

const anotherName = 'VuDao';
anotherName = 'Nguyen Vu'; // TypeError: Assignment to constant variable.
console.log(anotherName);
```

### Hoisting of **var**

Ngoài việc gây bất tiện cho việc quản lý biến, _var_ còn đem lại một vấn đề khác chính là **hoisting**.

```javascript
console.log(variable); // undefined
var variable = 'This is hoisting.';

console.log(constVariable); // ReferenceError: Cannot access 'constVariable' before initialization
const constVariable = 'This is not.';

console.log(letVariable); // ReferenceError: Cannot access 'letVariable' before initialization
const letVariable = 'Also this.';
```

_Hoisting_ là một cơ chế của Javascript khi mà các variable và phần khai báo function được đem lên đầu phạm vi của chúng trước khi thực thi code. Điều này có nghĩa là khi ta thực hiện một công việc như sau:

```javascript
console.log(hello);
var hello = 'Xin chao';
```

Điều này tương đơi với việc này:

```javascript
var hello;
console.log(hello); // undefined.
hello = 'Xin chao';
```

Chính nhờ vào _hoisting_ mà khi viết code với javascript, ta không cần quan tâm tới thứ tự các function. Ví dụ function A cần function B thực hiện một công việc thì trong các ngôn ngữ khác (C, C++,....) ta cần khai báo B trước khi khai báo A. Còn trong javascript ta có thể hoàn toàn khai báo A trước rồi mới tới B.

Và trong _hoisting_, **var** được khai báo và khởi tạo với một giá trị _undefined_.
Giống với **var** thì **let** cũng _hoisting_, được đem lên trên đầu của scope nhưng sẽ không được khởi tạo. Chính vì điều này mà khi ta truy cập tới biến được khai báo bằng _let_ trước khi khởi tạo, chúng ta sẽ gặp lỗi _Reference Error._

## Tóm lại

1. Đừng xài **var** khi không thể quản lý được việc biến bị override.
2. Đừng xài **var** sau khi đọc bài viết này.
3. Thay vì dùng **var**, tăng dùng **let** và **const** sẽ khiến cho cuộc sống của chúng ta tươi đẹp hơn.
4. Share bài viết này rộng rãi để mọi người có thể biết được sự nguy hiểm của **var** và cách mà ES6 hủy hoại nó :)

Cảm ơn các bạn đã đọc tới đây, nếu có thắc mắc gì hãy comment bên dưới nhé :D

[wiki]: https://en.wikipedia.org/wiki/ECMAScript
