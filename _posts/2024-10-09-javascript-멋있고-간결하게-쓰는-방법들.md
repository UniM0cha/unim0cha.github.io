---
layout: post
title: Javascript 멋있고 간결하게 쓰는 방법들
date: 2024-10-09 13:13 +0900
categories: ["개발", "Javascript"]
tags: ["Javascript"]
---

## 3항 연산자

#### Bad Code

```javascript
function getResult(score) {
  let result;
  if (score > 5) {
    result = "good";
  } else if (score <= 5) {
    result = "bad";
  }
  return result;
}
```

#### Good Code

```javascript
function getResult(score) {
  return score > 5 ? "good" : "bad";
}
```

## Nullish coalescing operator (Null, undefined 구별하기)

#### Bad Code

```javascript
function printMessage(text) {
  let message = text;
  if (text == null || text == undefined) {
    message = "Nothing to display";
  }
  console.log(message);
}
```

#### Good Code

```javascript
function printMessage(text) {
  const message = text ?? "Nothing to display";
  console.log(message);
}
```

번외) 디폴드 값은 undefined 만 구별할 수 있다.

```javascript
function printMessage(text = "Nothing to display") {
  console.log(text);
}
```

## Object Destructuring (객체 나누기)

```javascript
const person = {
  name: "Julia",
  age: 20,
  phone: "01077777777"
};
```

#### Bad Code

```javascript
function displayPerson(person) {
  displayName(person.name);
  displayPhone(person.phone);
  displayProfile(person.name, person.age);
}
```

#### Good Code

```javascript
function displayPerson(person) {
  const { name, age, phone } = person;
  displayName(name);
  displayPhone(phone);
  displayProfile(name, age);
}
```

#### Better Code

```javascript
function displayPerson({ name, age, phone }) {
  displayName(name);
  displayPhone(phone);
  displayProfile(name, age);
}
```

## Spread Syntax (...)

### Object

두 객체를 합치기

```javascript
const item = { type: "shirt", size: "M" };
const detail = { price: 20, made: "Korea", gender: "M" };
```

#### Bad Code

```javascript
item["price"] = detail.price;

// 또는
const newObject = {
  type: item.type,
  size: item.size,
  price: detail.price,
  mad: detail.made,
  gender: detail.gender
};
```

#### Good Code

```javascript
const shirt0 = Object.assign(item, detail);
```

#### Better Code

```javascript
const shirt1 = { ...item, ...detail };
```

### Array

```javascript
let fruits = ["수박", "오렌지", "바나나"];

// fruits.push('딸기');
fruits = [...fruits, "딸기"];

// fruits.unshift('딸기');
fruits = ["딸기", ...fruits];

const fruits2 = ["멜론", "복숭아", "파인애플"];

// let combined = fruits.concat(fruits2);
let combined = [...fruits, ...fruits2];
```

## 체이닝

```javascript
const bob = {
  name: "Julia",
  age: 20
};
const anna = {
  name: "Julia",
  age: 20,
  job: {
    title: "SoftwareEngineer"
  }
};
```

#### Bad Code

```javascript
function displayJobTitle(person) {
  if (person.job && person.job.title) {
    console.log(person.job.title);
  }
}
```

#### Good Code

```javascript
function displayJobTitle(person) {
  if (person.job?.title) {
    console.log(person.job.title);
  }
}
```

#### Better Code

```javascript
function displayJobTitle(person) {
  const title = person.job?.title ?? "No Job Yet!";
  console.log(title);
}
```

## items API

```javascript
const items = [1, 2, 3, 4, 5, 6];
```

#### Bad Code

```javascript
const evens = getAllEvens(items);
const multiple = multyplyByFour(evens);
const sum = sumArray(multiple);
console.log(sum);

function getAllEvens(items) {
  const result = [];
  for (let i = 0; i < items.length; i++) {
    if (items[i] % 2 === 0) {
      result.push(items[i]);
    }
  }
  return result;
}

// +당신들이 생각하는 그 복잡한 코드들....
```

#### Good Code

```javascript
const evens = items.filter((num) => num % 2 === 0);
const multiple = evens.map((num) => num * 4);
const sum = multiple.reducs((a, b) => a + b, 0);
console.log(sum);
```

#### Better Code (체이닝 활용)

```javascript
const result = items
  .filter((num) => num % 2 === 0)
  .map((num) => num * 4)
  .reducs((a, b) => a + b, 0);
console.log(result);
```

## promise → async/await

#### Bad Code

```javascript
function displayUser() {
  fetchUser() //
    .then((user) => {
      fetchProfile(user) //
        .then((profile) => {
          updateUI(user, profile);
        });
    });
}
```

#### Good Code

```javascript
async function displayUser() {
  const user = await fetchUser();
  const profile = await fetchProfile(user);
  updateUI(user, profile);
}
```

## 중복 요소 제거하기

```javascript
const array = ["개", "고양이", "강아지", "말", "개", "고양이"];
const array2 = [...new Set(array)];
```
