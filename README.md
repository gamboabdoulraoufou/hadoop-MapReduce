## hadoop-MapReduce

In this post, I show how to use **Map/Reduce** Module with streaming API (using Python code) to process data. 
I will show 2 examples:
- Word count
- Join data

### 1- Word count example
**1-1- Create mapper and reducer files**  
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

**1-2- Create input data** 
``` sh
echo "A long time ago in a galaxy far far away" > /home/hadoop/testfile1
echo "Another episode of Star Wars" > /home/hadoop/testfile2
```

**1-3- Create a directory on the HDFS file system**
``` sh
hdfs dfs -mkdir /user/hadoop/input
``` 

**1-4- Copy the files from local filesystem to the HDFS**
copy files
``` sh
hdfs dfs -put /home/hadoop/testfile1 /user/hadoop/input
hdfs dfs -put /home/hadoop/testfile2 /user/hadoop/input
```

check
``` sh
hdfs dfs -ls /user/hadoop/input
```

**1-5- Run the Hadoop WordCount example with the input and output specified**

``` sh
# test the mapper and reducer python file
echo "foo foo quux labs foo bar quux" | python wordcount_mapper.py | sort -k1,1 | python wordcount_reducer.py

hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
   -input /user/hadoop/input \
   -output /user/hadoop/output_new \
   -mapper /home/hadoop/wordcount_mapper.py \
   -reducer /home/hadoop/wordcount_reducer.py
```
``` sh
hadoop jar /home/hadoop/hadoop-install/contrib/streaming/hadoop-streaming-1.2.1.jar \
   -input /user/test/input \
   -output /user/test/output_new16 \
   -file wordcount_mapper.py -mapper 'python wordcount_mapper.py' \
   -file wordcount_reducer.py -reducer 'python wordcount_reducer.py'
   
# merge results
hadoop fs -cat /user/test/output_new16/part-* > output
cat outpu

``` 

**1-6- Check the output file to see the results**
``` sh
# show list file
hdfs dfs -ls /user/hadoop/output_new

# show file content
hdfs dfs -cat /user/hadoop/output_new/part-00000
```

### 2- Join example
**2-1- Create mapper and reducer files**  
Create map file called join_mapper.py
Add the code billow

```python
#!/usr/bin/env python   

import sys

for line in sys.stdin:
    line       = line.strip() 
    key_value  = line.split(",")  
    key_in     = key_value[0].split(" ")   
    value_in   = key_value[1]  

    #print key_in
    if len(key_in)>=2:          
        date = key_in[0]      
        word = key_in[1]
        value_out = date+" "+value_in    
        print( '%s\t%s' % (word, value_out) ) 
    else:   
        print( '%s\t%s' % (key_in[0], value_in) )  
```

Create reduce file called join_reducer.py
Add the code billow

```python
#!/usr/bin/env python
import sys

prev_word          = "  "  
months             = ['Jan','Feb','Mar','Apr','Jun','Jul','Aug','Sep','Nov','Dec']
dates_to_output    = [] 
day_cnts_to_output = [] 
line_cnt           = 0  

for line in sys.stdin:
    line       = line.strip()       
    key_value  = line.split('\t')  
    line_cnt   = line_cnt+1     

    curr_word  = key_value[0]         
    value_in   = key_value[1]       

    if curr_word != prev_word:
        if line_cnt>1:
	    for i in range(len(dates_to_output)):  
	         print('{0} {1} {2} {3}'.format(dates_to_output[i],prev_word,day_cnts_to_output[i],curr_word_total_cnt))
	    dates_to_output = []
            day_cnts_to_output = []
        prev_word = curr_word  #set up previous word for the next set of input lines

    if (value_in[0:3] in months): 
        date_day =value_in.split() 
        dates_to_output.append(date_day[0])
        day_cnts_to_output.append(date_day[1])
    else:
        curr_word_total_cnt = value_in  
                                           
for i in range(len(dates_to_output)):  
         print('{0} {1} {2} {3}'.format(dates_to_output[i],prev_word,day_cnts_to_output[i],curr_word_total_cnt))
```   

Enter the following to make it executable
``` sh
chmod +x join_mapper.py
chmod +x join_reducer.py
```

**2-2- Create input data** 
``` sh
echo "able,991" > /home/hadoop/Joinfile1.txt
echo "about,11" >> /home/hadoop/Joinfile1.txt
echo "burger,15" >> /home/hadoop/Joinfile1.txt
echo "actor,22" >> /home/hadoop/Joinfile1.txt

echo "Jan-01 able,5" > /home/hadoop/joinfile2 
echo "Feb-02 about,3" >> /home/hadoop/joinfile2 
echo "Mar-03 about,8" >> /home/hadoop/joinfile2 
echo "Apr-04 able,13" >> /home/hadoop/joinfile2 
echo "Feb-22 actor,3" >> /home/hadoop/joinfile2 
echo "Feb-23 burger,5" >> /home/hadoop/joinfile2 
echo "Mar-08 burger,2" >> /home/hadoop/joinfile2 
echo "Dec-15 able,100" >> /home/hadoop/joinfile2 
```

**2-3- Create a directory on the HDFS file system**
``` sh
hdfs dfs -mkdir /user/hadoop/input_join
hdfs dfs -mkdir /user/hadoop/output_join
``` 

**2-4- Copy the files from local filesystem to the HDFS**
copy files
``` sh
hdfs dfs -put /home/hadoop/joinfile1 /user/hadoop/input_join
hdfs dfs -put /home/hadoop/joinfile2 /user/hadoop/input_join
```

check
``` sh
hdfs dfs -ls /user/hadoop/input_join
```

**2-5- Run the Hadoop Join example with the input and output specified**
``` sh
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
   -input /user/hadoop/input_join \
   -output /user/hadoop/output_join \
   -mapper /home/hadoop/join_mapper.py \
   -reducer /home/hadoop/join_reducer.py
```

**2-6- Check the output file to see the results**
``` sh
# show list file
hdfs dfs -ls /user/hadoop/output_join

# show file content
hdfs dfs -cat /user/hadoop/output_join/part-00000
```
