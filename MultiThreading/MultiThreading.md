# MultiThreading in Python

## This documentation is about Multithreading implementation in Python in detail along with an example

MultiThreading is executing more than 1 thread of a process concurrently without any inconsistencies/anomalies that reduces the exection time and memory required for the execution.
The overall execution time is improved significantly and depends on no of threads inititiated. 

Consider a scenario where the requirement is to load huge volume of data, let's say in TB and conventional method like CREATE TABLE/INSERT INTO SELECT will take lot of time 
in processing/load but in a real world scenario, we are not able to give so much time and effort. Using multithreading in this situation will not only 
reduce the total execution time but will also take less memory while the program is running which is less prone to errors like memory issue etc.

Python provides threading and queue module to achieve multithreading. 

threading -  it provides a very simple and intuitive API for spawning multiple threads in a program.

queue - It is especially useful in threaded programming when information must be exchanged safely between multiple threads.

Using both these modules , we are going to implement the scenario mentioned above.

Use case - Our usecase is to load a historical data from a table of around 40 GB with 5 years of data with a simple transformation applied.

| Type        | Description  |
| ----------- | -----------  |
| Year        | 2003 to 2007 |
| Table Name  | TEST         |
| New Table   |TEST_HISTORY  |
| Approx data | 40 GB        |

Query - 

```sql
SELECT t.*, RANK() OVER (PARTITION BY TEST_DATE ORDER BY TEST_NAME) AS RN
        FROM TEST
        WHERE TEST_DATE BETWEEN '01-JAN-2003' AND '31-DEC-2007'
```

There are two options to load the data into history Table -

```sql 
   INSERT INTO TEST_HISTORY
   SELECT t.*, RANK() OVER (PARTITION BY TEST_DATE ORDER BY TEST_NAME) AS RN
   FROM TEST
   WHERE TEST_DATE BETWEEN '01-JAN-2003' AND '31-DEC-2007';
   
   CREATE TABLE TEST_HISTORY AS 
   SELECT t.*, RANK() OVER (PARTITION BY TEST_DATE ORDER BY TEST_NAME) AS RN
   FROM TEST
   WHERE TEST_DATE BETWEEN '01-JAN-2003' AND '31-DEC-2007';
```   

Conventional Approach :- Using the two conventional approaches as mentioned above took approx 40 mins to execute which for sample is not bad 
but consider if the data volume is pretty higher than we cannot afford to use the above approaches.

Multithreading Approach :- Using this approach with few threads the execution time will reduce to approximately half of time i.e. 20 mins.

Now, what we have to do is create a Python script importing threading & queue modules and module to make a connection to the database. 
Here, I will be using cx_Oracle for Oracle database.

And we will divide the data based on TEST_DATE for each year and execute the query using three threads. No of threads can be increased/decreased based on 
your requirement but please do note that the choice to the number of threads should be made in a way so that we achieve the optimum performance. 
More/less number of threads may impact the overall performance.

So, reference code will be -

```python

from threading import Thread
from queue import Queue

## Main Function

p_start_date1 = '01-JAN-2003'
p_end_date1   = '31-DEC-2003'
p_start_date2 = '01-JAN-2004'
p_end_date2   = '31-DEC-2004'
p_start_date3 = '01-JAN-2005'
p_end_date3   = '31-DEC-2005'
p_start_date4 = '01-JAN-2006'
p_end_date4   = '31-DEC-2006'
p_start_date5 = '01-JAN-2007'
p_end_date5   = '31-DEC-2007'
result_list = [[p_start_date1, p_end_date1], [p_start_date2, p_end_date2],
               [p_start_date3, p_end_date3], [p_start_date4, p_end_date4],
               [p_start_date5, p_end_date5]]
queue = Queue()
for elem in result_list:
    queue.put(elem)
thread_list = []    
for i in range(3):
    thread = Consumer(queue)
    thread_list.append(thread)
    thread_list[i].start()
for thread in thread_list:
    thread.join()

## Consumer Class

class Consumer(Thread) :
    def __init__(self, queue):
        Thread.__init__(self)
        self.queue = queue
    
    def run(self):
        while not self.queue.empty():
            elem = self.queue.get()
            p_start_date = str(elem[0])
            p_end_date   = str(elem[1])
            print('Start time:' + str(datetime.datetime.now()) + ' ' + p_start_date + ' ' + p_end_date)
            qry = """ CREATE TABLE TEST_HISTORY_""" + str(p_start_date[-4:0]) + """ AS 
                      SELECT t.*, RANK() OVER (PARTITION BY TEST_DATE ORDER BY TEST_NAME) AS RN
                      FROM TEST
                      WHERE TEST_DATE BETWEEN TO_DATE('{p_start_date}', 'DD-MON-YYYY') AND TO_DATE('{p_end_date}', 'DD-MON-YYYY') 
                      """.format(p_start_date=p_start_date, p_end_date=p_end_date)
            conn = cx.connect(username, pwd, host) 
            cursor = conn.cursor()
            cursor.execute(qry)
            print('Start time:' + str(datetime.datetime.now()) + ' ' + p_start_date + ' ' + p_end_date)
            conn.close()
            self.queue.task_done()
```

Note the execution of threads and performance of the overall process.

Happy Coding!!

