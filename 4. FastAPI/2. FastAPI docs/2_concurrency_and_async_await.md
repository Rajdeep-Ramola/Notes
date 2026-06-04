# Concurrency and Async/Await

- You can only use `await` inside of functions created with `async def`

- **Asynchronous code** — a way to tell the computer that at some point, it will have to wait for something (e.g. a task) to finish, and during that time the computer can go do some other work (e.g. handling a request from another client).

  The computer will come back every time it has a chance to see if the task it was waiting for has been completed. If the task has been completed, the computer continues with other tasks or tasks associated with the completed task.

  The tasks which the computer has to wait for are generally **I/O operations** that are relatively slow, for example:

  - the data from the client to be sent through the network
  - the data sent by your program to be received by the client through the network
  - the contents of a file in the disk to be read by the system and given to your program
  - the contents your program gave to the system to be written to disk
  - a remote API operation
  - a database operation to finish
  - a database query to return the results

  This asynchronous code can also be called **concurrency**.

- **Common CPU-bound operation examples:**

  - **Audio** or **image processing**
  - **Computer vision** — an image is composed of millions of pixels, each pixel has 3 values/colors, processing that normally requires computing something on those pixels, all at the same time
  - **Machine Learning** — it normally requires lots of "matrix" and "vector" multiplications
  - **Deep Learning**
