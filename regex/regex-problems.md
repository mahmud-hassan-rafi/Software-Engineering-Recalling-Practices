## Problem: Bangladeshi Phone Number Matching

### code
```js
const regex = /^(\+?880[- ]?|0)1[3-9]\d{8}$/

console.log(regex.test("+8801870954580")) // true
console.log(regex.test("+88001870954580")) // false
console.log(regex.test("880 1870954580")) // true
console.log(regex.test("+88 01870954580")) // false
console.log(regex.test("880-1870954580")) // true
console.log(regex.test("1870954580")) // false
```

## Problem: username validation

### code
```js
const regex = /^[a-z]\w{3,11}$/i;

console.log(regex.test("Rafi123"))
console.log(regex.test("rafi_01"))
console.log(regex.test("1rafi"))
console.log(regex.test("_rafi"))
console.log(regex.test("raf"))
console.log(regex.test("rafi"))
```