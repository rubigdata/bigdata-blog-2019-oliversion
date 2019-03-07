# Big data.
## Blog post. Hadoop 2.9.2 and Map-Reduce

In this assignment, Hadoop was installed on own “pseudo-cluster”, and Map-Reduce was used to do some basic count operations.

First of all, the Hadoop Distributed File System (HDFS) was installed. Various shell-like commands '''hdfs dfs''' are used in irder to directly interact with HDFS.
[Complete Shakespeare] (https://www.gutenberg.org/cache/epub/100/pg100.txt) was downloaded from the alternative github website (as original link did not work) and save it to the HDFS. As this file is gz archive, text was extracted using 

'''
mv pg100.txt pg100.gz
gunzip pg100.gz
'''

Using nano text editor, WordCount.java was created using copy-paste the sourcecode from the [relevant Hadoop documentation] (https://hadoop.apache.org/docs/r2.9.2/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Source_Code).


Running '''bin/hadoop jar wc.jar WordCount input output''' we have got output of the word count procedure at a file *part-r-00000* in a folder *output*. 

Using '''bin/hdfs dfs -get output output''' output is copied to docker and to the local filesystem after that using '''docker cp 671888f8f041:/opt/docker/hadoop-2.9.2/output/output/part-r-00000 ~'''. The result of the WordCount can be inspected using '''cat output/output/part-r-00000'''. 


Mapreduce uses string tokenizer to split text in single words that does not take into account punctuation marks.
'''
public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }
'''
 As a result words 'Romeo', 'Romeo,' considered as different words. In total in different variations, Romeo appeared 156 times, whereas Juliet 82.
'''
Romeo 	2
Romeo, 	1
ROMEO	1
Romeo	47
Romeo!	15
Romeo's	18
Romeo's,	1
Romeo,	36
Romeo-	1
Romeo.	20
Romeo.]	1
Romeo;	3
Romeo?	10

JULIET,	2
JULIET.	7
JULIET]	1
Juliet	22
Juliet!	2
Juliet's	7
Juliet,	16
Juliet.	14
Juliet;	2
Juliet?	5
Julietta	1
Julietta's	1
'Juliet,'	1
'Juliet.'	1
'''
