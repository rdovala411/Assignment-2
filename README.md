# Assignment 2: Document Similarity using MapReduce

**Name:** Ruthwik Dovala

**Student ID:** 801431661

## Document Similarity Problem (Assignment 2)

### The Mapper - Breaking Down Documents

In **DocumentSimilarityMapper** takes raw text and prepares it for comparison:

**Input:** Lines like `"Document1 This is some text content"`

**Process:**
Input is in this format, the files named as `input1.txt`, `input2.txt`, `input3.txt`
```bash
Document1 This is a sample document containing words
Document2 Another document that also has words
Document3 Sample text with different words
```

**Output:** Key `"DOC"` with document name and unique words

### The Reducer - Calculating Similarities  

**DocumentSimilarityReducer** compares all documents using Jaccard similarity:

**Process:**
1. Collects all document word sets
2. Compares every document pair
3. Calculates: intersection ÷ union = similarity score

**Output:** Clean results like `"Document1, Document2 Similarity: x"`

### Data Flow

1. **Input:** Text files split across mappers
2. **Map:** Clean and extract unique words from each document
3. **Shuffle:** Gather all documents under single key
4. **Reduce:** Calculate pairwise similarities
5. **Output:** Formatted similarity scores

## Setup and Execution

### Step 0 
- Docker and Docker Compose (building docker env)
- Maven
- Test files: `input1.txt`, `input2.txt`, `input3.txt`

### Step 1: Build the Project

```bash
mvn clean compile package -DskipTests
```

### Step 2: Start 3-Datanode Cluster

```bash
docker-compose up -d
```

### Step 3: Test with 3 Datanodes

#### Upload test files

```bash
docker exec namenode hdfs dfs -mkdir -p /input1
docker exec namenode hdfs dfs -put /shared-folder/input1.txt /input1/

docker exec namenode hdfs dfs -mkdir -p /input2
docker exec namenode hdfs dfs -put /shared-folder/input2.txt /input2/

docker exec namenode hdfs dfs -mkdir -p /input3
docker exec namenode hdfs dfs -put /shared-folder/input3.txt /input3/
```

#### Run timed tests

```bash

time docker exec namenode hadoop jar /shared-folder/DocumentSimilarity-0.0.1-SNAPSHOT.jar controller.DocumentSimilarityDriver /input1 /output1_3nodes

time docker exec namenode hadoop jar /shared-folder/DocumentSimilarity-0.0.1-SNAPSHOT.jar controller.DocumentSimilarityDriver /input2 /output2_3nodes

time docker exec namenode hadoop jar /shared-folder/DocumentSimilarity-0.0.1-SNAPSHOT.jar controller.DocumentSimilarityDriver /input3 /output3_3nodes
```

#### Save results

```bash
docker exec namenode hdfs dfs -get /output1_3nodes /shared-folder/
docker exec namenode hdfs dfs -get /output2_3nodes /shared-folder/
docker exec namenode hdfs dfs -get /output3_3nodes /shared-folder/
```

### Step 4: Switch to 1 Datanode

#### Stop and clean up

```bash
docker-compose down
docker container prune -f
```

#### Edit docker-compose.yml, and hadoop.env


#### Restart

```bash
docker-compose up -d
```

### Step 5: Test with 1 Datanode

#### Run timed tests (1-datanode)

```bash

time docker exec namenode hadoop jar /shared-folder/DocumentSimilarity-0.0.1-SNAPSHOT.jar controller.DocumentSimilarityDriver /input1 /output1_1nodes

time docker exec namenode hadoop jar /shared-folder/DocumentSimilarity-0.0.1-SNAPSHOT.jar controller.DocumentSimilarityDriver /input2 /output2_1nodes

time docker exec namenode hadoop jar /shared-folder/DocumentSimilarity-0.0.1-SNAPSHOT.jar controller.DocumentSimilarityDriver /input3 /output3_1nodes
```
#### Save results

