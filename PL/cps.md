``` ts
function f(n: number) {
  if (n === 1) {
    return 1
  }
  return 1 + f(n - 1)
}

function g(n: number, res: number) {
  if (n === 1) {
    return res + 1
  }
  return g(n - 1, res + 1)
}

function h(n: number, cont: Function) {
  if (n === 1) {
    return cont(1)
  }
  return h(n - 1, (x: number) => cont(x + 1))
}

// trampoline
function hh(n: number) {
  let cont = (x: number) => x
  let num = n
  return c()

  function c() {
    if (num === 1) {
      return cont(1)
    }

    let oldCont = cont
    cont = (x: number) => oldCont(x + 1)
    num -= 1
    return c()
  }
}

// 更复杂的，斐波拉切数列
function fi(n: number) {
  if (n === 0 || n === 1) {
    return 1
  }
  return fi(n - 1) + fi(n - 2)
}

// 斐波拉切数列的cps形式
function hi(n: number, cont: Function) {
  if (n === 0 || n === 1) {
    return cont(1)
  }

  return hi(n - 1, (x: number) => hi(n - 2, (y: number) => cont(x + y)))
}

// 斐波拉切数列的cps + trampoline形式
function hhi(n: number) {
  let cont = (x: number) => x
  let num = n
  return c()

  function c() {
    if (num === 0 || num === 1) {
      return cont(1)
    }

    let oldCont = cont
    let oldNum = num
    cont = (x: number) => {
      cont = (y: number) => oldCont(x + y)
      num = oldNum - 2
      return c()
    }
    num = oldNum - 1
    return c()
  }
}

// 斐波拉切数列的阴阳形式
function hhhi(n: number) {
  return ying(n, x => x)

  function ying(i: number, cont: Function) {
    if (i === 0 || i === 1) {
      return cont(1)
    }

    return yang(i - 1, (x: number) => ying(i - 2, (y: number) => cont(x + y)))
  }

  function yang(i: number, cont: Function) {
    if (i === 0 || i === 1) {
      return cont(1)
    }

    return ying(i - 1, (x: number) => yang(i - 2, (y: number) => cont(x + y)))
  }
}

// Todo: callcc

console.log(f(33))
console.log(g(33, 0))
console.log(h(33, x => x))
console.log(hh(33))
console.log(fi(6))
console.log(hi(6, x => x))
console.log(hhi(6))
console.log(hhhi(6))
```