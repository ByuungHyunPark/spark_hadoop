# __2장 :__ 스칼라와 스파크를 활용한 데이터 분석

---
__sc__ 라는 변수는 클러스터상에서 스파크 작업 실행을 관리하는 SparkContext 객체의 참조이다.
```
scal > sc
...
res2: org.apache.spark.SparkContext = org.apache.spark.SparkContext@648f48d3
```
REPL에 스칼라 객체를 출력하라고 하면 그 객체의 문자열 형식을 출력한다 . SparkContext객체의 경우, 그 값은 단순히 이 객체의 이름에 16진수로 된 메모리 주소를 붙인 문자열이다. <br>

- SparkContext는 스칼라 객체이며 여러 메서드를 가지고 있다. 스칼라 REPL에서 객체의 변수 이름에 점(.)을 찍고 탭(TAB)을 누르면 아래와 같이 객체가 제공하는 메서드 확인 가능

```
scala > sc.[\tab]
...
accumulable             defaultMinPartitions      isStopped               runApproximateJob
accumulableCollection   defaultParallelism        jars                    runJob
accumulator             deployMode                killExecutor            sequenceFile
addFile                 doubleAccumulator         killExecutors           setCallSite
addJar                  emptyRDD                  killTaskAttempt         setCheckpointDir
addSparkListener        files                     listFiles               setJobDescription
appName                 getAllPools               listJars                setJobGroup
applicationAttemptId    getCheckpointDir          longAccumulator         setLocalProperty
applicationId           getConf                   makeRDD                 setLogLevel
binaryFiles             getExecutorMemoryStatus   master                  sparkUser
binaryRecords           getLocalProperty          newAPIHadoopFile        startTime
broadcast               getPersistentRDDs         newAPIHadoopRDD         statusTracker
cancelAllJobs           getPoolForName            objectFile              stop
cancelJob               getRDDStorageInfo         parallelize             submitJob
cancelJobGroup          getSchedulingMode         range                   textFile
cancelStage             hadoopConfiguration       register                uiWebUrl
clearCallSite           hadoopFile                removeSparkListener     union
clearJobGroup           hadoopRDD                 requestExecutors        version
collectionAccumulator   isLocal                   requestTotalExecutors   wholeTextFiles
```

SparkContext 객체가 제공하는 많은 메서드중에서 우리는 __탄력적 분산 데이터셋(RDD)__ 을 생성하는 메서드를 가장 자주 사용하게 될 것이다. RDD는 스파크에서 사용하는 기본적인 추상개념으로, 클러스터의 여러 노드에 분산할 수 있는 객체들의 컬렉션을 의미한다. 
<br>스파크에서 RDD를 생성하는 방법은 두가지이다.
- 첫 번째는 SparkContext를 이용하여 외부 데이터로부터 RDD를 생성하는 방법이다. 사용할 수 있는 외부데이터에는 HDFS상의 파일, JDBC를 통한 데이터베이스 테이블, 스파크 셸에서 생성한 로컬 객체 컬렉션 등이 있다.
- 두번째는 이미 만들어진 하나 이상의 RDD를 레코드 필터링, 집계, 조인하는 등 가공하여 새로운 RDD를 생성하는 방법이다.


__RDD란 ?__
```
RDD는 클러스터의 여러 노드에 파티션으로 나뉘어 분산되며, 각 파티션은 RDD의 전체 데이터 중 일부를 담게 된다. 파티션은 스파크에서 병렬 처리되는 단위다.
스파크 프레임워크는 한 파티션 내의 객체들을 순차적으로 처리하고, 파티션끼리는 병렬로 처리한다. RDD를 생성하는 아주 간단한 방법은 로컬 객체 컬렉션을 인수로 하여 SparkContext의 parallelize 메서드를 실행하는 것이다.

    scala > val rdd = sc.parallelize(Array(1, 2, 2, 4), 4)
    ...
    rdd: org.apache.spark.rdd.RDD[Int] = ...

첫 번째 인수는 병렬 처리하려는 객체 컬렉션이며, 두번째 인수는 파티션의 개수다.
파티션 내의 객체들에 연산을 수행할 때가 되면 스파크는 구동자 프로세스로부터 객체 컬렉션의 일부를 가져온다.

RDD를 HDFS 같은 분산 파일시스템에 위치한 텍스트파일이나 텍스트 파일들을 포함한 디렉터리로부터 생성하려면 textFile 메서드에 해당 파일 또는 디렉터리의 이름을 인수로 주면 된다.

    scala > val rdd2 = sc.textFile("hdfs://some/path.txt")
    ...
    rdd2: org.apache.spark.rdd.RDD[String] = ...

로컬 모드에서는 textFile 메서드에 로컬 파일시스템의 경로를 인수로 주면 된다. textFile 메서드의 인수로 개별 파일 이름 대신 디텍터리 이름을 넣으면 스파크는 그 디렉터리의 모든 파일을 RDD의 구성 요소로 간주한다. 덧붙여서, 이 시점까지는 클라이언트나 클러스터 그 어디에서도 스파크가 데이터를 읽어 들이거나 메모리에 데이터를 올리는 일은 실제로 일어나지 않는다. 스파크는 파티션 내의 객체들에 연산을 수행할 때가 되어서야 입력 파일을 섹션(split이라고도 한다) 단위로 읽어 다른 RDD들에 정의한 필터링과 집계 같은 일련의 변환에 적용한다.
```
예제의 레코드 링크 데이터는 텍스트 파일 하나에 저장되어 있으며, 이 파일의 각 줄은 하나의 레코드를 의미한다. 이제 SparkContext의 textFile 메서드를 사용해 이 데이터로 RDD 객체를 생성
```
scala > val rawblocks = sc.textFile("linkage")
...
rawblocks: org.apache.spark.rdd.RDD[String] = ...
```
새로운 변수 rawblocks를 선언했는데, 변수의 자료형은 자동으로 RDD[String]으로 지정된것을 확인 간으. 스칼라 언어가 제공하는 자료형 추정 기능으로, 스칼라는 변수가 사용되는 문맥을 파악할 수 있다면 이를 통해 변수의 자료형이 무엇인지를 유추하여 개발자의 코드 작성을 도와준다. 여기서 SparkContext 객체의 textFile 메서드의 반환 자료형이 RDD[String]임을 확인하고 rawblocks 변수를 그 자료형으로 지정한 것이다.


스칼라 언어에서는 새로운 변수를 만들 때 항상 변수앞에 val 또는 var 을 선언해야 한다.
- val로 선언된 변수는 값이 한번 지정되면 변경 X
- var로 선언되면 다른값으로 변경 가능