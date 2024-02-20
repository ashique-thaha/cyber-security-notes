`Race conditions` happen when two sections of codes are meant to be executed in a sequence but get out of sequence or order.

##### Concurrency
Concurrency is the ability to execute different parts of program simultaneously without affecting the outcome of the program. This will improve the program's performance as multiple parts of the program could be run at once.

concurrency has two types:` Multiprocessing `, `Multithreading`


##### Multiprocessing

Multiprocessing in short is using multiple CPUs, (the hardware that executes instructions to perform simultaneous computation).


##### Multithreading

Multithreading is the ability of a single CPU to provide multiple threads( concurrent executions).
But threads don't execute simultaneously, they take turns as threads become available. example: If one thread is suspended while waiting for user input, another can take over the CPU to execute its computations.


Arranging the sequence of execution of multiple threads is called `scheduling`. Different systems use different scheduling algorithms, some schedule tasks based on priority while another system might execute its tasks by giving out computational time in turns, regardless of priority.

This flexible scheduling is precisely what causes` race conditions`.


As the scheduling algorithm can swap between threads at any time, we can't really predict the sequence of threads executing each actions.


##### Example

 Two concurrent threads are trying to increase the value of a global variable by one, ie if the value of global variable is 0 the answer should be 2(because two threads worked here).


`Normal Execution of Two Threads Operating on the Same Variable`

![[Screenshot 2024-01-21 at 11.00.46 AM.png]]


But if the two threads are run simultaneously, without any consideration of conflicts that may occur, it will be like this:

`Incorrect Calculation Due to a Race Condition`

![[Screenshot 2024-01-21 at 11.02.30 AM.png]]

Here the value of variable becomes 1 where it should have been 2.

In summary, race conditions happen when the outcome of the execution of one thread depends on the outcome of another thread, and when two threads operate on the same resources without considering that other threads are also using those resources.

##### Race Condition Becomes a Vulnerability

This becomes a vulnerability when attackers could complete a sensitive action before security check is completed.

###### scenario:

Imagine instead of the global variables in previous example, threads are working on a bank money transfer.

Here bank has to perform three sub-tasks, to transfer money correctly.

1.  Check the originating account has high enough balance.
2.  If so, add money to destination account
3.  Deduct the transferred amount from the originated account.

ie,

There are two bank accounts, `Account A` and  `Account B`

Balance in Account A : 500
Balance in Account B : 0

We  are initiating two money transfers from Account A to Account B at the same time.


`In normal cases the money transfer should be like this`  

![[Screenshot 2024-01-21 at 11.24.39 AM.png]]

But when you can do the amount transfers simultaneously, you might be able to induce a race condition.

`Faulty Transfer Results Due to a Race Condition`

![[Screenshot 2024-01-21 at 11.28.43 AM.png]]

The same methodology can be used for authentication bypass, to rig online votes etc.


### Prevention

The key to avoiding race conditions is to protect the resources using a method called `synchronisation` or implementing mechanisms that ensure threads using the same resources won't execute simultaneously.

###### Resource locks

This is one of the mechanisms mentioned above. They block other threads from operating on the same resource by locking a resource.

In the bank transfer example, thread 1 could lock the balance of accounts A and B before modifying them, so that thread 2 would have to wait for it to finish before accessing the resources.

###### least privilege 

Beyond synchronisation, following secure coding practices, like the principle of least privilege, can prevent race conditions from turning into more severe security issues.

The principle of least privilege means that applications and processes should be granted only the privileges they need to complete their tasks.

For example, when an application requires only read access to a file, it should not be granted any write or execute permissions. You should 
grant applications precisely the permissions that they need instead. 

This lowers the risks of complete system compromise during an attack.



### Hunting for Race Conditions

###### Find Features Prone to Race Conditions

Most of the time, race conditions occur in features that deal with numbers, such as online voting, online gaming scores, bank transfers, e-commerce payments, and gift card balances. 

Look for these features in an application and take note of the request involved in updating these numbers.

For example, let’s say that, in your proxy, you’ve spotted the request used to transfer money from your banking site.

###### Send Simultaneous Requests

You can then test for and exploit race conditions in the target by sending multiple requests to the server simultaneously

For example, if you have 500 in your bank account and want to see if you can transfer more money than you have, 

You can simultaneously send multiple requests for transfer to the server via the curl command. If you’ve copied the command from Burp, you can simply paste the command into your terminal multiple times and insert a & character between each one. 

In the Linux terminal, the & character is used to execute multiple commands simultaneously in the background.

###### Check the Results

Check if your attack has succeeded. In our example, if your destination account ends up with more than 500 additions after the simultaneous requests, your attack has succeeded, and you can determine that a race condition exists on the transfer balance endpoint.

Note that whether your attack succeeds depends on the server’s process scheduling algorithm, which is a matter of luck. However, the more requests you send within a short time frame, the more likely your attack will succeed.

Also, many tests for race conditions won’t succeed the first time, so it’s a good idea to try a few more times before giving up.

###### Create a Proof of Concept

Once you have found a race condition, you will need to provide proof of the vulnerability in your report. The best way to do this is to lay out the steps needed to exploit the vulnerability. 

For example, you can lay out the exploitation steps like so:

1. Create an account with a 500 balance and another one with zero balance. The account with 500 will be the source account for our transfers and the one with zero balance will be the destination.

2. Execute this command:

```shell
curl (transfer 500) & curl (transfer 500) & curl (transfer 500) & curl (transfer 500) & curl (transfer 500) & curl (transfer 500)

```
This will attempt to transfer 500 to another account multiple times simultaneously.

3. You should see more than 500 in the destination account. Reverse the transfer and try the attack a few more times if you don’t see more than 500 in the destination account.


Since the success of a race condition attack depends on luck, make sure you include instructions to try again if the first test fails. If the vulnerability exists, the attack should succeed eventually after a few tries.