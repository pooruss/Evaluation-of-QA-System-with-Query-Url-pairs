# Evaluation-of-QA-System-with-Query-Url-pairs
internship tasks
### 竞品评估pipeline：

输入：qu pairs ( ~/liangshihao/evaluate/url_to_para/preprocessing/data/file 1)

输出：每条q的排序最高得分的数据，包括query、url、title、para、probability

- 合并去重Merge and Filter repeated qu:

  ```shell
  cd ~/liangshihao/evaluate/url_to_para/preprocessing
  cat python merge_and_filt.py ./data/file1 (./data/file2) > ./output/qu_total_file
  ```

- 取url:

  ```shell
  cat ./output/qu_total_file | python qu_to_u.py > ../hadoop/data/urls_total_file
  ```

- hadoop上获取url对应段落Urls to Paras ，修改shell脚本中的输出路径、输入file, 修改mapper.py里的file)。**若数据量太大，使用opt.sh**

  ```shell
  cd ~/liangshihao/evaluate/url_to_para/hadoop
  sh run.sh 
  ```

- 分段Para Segmentation:

  ```shell
  cd ~/liangshihao/evaluate/irqa_preprocess_aurora
  a_hadoop fs -cat /user/nlp-aurora/liangshihao/result_directory/* > ./data/paras_data
  
  cat ./data/paras_data | ./python_gcc4/bin/python IrqaPipeline_QW_CD.py ./conf/irqa_pipeline_quanwang.conf > ./result/seg_file
  ```

- 过滤分段结果，取content_se不为空，随机采样抽**100条q**，100条的file为seg_cse_qu_sample:

  ```shell
  cd ~/liangshihao/evaluate/request
  cat ../irqa_preprocess_aurora/result/seg_file |python qu_match_from_pseg_qudata.py ../url_to_para/preprocessing/output/qu_total_file 100 > ./data/seg_content_se_file
  ```

- 用100条q校验Recall & Probability:

  ```shell
  cat ./data/seg_cse_qu_Num_sample | python recall_Compare.py > ./result/recall_compare_sample.res
  ```

  ```shell
  cat ./data/seg_cse_qu_Num_sample  | python prob_Compare.py > ./result/prob_compare_sample.res
  ```

- 总结果Total Recall & Probability:

  ```shell
  cat ./data/seg_content_se_file | python prob_Compare.py > ./result/prob_compare.res
  ```



### 基于Hadoop的url_to_paras（数据量大）：

- 遍历数据库再遍历target_urls时间太长，一晚上只跑了0.01%。省时方法：利用map和reduce中间的sort步骤，先将数据库和target_urls在map阶段全部输出，附上来自数据库的flag（flagA）和来自target_urls的flag（flagB），得到按字典序的输出列表；再在reduce阶段遍历map输出列表，根据flag匹配target_urls。
- 换方法后，200w的url大约需10小时。

- reduce阶段可能报错

```shell
cd ~/liangshihao/evaluate/url_to_para/hadoop/
sh opt.sh
```