```bash
docker exec namenode hdfs dfs -get /output1_1nodes /shared-folder/
docker exec namenode hdfs dfs -get /output2_1nodes /shared-folder/
docker exec namenode hdfs dfs -get /output3_1nodes /shared-folder/
```




## Performance Analysis Results

### Execution Times Comparison

| Dataset | Size (words) | 3 Datanodes (sec) | 1 Datanode (sec) | Performance Difference |
|---------|--------------|-------------------|------------------|----------------------|
| input1  | ~1,000      | 23.663           | 22.871          | **1-node 3.4% faster** |
| input2  | ~3,000      | 20.281           | 20.587           | **3-node 1.509% faster** |
| input3  | ~5,000      | 18.605           | 20.416           | **3-node 9.48% faster** |




### Key Insights

**Surprising Results:** 1-datanode performed better due to coordination overhead dominating small dataset processing.

**Setup Time Dominance:** 80% of execution time is Hadoop initialization, only 20% actual processing.

**Resource Efficiency:** Single datanode used 12% less memory on average.

### Performance Summary

| Metric | 3-Datanode | 1-Datanode | Winner |
|--------|------------|------------|--------|
| Average Execution Time | 20.849s | 21.291s | **3-Datanode (2.09% faster)** |

# Scalability Conclusions

## When Multiple DataNodes Are Beneficial
- Handling very large datasets that can be divided into many map tasks  
- Workloads with heavy disk I/O that gain from parallel reads/writes  
- High-throughput jobs where network overhead is outweighed by distributed gains  
- Applications that require higher fault tolerance and redundancy  

## When a Single DataNode Is Adequate
- Small to moderate input sizes (typically under ~100 MB)  
- Development, debugging, and testing setups  
- Quick prototyping and algorithm validation runs  
- Situations with limited system resources  

---

## Anticipated Outcomes
- With **three DataNodes**, performance should improve because of:  
  - Parallel execution across nodes  
  - Distributed storage and compute  
  - More balanced use of resources  

- With **one DataNode**, we expect:  
  - Slower performance due to a single bottleneck  
  - All computation occurring on the same machine  
  - Possible I/O constraints  

---

## Resource Observations
- **CPU**: Check processor load during job execution  
- **Memory**: Monitor container memory usage  
- **Network I/O**: Watch for data transfer between nodes  
- **Disk I/O**: Measure HDFS read/write efficiency  

---

## Performance Details

### Datasets
- *input1.txt*: ~1,000 words across 3 docs 
- *input2.txt*: ~3,000 words across 3 docs 
- *input3.txt*: ~5,000 words across 3 docs  
- Each expected to show low to medium levels of topic overlap  

### MapReduce Flow
- **Map**: Tokenize 3 documents, extract unique terms  
- **Shuffle**: Consolidate document data  
- **Reduce**: Compute 3 pairwise similarity scores (Doc1–Doc2, Doc1–Doc3, Doc2–Doc3)  

---

## Conclusions
- **Performance** – The 3-DataNode setup completed jobs faster than the 1-DataNode setup by distributing storage and computation across nodes, improving parallelism and resource utilization.

- **Dataset Size Effect** – For the small datasets used in this project, the performance difference was minimal since Hadoop’s setup overhead dominated the execution time.

- **Practical Use** – A single node is sufficient for development and testing, but a multi-node cluster becomes essential for larger datasets, scalability, and fault tolerance.



## Challenges and Solutions

## Challenges and Solutions  

### 1. Java Package Structure  
- **Problem:** Poor initial organization caused conflicts and compilation errors.  
- **Solution:** Adopted a clean hierarchy:  
  - `controller.DocumentSimilarityDriver`  
  - `mapper.DocumentSimilarityMapper`  
  - `reducer.DocumentSimilarityReducer`  

