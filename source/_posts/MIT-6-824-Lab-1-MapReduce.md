---
title: 'MIT 6.824 Lab 1: MapReduce'
date: 2018-08-18 01:32:59
tags: MapReduce
intro : MapReduce learning notes
comments: false
---

# MapReduce

MapReduce 是一个编程模型，也是一个处理和生成超大数据集的算法模型的相关实现。用户首先创建一 个 Map 函数处理一个基于 key/value pair 的数据集合，输出中间的基于 key/value pair 的数据集合;然后再创建 一个 Reduce 函数用来合并所有的具有相同中间 key 值的中间 value 值。

文中代码，以课程中的WordCount为例
# doMap
在WordCount的例子中```mapF()```实现了将文本转换为KV格式的数组。
```go

func mapF(filename string, contents string) []mapreduce.KeyValue {
	// Your code here (Part II).

	f := func(c rune) bool {
		return !unicode.IsLetter(c) && !unicode.IsNumber(c)
    }
    //按照非字母非数字的分割，将元数据才成单词
	words := strings.FieldsFunc(contents, f)

	var kvs []mapreduce.KeyValue

	for _, v := range words {
		kv := mapreduce.KeyValue{Key: v, Value: "1"}
		kvs = append(kvs, kv)
	}
	return kvs

}

func doMap(
	jobName string, // the name of the MapReduce job
	mapTask int, // which map task this is
	inFile string,
	nReduce int, // the number of reduce task that will be run ("R" in the paper)
	mapF func(filename string, contents string) []KeyValue,
) {
    //读取源文件
    data, err := ioutil.ReadFile(inFile)
	if err != nil {
		panic(err)
    }
    //将元数据转换成程序需要的KV格式
	keyValues := mapF(inFile, string(data))

	for i := 0; i < nReduce; i++ {
        //生成文件名
		rn := reduceName(jobName, mapTask, i)
		fo, err := os.Create(rn)
		defer fo.Close()

		if err != nil {
			panic(err)
		}
		enc := json.NewEncoder(fo)
		for _, kv := range keyValues {
            //以hash将大文件中的数据，平均拆分成nReduce个小文件中
			if ihash(kv.Key)%nReduce == i {
				err := enc.Encode(&kv)
				if err != nil {
					panic(err)
				}
			}
		}
	}
}

func ihash(s string) int {
	h := fnv.New32a()
	h.Write([]byte(s))
	return int(h.Sum32() & 0x7fffffff)
}

```

# doReduce
在WordCount的例子中```reduceF()```实现了将KV数据统计个数。
```go
func reduceF(key string, values []string) string {
	// Your code here (Part II).
	return strconv.Itoa(len(values))
}

func doReduce(
	jobName string, // the name of the whole MapReduce job
	reduceTask int, // which reduce task this is
	outFile string, // write the output here
	nMap int, // the number of map tasks that were run ("M" in the paper)
	reduceF func(key string, values []string) string,
) {
    kvs := make(map[string][]string)

	for i := 0; i < nMap; i++ {
		fn := reduceName(jobName, i, reduceTask)

		fin, err := os.Open(fn)
		defer fin.Close()

		if err != nil {
			panic(err)
		}

		dec := json.NewDecoder(fin)

		//decode data to kvs
		for {
			var kv KeyValue
			err := dec.Decode(&kv)
			if err == io.EOF {
				break
			}

			_, ok := kvs[kv.Key]
			if !ok {
				kvs[kv.Key] = make([]string, 0)
			}

			kvs[kv.Key] = append(kvs[kv.Key], kv.Value)
		}

	}

	var keys []string
	for k := range kvs {
		keys = append(keys, k)
	}
	sort.Strings(keys)

	mn := mergeName(jobName, reduceTask)
	fout, err := os.Create(mn)
	defer fout.Close()

	if err != nil {
		panic(err)
	}
	enc := json.NewEncoder(fout)

	for i := 0; i < len(keys); i++ {
		err = enc.Encode(&KeyValue{keys[i], reduceF(keys[i], kvs[keys[i]])})
		if err != nil {
			panic(err)
		}
	}
}
```


