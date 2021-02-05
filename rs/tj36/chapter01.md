# 结构化文本

## 关键词提取

### TF-IDF
TF-IDF算法在很多场景都会用户，比如在IR（信息检索 ）中计算词项权重在文档中的权重用来排序召回文档。可以参考我在[Lucene源码解析系列](https://github.com/hongfuli/study_notes/blob/master/lucene/blog_3.md)文章。
TF: 与 词项在某个文档中出现的频率 成正比\
IDF: 与 包含有该词项的所有文档数量 成反比

思路就是先用`中文分词`技术把文档分词，然后对每个词计算TF-IDF值，最后取topK值。代码实现可以参考[jieba](https://github.com/fxsjy/jieba/blob/master/jieba/analyse/tfidf.py)分词。

```python
class TFIDF(KeywordExtractor):

    def __init__(self, idf_path=None):
        self.tokenizer = jieba.dt
        self.postokenizer = jieba.posseg.dt
        self.stop_words = self.STOP_WORDS.copy()
        self.idf_loader = IDFLoader(idf_path or DEFAULT_IDF)
        self.idf_freq, self.median_idf = self.idf_loader.get_idf()

    def set_idf_path(self, idf_path):
        new_abs_path = _get_abs_path(idf_path)
        if not os.path.isfile(new_abs_path):
            raise Exception("jieba: file does not exist: " + new_abs_path)
        self.idf_loader.set_new_path(new_abs_path)
        self.idf_freq, self.median_idf = self.idf_loader.get_idf()

    def extract_tags(self, sentence, topK=20, withWeight=False, allowPOS=(), withFlag=False):
        """
        Extract keywords from sentence using TF-IDF algorithm.
        Parameter:
            - topK: return how many top keywords. `None` for all possible words.
            - withWeight: if True, return a list of (word, weight);
                          if False, return a list of words.
            - allowPOS: the allowed POS list eg. ['ns', 'n', 'vn', 'v','nr'].
                        if the POS of w is not in this list,it will be filtered.
            - withFlag: only work with allowPOS is not empty.
                        if True, return a list of pair(word, weight) like posseg.cut
                        if False, return a list of words
        """
        if allowPOS:
            allowPOS = frozenset(allowPOS)
            words = self.postokenizer.cut(sentence)
        else:
            words = self.tokenizer.cut(sentence)
        freq = {}
        for w in words:
            if allowPOS:
                if w.flag not in allowPOS:
                    continue
                elif not withFlag:
                    w = w.word
            wc = w.word if allowPOS and withFlag else w
            if len(wc.strip()) < 2 or wc.lower() in self.stop_words:
                continue
            freq[w] = freq.get(w, 0.0) + 1.0
        total = sum(freq.values())
        for k in freq:
            kw = k.word if allowPOS and withFlag else k
            freq[k] *= self.idf_freq.get(kw, self.median_idf) / total

        if withWeight:
            tags = sorted(freq.items(), key=itemgetter(1), reverse=True)
        else:
            tags = sorted(freq, key=freq.__getitem__, reverse=True)
        if topK:
            return tags[:topK]
        else:
            return tags
```

### TextRank
借鉴 [PageRank](https://en.wikipedia.org/wiki/PageRank) 思想，在PageRank 中，网页之间相互引用关系提供重要性证明，在 TextRank 中，词与词相邻出现关系提供重要性证明，从页选出文本中重要的关键词。

阅读论文：[TextRank: Bringing Order into Texts](https://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf)

## FastText
[FastText](https://fasttext.cc/) 是 Facebook 开源的文本分类和文本词向量相关工程实现。

## 实体识别
算法上可以参考 jieba 里的基于 HMM(隐马尔科夫模型)。

## Word Embedding
词嵌入，可以根据数据集学习出一个稠密的词向量模型。这样我们可以得到词与词之前的相似度，或者根据已有词之间的关系，推算出某个词的类比关系。当前比较好的工程实现是 Word2Vec。