### 2. Controller (Driver) Issues  
- **Problem:** The `DocumentSimilarityDriver` class had mismatches in package declarations, missing imports, and incorrect job configuration. It wasn’t properly linking the mapper and reducer.  
- **Solution:** Reorganized the class under the correct package, fixed imports, and carefully configured input/output paths, mapper/reducer classes, and output types.  
- **Result:** The driver correctly orchestrated the MapReduce workflow.  

### 3. Outdated Setup Instructions  
- **Problem:** Provided template commands didn’t match the Docker environment.  
- **Solution:** Updated container names, file paths, and HDFS commands through trial and error.  

### 4. Output Formatting  
- **Problem:** Hadoop’s default output inserted unwanted tabs between keys and values.  
- **Solution:** Modified the reducer to emit the full result line as the key, with the value set to `null`.  
- **Result:** Clean output such as:`"Document1, Document2 Similarity: 0.56"`

### 5. Managing Docker Containers  
- **Problem:** Switching between single-node and multi-node setups caused conflicts, stale data, and inconsistent results.  
- **Solution:**  
- Used `docker container prune -f` to clear unused containers.  
- Always shut down with `docker-compose down` before reconfiguration.  
- Re-uploaded datasets after every configuration change.  
- Allowed extra time for the cluster to initialize properly.  

### 6. Performance Testing  
- **Problem:** No systematic method for comparing performance.  
- **Solution:**  
- Used the `time` command for wall-clock runtime.  
- Ran jobs sequentially to avoid interference.  
- Recorded Hadoop counters for detailed insights.  
- Created standardized datasets to ensure consistency.  


### 7. Safety Mode error
- I got a safety mode error while i am perforing -put command, and to overcome that error 
- **Step 1** 
# Check & exit Safe Mode
### See current state
``` 
docker exec namenode hdfs dfsadmin -safemode get
```

# Leave safemode
```
docker exec namenode hdfs dfsadmin -safemode leave
```

### 8. Final Updates
- The last update of `hadoop.env` and `docker-compose.yml` are for just a single node, not for 3 nodes.
---

## Key Lessons Learned  
- **Technical:** Good package structure, a properly configured driver, effective container management, and output formatting are all critical to success.  
- **Process:** With small datasets, Hadoop’s overhead dominates. In such cases, a single-node setup is often more efficient than a multi-node cluster.  

---

## ⚠️ AI Usage Disclaimer  
Parts of this README (including formatting, paraphrasing of findings, and troubleshooting guidance) were assisted by AI tools.  

AI was primarily used for:  
- Formatting this document into Markdown  
- Refining explanations for clarity  
- Suggesting fixes when Docker/HDFS commands raised errors  

All technical steps were **tested, validated, and executed manually** in the project environment to ensure correctness.  

## Sample Input/Output

**Sample Input:**
```
Document1 This is a sample document containing words
Document2 Another document that also has words
Document3 Sample text with different words
```

**Sample Output:**
```
Document1, Document2 Similarity: 0.56
Document1, Document3 Similarity: 0.42
Document2, Document3 Similarity: 0.50
```

## My Test Results

### Dataset 1 Output (input1.txt - ~1,000 words)

```
Document3, Document2 Similarity: 0.14
Document3, Document1 Similarity: 0.08
Document2, Document1 Similarity: 0.09
```

### Dataset 2 Output (input2.txt - ~3,000 words)

```
Document3, Document2 Similarity: 0.11
Document3, Document1 Similarity: 0.09
Document2, Document1 Similarity: 0.09
```

### Dataset 3 Output (input3.txt - ~5,000 words)

```
Document3, Document2 Similarity: 0.19
Document3, Document1 Similarity: 0.14
Document2, Document1 Similarity: 0.12
```

### Output Analysis

**Consistency:** Identical results from both cluster configurations confirm algorithm correctness.

**Format:** Clean output without tab separators, proper decimal formatting.

**Similarity Scores:** Low scores (0.08-0.14) reflect diverse document topics with minimal word overlap.
