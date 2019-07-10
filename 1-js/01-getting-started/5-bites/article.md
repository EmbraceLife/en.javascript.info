
# JS bites by 深度碎片

## Object to primitive conversion

### Problems By Code

  ```js run
  let obj = { };
  if (obj) { // it is a feature, not really a problem
    console.log("obj is true in boolean context");
  };
  ```
  ```js run
  let numObj = {
    value : 5, 
  };
  console.log(numObj.value); // 5
  console.log(numObj - 1); // numeric context => NaN
  console.log(Number(numObj)); // numeric context => NaN
  ```
  ```js run
  let strObj = {
    value : "what would happen?",
  }
  console.log(strObj); // object
  console.log(String(strObj)); // string context => [object Object]
  console.log(strObj + "yes?"); // string context => [object Object] yes?
  ```
  ```js run
  let num0 = {
    value0 : 5,
  };

  let num1 = {
    value1 : 3,
  };
  console.log(num0.value0 > num1.value1); // true
  console.log(num0 > num1); // numeric context => NaN > NaN => false
  console.log(NaN > NaN); // false
  ```

### Solution by Code
  ```js run
  let user1 = {
    name: "John",
    money: 1000,

    // how object should react to different contexts
    [Symbol.toPrimitive](hint) {
      console.log(`hint: ${hint}`);
      // divide into string and non string contexts
      // default context usually convert to numeric reaction too
      return hint == "string" ? this.name : this.money; 
    }
  };

  console.log(user1); // just object no hint
  console.log("hello" + user1 ); // hint: default (but treated as number)
  console.log(`hello ${user1}`); // hint: string
  console.log(+user1); // hint: number -> 1000
  console.log(user1 + 500); // hint: default -> 1500
  alert(user1) // hint: string
  ```
  ```js run
  let user1 = {
    name: "John",
    money: 1000,

    // string context reaction, before Symbol time
    toString() {
      return this.name;
    },

    // non-string: numeric or default context, before Symbol time
    valueOf() {
      return this.money;
    }

  };

  console.log(user1); // just object no hint
  console.log("hello" + user1 ); // hint: default (but treated as number)
  console.log(`hello ${user1}`); // hint: string
  console.log(+user1); // hint: number
  console.log(user1 + 500); // hint: default -> 1500
  alert(user1) // hint: string
  ```
  ```js run
  let user1 = {
    name: "John",
    money: 1000,

    [Symbol.toPrimitive](hint) {
      console.log(`hint: ${hint}`);
      // you can set reaction to default context the same to string context
      if (hint == "string" || "default") {
        return this.name;
      };
    },
  };

  console.log(user1); 
  console.log(user1 + 500); // default convert to string -> John500
  ```
