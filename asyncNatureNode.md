## Understanding Async Nature of Nodejs

Async nature of nodejs is sometimes hard to understand. The key thing is **Nodejs doesn't wait until it has to.**  
Let me explain what does this mean,

1. **Nodejs doesn't wait**, when the program is executed nodejs will not wait for any non-blocking operation to complete. Non-blocking operations are I/O operations, file reading, os operation, user input from console, http request, etc. Now the question arises if nodejs doesn't wait for these operation then how theses operation are executed?  
   Here comes **Event** and **Callback**, which are core of nodejs architecture and this is what differentiate it from other programming languages.  
   So what happen is, when we start the nodejs program whole program is executed at one go. Event listener are set, like listening http request on a specified port or listening for user Input. Note, if there are no listener program will exit. Now the **Event Loop** will come into picture, it acts are bridge between events and callback. Event loop will execute the callback associated with the event, whenever event is fired. So single thread is there listening for all the events, and executing their callback. The event can use other thread for their purpose but the callback associated with them will be executed in the main program.

2. _until it has to_, this refer to the CPU intensive operations. If we do any CPU intensive calculation inside any callback, process will block and the performance will suffer, as the other callback (without blocking code) have to wait for the blocking callback to execute. That's why utmost care need to take to not block the event loop.

## Example: C++ vs nodejs

Here I have taken the same example to show how nodejs and other programming language are different.

### C++ example

```C++
#include <iostream>

using namespace std;

int main(int argc, char const *argv[])
{
    int num1{101};

        while (num1 <= 100)
    {
        cout << "Enter a integer greater than 100: " << endl;
        cin >> num1;
    }
    cout << "Bingo! You have entered correct value: " << endl;
    cout << "BYE BYE !!!" << endl;


    return 0;
}

```

The program is asking for user input and only terminates when user input is greater than 100. Program will wait for the user to enter number greater than 100 the it proceed with further execution.

### Nodejs example

Now the same example in nodejs

```js
const readline = require("readline");
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

rl.on("line", function (input) {
  if (input > 100) {
    console.log("Bingo! You have entered a correct value");
    rl.close();
  } else {
    console.log("Enter a greater than 100 ?");
  }
});

setInterval(() => {
  console.log("\nPardon me for interfering");
}, 3000);

rl.on("close", function () {
  console.log("\nBYE BYE !!!");
  process.exit(0);
});
console.log("Program executed");

console.log("Enter a greater than 100 ?");
```

Nodejs version of the same looks intimidating at first. `process.stdin` and `process.stdout` are use to interact with the console from the nodejs.

> readline.createInterface() is used for creating an instance of readline by configuring the readable and the writable streams. The input key takes a readable stream like process.stdin or fs.createReadStream('file.txt') and the output key takes a writable stream like process.stdout or process.stderr.
> `rl` is the instance of the readline.createInterface() on which are are listening for the "line" event, which is emitted whenever the input stream receives an end-of-line input. It is also listening for "close" event, which executes the callback and terminates the program.

The first difference we see is the position of prompt "Enter a greater than 100 ?" is at the end, as the execution is not linear. Event listener are set up and program will reach at the end and will not terminates as there are pending listening event.  
Also an important thing to see here is program is not waiting for the user to input instead it is reacting to the user input event with the specified callback. Therefore as the program listen for the user input we will see the callback in the setInterval also firing every 3 seconds with message "Pardon me for interfering".

## Conclusion

Hope I am able increase you understanding how async nodejs program works in comparison to the traditional way. Thank you for reading.
