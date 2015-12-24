## hadoop-MapReduce

In this post, I show how to use **Map/Reduce** Module with streaming API (using Python code) to process data. In this case I make word count.

**1- Create mapper and reducer files**  
Create map file called wordcount_mapper.py
Add the code billow

```python
#!/usr/bin/env python   

import sys

for line in sys.stdin:
    line = line.strip()
    keys = line.split() 
    for key in keys:
        value = 1        
        print('{0}\t{1}'.format(key, value))
```

Create reduce file called wordcount_reducer.py
Add the code billow

```python
#!/usr/bin/env python

import sys

last_key      = None            
running_total = 0

for input_line in sys.stdin:
    input_line = input_line.strip()
    this_key, value = input_line.split("\t", 1)
    value = int(value)
 
    if last_key == this_key:
        running_total += value
    else:
        if last_key:
            print( "{0}\t{1}".format(last_key, running_total))
        running_total = value
        last_key = this_key

if last_key == this_key:
    print( "{0}\t{1}".format(last_key, running_total))
```   

Enter the following to make it executable
``` sh
chmod +x wordcount_mapper.py
chmod +x wordcount_reducer.py
```

**2- Create input data** 
``` sh
echo "A long time ago in a galaxy far far away" > /home/hadoop/testfile1
echo "Another episode of Star Wars" > /home/hadoop/testfile2
```

**3- Create a directory on the HDFS file system**
``` sh
hdfs dfs -mkdir /user/hadoop/input
``` 

**4- Copy the files from local filesystem to the HDFS**
copy files
``` sh
hdfs dfs -put /home/hadoop/testfile1 /user/hadoop/input
hdfs dfs -put /home/hadoop/testfile2 /user/hadoop/input
```

check
``` sh
hdfs dfs -ls /user/hadoop/input
```

**5- Run the Hadoop WordCount example with the input and output specified**
``` sh
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
   -input /user/hadoop/input \
   -output /user/hadoop/output_new \
   -mapper /home/hadoop/wordcount_mapper.py \
   -reducer /home/hadoop/wordcount_reducer.py
```

**6- Check the output file to see the results**
``` sh
# show list file
hdfs dfs -ls /user/cloudera/output_new

# show file content
hdfs dfs -cat /user/hadoop/output_new/part-00000
```
