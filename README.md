[![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/big-data-europe/Lobby)

# MapReduce 프로그래밍



## Hadoop Docker 다운 받기

```
git clone https://github.com/CUKykkim/hadoop-docker
```

```
remote: Enumerating objects: 36, done.
remote: Counting objects: 100% (36/36), done.
remote: Compressing objects: 100% (32/32), done.
Receiving objects: 100% (36/36), 37.39 KiB | 4.67 MiB/s, done.0eceiving objects:   2% (1/36)

Resolving deltas: 100% (3/3), done.
```

코드를 다운 받으면 hadoop-docker 라는 폴더가 생성된다. 
그 폴더 안으로 들어간다. 

```
cd hadoop-docker
```

## Mareduce 프로그래밍
-hadoop-docker 폴더 안에서 다음 명령어를 수행한다.

```
docker compose up
```

- 명령어를 수행하고 나면, 맵리듀스 프로그래밍 준비가 끝난 하둡이 컨테이너가 수행된다. 

```
docker ps
```

```
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                    PORTS                                                                                  NAMES
518b4fb6b372   d52e3b1d4d9c   "/entrypoint.sh /run…"   35 seconds ago   Up 31 seconds (healthy)   8088/tcp                                                                               resourcemanager
3253ec96e678   76ed22345d0c   "/entrypoint.sh /run…"   35 seconds ago   Up 31 seconds (healthy)   9864/tcp                                                                               datanode
ccd1621f9a2a   f431afd1feaf   "/entrypoint.sh /run…"   35 seconds ago   Up 30 seconds (healthy)   0.0.0.0:9000->9000/tcp, :::9000->9000/tcp, 0.0.0.0:9870->9870/tcp, :::9870->9870/tcp   namenode
27ab24004340   a0c8f6582e40   "/entrypoint.sh /run…"   35 seconds ago   Up 31 seconds (healthy)   8188/tcp                                                                               historyserver
f065ff0eac53   e448bd831a63   "/entrypoint.sh /run…"   35 seconds ago   Up 31 seconds (healthy)   8042/tcp                                                                               nodemanager
```


## 하둡 네임노드 컨테이너로 진입

```
docker exec -it namenode /bin/bash
```

## input.txt를 HDFS로 변환

먼저 HDFS의 폴더를 만들어준다. 
```
hdfs dfs -mkdir -p /user/hduser hdfs dfs -ls /user
```

로컬 파일 시스템 상에서 입력 데이터가 있는 곳으로 이동한다.
```
cd /hadoop-data/HadoopWithPython/python/MapReduce/HadoopStreaming
```

만들어준 HDFS 디렉토리 상에 input.txt를 올린다.

```
hdfs dfs -put input.txt /user/hduser
```


## 맵퍼 작성

mapper를 다음과 같이 작성한다.

- mapper.py

```
#!/root/anaconda3/bin/python
import sys
for line in sys.stdin:
  line=line.strip()
  keys=line.split()
  for key in keys:
    value=1
    print("{0}\t{1}".format(key,value))
```

## 리듀서 작성

- reducer.py

```
#!/root/anaconda3/bin/python
import sys
last_key=None
running_total=0

for input_line in sys.stdin:
  input_line=input_line.strip()
  this_key,value=input_line.split("\t",1)
  value=int(value)
    #기존에 존재하는 단어이면 카운트 증가 처리
  if last_key == this_key:
    running_total += value
  else: #새로운 단어이면
    if last_key:
      print("{0}\t{1}".format(last_key,running_total))
    running_total=value
    last_key=this_key

if last_key==this_key:
  print("{0}\t{1}".format(last_key, running_total))
```


## Hadoop 시스템에서 맵리듀스 작업 수행

다음 명령어를 입력하여 맵리듀스 작업을 수행한다.

```
hadoop jar /opt/hadoop-3.2.1/share/hadoop/tools/lib/hadoop-streaming-3.2.1.jar -input /user/hduser/input.txt -output /user/wordcount -mapper mapper.py -file mapper.py -reducer reducer.py -file reducer.py
```

위 명령어를 입력하면 Map과 Reduce 작업이 완료 되고
HDFS 상의 /user/wordcount 에 결과파일이 있는 것을 확인할 수 있다. 

```
hdfs dfs -ls /user/wordcount 
```

맵리듀스의 결과를 읽어들일수 있다. 

```
hdfs dfs -cat /user/worcount/part-00000
```

```
Cyber   2
Korea   1
The     2
University      1
of      1
```

