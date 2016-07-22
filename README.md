* alter by thidtc 2016
  - 修改
    + 关闭HMM模型
    + 增加词性标注（不影响现有分词的功能以及接口)
  - 使用过程
    + 运行bin/build.sh得到target目录下的jar包文件
      * jieba-analysis-1.0.3-SNAPSHOT.jar        
      * jieba-analysis-1.0.3-SNAPSHOT-javadoc.jar
      * jieba-analysis-1.0.3-SNAPSHOT-sources.jar

* 结巴分词(java版) jieba-analysis
  首先感谢jieba分词原作者[[https://github.com/fxsjy][fxsjy]]，没有他的无私贡献，我们也不会结识到结巴
  分词，更不会有现在的java版本。

  结巴分词的原始版本为python编写，目前该项目在github上的关注量为170，
  打星727次（最新的数据以原仓库为准），Fork238次，可以说已经有一定的用户群。



* 简介
** 支持分词模式
   - Search模式，用于对用户查询词分词SegMode.SEARCH
   - Index模式，用于对索引文档分词SegMode.INDEX 在Search模式的基础上，对长词再次切分，提高召回率，适合用于搜索引擎分词
   
** python支持三种分词模式：
   - 精确模式，试图将句子最精确地切开，适合文本分析；
   - 全模式，把句子中所有的可以成词的词语都扫描出来, 速度非常快，但是不能解决歧义；
   - 搜索引擎模式，在精确模式的基础上，对长词再次切分，提高召回率，适合用于搜索引擎分词。

** 特性
   - 支持多种分词模式
   - 全角统一转成半角
   - 用户词典功能
   - conf 目录有整理的搜狗细胞词库
   - 因为性能原因，最新的快照版本去除词性标注，也希望有更好的 Pull Request 可以提供该功能
   - 支持繁体分词
   - 支持自定义词典
   
** 算法
   - 基于前缀词典实现高效的词图扫描，生成句子中汉字所有可能成词情况所构成的有向无环图 (DAG)
   - 采用了动态规划查找最大概率路径, 找出基于词频的最大切分组合
   - 对于未登录词，采用了基于汉字成词能力的 HMM 模型，使用了 Viterbi 算法

* 如何获取
  - 当前稳定版本
    #+BEGIN_SRC xml
      <dependency>
        <groupId>com.huaban</groupId>
        <artifactId>jieba-analysis</artifactId>
        <version>1.0.2</version>
      </dependency>
    #+END_SRC

  - 当前快照版本
    #+BEGIN_SRC xml
      <dependency>
        <groupId>com.huaban</groupId>
        <artifactId>jieba-analysis</artifactId>
        <version>1.0.3-SNAPSHOT</version>
      </dependency>
    #+END_SRC


* 如何使用
  - Demo
  #+BEGIN_SRC java

    @Test
    public void testDemo() {
        JiebaSegmenter segmenter = new JiebaSegmenter();
        String[] sentences =
            new String[] {"这是一个伸手不见五指的黑夜。我叫孙悟空，我爱北京，我爱Python和C++。", "我不喜欢日本和服。", "雷猴回归人间。",
                          "工信处女干事每月经过下属科室都要亲口交代24口交换机等技术性器件的安装工作", "结果婚的和尚未结过婚的"};
        for (String sentence : sentences) {
            System.out.println(segmenter.process(sentence, SegMode.INDEX).toString());
        }
    }
  #+END_SRC

* 算法(wiki补充...)
  - [ ] 基于 =trie= 树结构实现高效词图扫描
  - [ ] 生成所有切词可能的有向无环图 =DAG=
  - [ ] 采用动态规划算法计算最佳切词组合
  - [ ] 基于 =HMM= 模型，采用 =Viterbi= (维特比)算法实现未登录词识别

* 性能评估
  - 测试机配置
  #+BEGIN_SRC screen
    Processor 2 Intel(R) Pentium(R) CPU G620 @ 2.60GHz
    Memory：8GB

    分词测试时机器开了许多应用(eclipse、emacs、chrome...)，可能
    会影响到测试速度
  #+END_SRC
  - [[src/test/resources/test.txt][测试文本]]
  - 测试结果(单线程，对测试文本逐行分词，并循环调用上万次)
    #+BEGIN_SRC screen
      循环调用一万次
      第一次测试结果：
      time elapsed:12373, rate:2486.986533kb/s, words:917319.94/s
      第二次测试结果：
      time elapsed:12284, rate:2505.005241kb/s, words:923966.10/s
      第三次测试结果：
      time elapsed:12336, rate:2494.445880kb/s, words:920071.30/s

      循环调用2万次
      第一次测试结果：
      time elapsed:22237, rate:2767.593144kb/s, words:1020821.12/s
      第二次测试结果：
      time elapsed:22435, rate:2743.167762kb/s, words:1011811.87/s
      第三次测试结果：
      time elapsed:22102, rate:2784.497726kb/s, words:1027056.34/s
      统计结果:词典加载时间1.8s左右，分词效率每秒2Mb多，近100万词。

      2 Processor Intel(R) Core(TM) i3-2100 CPU @ 3.10GHz
      12G 测试效果
      time elapsed:19597, rate:3140.428063kb/s, words:1158340.52/s
      time elapsed:20122, rate:3058.491639kb/s, words:1128118.44/s

    #+END_SRC

* 使用本库项目
  - [[https://github.com/sing1ee/analyzer-solr][analyzer-solr]] @sing1ee


* 许可证
  jieba(python版本)的许可证为MIT，jieba(java版本)的许可证为ApacheLicence 2.0
  #+BEGIN_SRC screen
    Copyright (C) 2013 Huaban Inc

    Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
  #+END_SRC
