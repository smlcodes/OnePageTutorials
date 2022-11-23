### So here the Case.
1. I have list of 100 User Objects
2. I have FixedThreadPool of 10
3. I want to sumbit these 100 objects to 10 threads to Save in the Database & it returns same user Object with UserId, which is Stored in DB
4. I will collect each saved User object in Future.

### Expectation
I thought 10 Theards will exceute 100 User Objects parallally.

### Here the Problem
Also `Future.get()` is a blocking method, which means it will block the execution of current thread and next iteration of loop will not continue unless this future (on which the get is called) returns. This will happen in every iteration, which makes the code non parallel.
