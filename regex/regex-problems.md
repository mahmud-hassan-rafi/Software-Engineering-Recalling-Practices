## Problem: Transaction Log Parser

### code
```js
const logs = `
[2026-10-17 10:30:15] SUCCESS | user:rafi | email:rafi@gmail.com | amount:$500 | tx:TX-1024
[2026-10-17 10:32:45] FAILED  | user:sakib | email:sakib@yahoo.com | amount:$700 | tx:TX-1025
[2026-10-17 11:05:20] PENDING | user:nabil | email:nabil@gmail.com | amount:$1200 | tx:TX-1026
[2026-10-17 11:15:00] SUCCESS | user:hasan | email:hasan@outlook.com | amount:$300 | tx:TX-1027
[2026-10-17 11:20:10] SUCCESS | user:rafi | email:rafi@gmail.com | amount:$950 | tx:TX-1028
[2026-10-17 11:25:30] CANCELLED | user:karim | email:karim@gmail.com | amount:$400 | tx:TX-1029
`;

const regex = /\[(\S+) (\S+)\] (SUCCESS|PENDING) \| user:(\w+) \| email:([a-z](?:\w+)\@(?:[a-z]+)\.(?:[a-z]+)) \| amount:\$(\d+) \| tx:(TX-(?:\d+){4})/gm;

const matchedResult = [...logs.matchAll(regex)].map(([,date, time, status, username, email, amount, trxId]) => (
    {
        status,
        date,
        time,
        username,
        email,
        amount,
        transactionId: trxId
    }
))

console.log(matchedResult)
```
### output
```js
    [
  {
    status: 'SUCCESS',
    date: '2026-10-17',
    time: '10:30:15',
    username: 'rafi',
    email: 'rafi@gmail.com',
    amount: '500',
    transactionId: 'TX-1024'
  },
  {
    status: 'PENDING',
    date: '2026-10-17',
    time: '11:05:20',
    username: 'nabil',
    email: 'nabil@gmail.com',
    amount: '1200',
    transactionId: 'TX-1026'
  },
  {
    status: 'SUCCESS',
    date: '2026-10-17',
    time: '11:15:00',
    username: 'hasan',
    email: 'hasan@outlook.com',
    amount: '300',
    transactionId: 'TX-1027'
  },
  {
    status: 'SUCCESS',
    date: '2026-10-17',
    time: '11:20:10',
    username: 'rafi',
    email: 'rafi@gmail.com',
    amount: '950',
    transactionId: 'TX-1028'
  }
]
```


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