---
published: true
title:  "[Python] KoNLPy, 사이킷런을 이용한 주요 어휘 추출"
excerpt: "KoNLPy 와 Scikit-learn 을 이용해 텍스트에서 주요 어휘 추출하는 방법을 알아보자🙋‍♀️"

categories:
  - About Dev
tags:
  - [NLP ,Python]

toc: true
toc_sticky: true
 
date: 2022-04-29 02:12:08
last_modified_at: 2022-04-29 02:12:12
---

<br>

텍스트에서 주효 어휘를 추출해야 할 일이 생겼다. 단순 단어의 빈도수만 따져서 주요 어휘를 추출하는 것은 의미가 없을 것 같아 자연어처리 쪽을 알아보았다. 파이썬에 유용한 라이브러리가 많아 파이썬으로 구현해 보았다.

# 🏷️ 텍스트 마이닝과 Scikit-learn

텍스트 마이닝이란 비정형 테스트에서 자연어 처리 기술을 활용하여 패턴, 관계 등을 분석해 의미있는 정보를 도출해내는 데이터 마이닝(Mining) 기법이다. 기존에 문자로 구성돼 있던 데이터 모델에 적용할 수 있도록 특징을 뽑아 어떤 값으로 바꿔서 수치화한다.

*Scikit-learn(사이킷런)*은 파이썬용 머신러닝 라이브러리이다. 머신러닝 기술을 활용하는 데 필요한 다양한 모듈을 제공하는데, 오늘은 그중 텍스트 마이닝을 위한 자연어처리 모듈들은 다음과 같다.

**📍 자연어 특징 추출 Scikit-learn 모듈:**
- CountVectorizer: 각 텍스트에서 횟수를 기준으로 특징을 추출하는 방법
- TfidfVectorizer: **TF-IDF**라는 값을 사용해 텍스트에서 특징을 추출
- HashingVectorizer: CounterVectorizer와 사용방법은 동일하지만 텍스트를 처리할 때 해시 함수를 사용하기 때문에 실행 시간을 크게 줄일 수 있음. 텍스트의 크기가 클수록 HashingVectorizer를 사용하는 것이 좋음

`CountVectorizer` 는 단순 횟수를 특징으로 잡기 때문에 자주 사용되는 조사나 관사, 지시대명사 등이 높은 특징값을 가져 유의미하게 사용하기 어려울 수 있다. 그래서 **TF-IDF**를 활용한 `TfidfVectorizer`을 사용하기로 했다.

먼저 TF-IDF 에 대해 알아야 한다.

# 🏷️ TF-IDF

![image](https://user-images.githubusercontent.com/67352902/165788715-99d28472-f135-4724-9dad-df9cae702772.png){: .align-center}
*TF-IDF*

- TF(Term Frequency): 특정 단어가 데이터에서 등장하는 횟수
- DF(Document Frequency): 문서 빈도 값, 특정 단어가 얼마나 자주 등장하는지 알려주는 지표
- IDF(Inverse Document Frequency): 특정 단어가 다른 데이터에 등장하지 않을수록 값이 커짐
  - 조사나 지시대명사처럼 자주 등장하는 단어는 TF는 크지만 IDF는 작아지므로 CountVectorizer가 가진 단점을 해결 할 수 있다.

# 🏷️ TfidfVectorizer

사이킷런에선 TF-IDF를 `TfidfVectorizer`로 쉽게 구현 할 수 있게 해준다.

각 모듈들의 사용법은 크게 다르지 않다. 사이킷 런 공식문서에 자세하게 나와있다. ( [CountVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html#sklearn.feature_extraction.text.CountVectorizer), [TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html) )

기본적인 사용법은 아래와 같다.

```python
from sklearn.feature_extraction.text import TfidfVectorizer
import pprint

corpus = [
    'This is the first document.',
    'This document is the second document.',
    'And this is the third one.',
    'Is this the first document?',
]

vectorizer = TfidfVectorizer()
vectorizer.fit(corpus)
matrix = vectorizer.transform(corpus)

pprint.pprint(vectorizer.vocabulary_) # 단어 사전
pprint.pprint(matrix.toarray()) # 분석 결과
```

```
# 단어 사전
{'and': 0,
 'document': 1,
 'first': 2,
 'is': 3,
 'one': 4,
 'second': 5,
 'the': 6,
 'third': 7,
 'this': 8}

# 분석 결과
array([[0.        , 0.46979139, 0.58028582, 0.38408524, 0.        ,
        1.        , 0.38408524, 0.        , 0.38408524],
       [0.        , 0.6876236 , 0.        , 0.28108867, 0.        ,
        0.53864762, 0.28108867, 0.        , 0.28108867],
       [0.51184851, 0.        , 0.        , 0.26710379, 0.51184851,
        1.        , 0.26710379, 0.51184851, 0.26710379],
       [0.        , 0.46979139, 0.58028582, 0.38408524, 0.        ,
        1.        , 0.38408524, 0.        , 0.38408524]])
```

`vectorizer.fit(corpus)`를 통해 corpus가 가지고 있는 모든 단어를 BoW로 구성하고, 이 단어들에 대해 TF-IDF 계산을 한 뒤 각 단어의 인덱스 위치에 TF-IDF 값이 들어간 벡터가 만들어진다. 표현되지 않은 값들은 모두 0이다.(한글자 같은)

## 🔷 예쁘게 출력하기

단어와 value를 함께 출력하고, 내림차순으로 정렬하고 싶다면 딕셔너리를 사용해 이렇게 구현 하면 된다.

```python
from collections import defaultdict

# 단어 사전: {"token": id}
vocabulary_word_id = defaultdict(int)
    
for idx, token in enumerate(vectorizer.get_feature_names()):
    vocabulary_word_id[token] = idx
    
# 특징 추출 결과: {"token": value}
result = defaultdict(str)
    
for token in vectorizer.get_feature_names():
    result[token] = matrix[0, vocabulary_word_id[token]]
    
# 내림차순 (중요도 high) 기준 정렬
result = sorted(result.items(), key = lambda item: item[1], reverse = True)
pprint.pprint(result)
```
```python
# 결과
[('first', 0.5802858236844359),
 ('document', 0.46979138557992045),
 ('is', 0.38408524091481483),
 ('the', 0.38408524091481483),
 ('this', 0.38408524091481483),
 ('and', 0.0),
 ('one', 0.0),
 ('second', 0.0),
 ('third', 0.0)]
```

## 🔷 특정 tokenizer 사용

tokenizer를 지정하여 사용 할 수 있다.

```python
vectorizer = TfidfVectorizer(tokenizer = 사용 할 토크나이저)
```
이렇게 하면 분석할 텍스트에 대해 원하는 전처리를 할 수 있다. 사용 할 토크나이저들은 아래에 소개한다.

# 🏷️ 형태소 분석을 위한 NLP 라이브러리

## 🔷 NLTK: 영어

영어 텍스트 전처리 작업엔 NLTK를 많이 쓴다. 이 글에서는 한글 텍스트 분석을 주로 다루기 때문에 잘 설명되어 있는 [레퍼런스](https://mr-doosun.tistory.com/23)만 소개하겠다.

## 🔷 KoNLPy: 한국어

한국어 정보처리를 위한 파이썬 패키지이다. 참고로 '코엔엘파이' 라고 읽는다. KoNLPy는 다양한 태깅 패키지들을 제공한다. [공식문서](https://konlpy.org/ko/latest/index.html) 가 잘 되어있기 때문에 각 태깅 클래스들의 성능을 비교하려면 쭉 읽어보는것도 좋다.

**tag Classes:**
- Hannanum
- Kkma
- Komoran
- Mecab (Mac에서만 사용 가능)
- Okt

이중 Okt 사용해 보려고 한다. 필자도 이 글을 쓰다가 알게 된건데 기존의 Twitter 패키지는 Okt로 변경 되었다고 한다.([#141](https://github.com/konlpy/konlpy/issues/141))

### 🔸 KoNLPy 사용법

```python
>>> from konlpy.tag import Okt
>>> okt = Okt()
>>> print(okt.morphs(u'단독입찰보다 복수입찰의 경우'))
['단독', '입찰', '보다', '복수', '입찰', '의', '경우']
>>> print(okt.nouns(u'유일하게 항공기 체계 종합개발 경험을 갖고 있는 KAI는'))
['항공기', '체계', '종합', '개발', '경험']
>>> print(okt.phrases(u'날카로운 분석과 신뢰감 있는 진행으로'))
['날카로운 분석', '날카로운 분석과 신뢰감', '날카로운 분석과 신뢰감 있는 진행', '분석', '신뢰', '진행']
>>> print(okt.pos(u'이것도 되나욬ㅋㅋ'))
[('이', 'Determiner'), ('것', 'Noun'), ('도', 'Josa'), ('되나욬', 'Noun'), ('ㅋㅋ', 'KoreanParticle')]
>>> print(okt.pos(u'이것도 되나욬ㅋㅋ', norm=True))
[('이', 'Determiner'), ('것', 'Noun'), ('도', 'Josa'), ('되나요', 'Verb'), ('ㅋㅋ', 'KoreanParticle')]
>>> print(okt.pos(u'이것도 되나욬ㅋㅋ', norm=True, stem=True))
[('이', 'Determiner'), ('것', 'Noun'), ('도', 'Josa'), ('되다', 'Verb'), ('ㅋㅋ', 'KoreanParticle')]
```
예제 출처: [공식문서](https://konlpy.org/ko/latest/api/konlpy.tag/#okt-class)

- `morphs()` : 텍스트를 형태소 단위로 나눈다. 옵션으로 norm(문장 정규화), stem(어간 추출) 이 있는데, True/False 값을 주면 된다.
- `nouns()` : 텍스트에서 명사만 추출
- `pharases()` : 텍스트에서 어절을 추출
- `pos()` : 각 품사를 태깅. 주어진 텍스트를 형태소 단위로 나누고, 각 형태소에 해당하는 품사와 함께 리스트화 한다. 옵션으로 norm, stem, join이 있다.

### 🔸 TfidfVectorizer에 적용

#### KoNLPy 적용 전

좀 전에 [TfidfVectorizer](https://devyuseon.github.io/about%20dev/konlpy-extract-keywords/#tfidfvectorizer) 에서 사용한 코드를 별도의 토크나이저 지정 없이 아래 corpus를 넣어서 돌려보았다.

```python
corpus = [
    '철수는 통계학과에 다닌다.',
    '빅데이터 분석에 필요한 것은 통계학적 지식과 프로그래밍 능력이다.'
    '4차산업의 핵심기술로 인공지능과 빅데이터가 있다.',
    '텍스트자료는 빅데이터에서 중요한 재료이다.'
]
```
예제 출처 : [bigdata.dongguk.ac.kr](http://bigdata.dongguk.ac.kr/lectures/TextMining)

```python
[('다닌다', 0.5773502691896257),
 ('철수는', 0.5773502691896257),
 ('통계학과에', 0.5773502691896257),
 # 0인 결과는 생략
 ]
```

#### KoNLPy 적용 후

```python
def okt_tokenizer(text):
    okt = Okt()
    return okt.nouns(text)

vectorizer = TfidfVectorizer(tokenizer=okt_tokenizer)
# 생략
```

```python
[('철수', 0.5773502691896257),
 ('통계', 0.5773502691896257),
 ('학과', 0.5773502691896257),
 # 0인 결과는 생략
 ]
```
우리가 원하는 데이터에 훨씬 더 가까워졌다.

### 🔸 특수기호 및 불용어 제거

'!,.%$' 와 같은 특수기호나 조사, 접속사, 지시대명사 등의 불용어를 추가로 제거해야 유의미한 데이터를 얻을 수 있다.

영어의 경우 NLTK에서 불용어 리스트를 제공하지만 한국어는 따로 제공되는 리스트가 없다. 따라서 따로 정의해놓고 사용해야 한다.

공개된 한국어 불용어 리스트:
- https://www.ranks.nl/stopwords/korean
- https://bab2min.tistory.com/544

필자는 검색하며 찾은 불용어들을 텍스트 파일로 합쳐서 사용했다.
- [불용어 리스트 합본](https://github.com/2E2I/donation-crawler/blob/main/stopwords.txt)

**텍스트 전처리 소스코드:**

```python
def get_stopwords():
    stopwords = list()
    
    f = open('./stopwords.txt', 'r', encoding='utf-8')
    
    while True:
        line = f.readline()
        if not line: break
        stopwords.append(line.strip())
        
    return stopwords

def okt_tokenizer(text):
    okt = Okt()
    text = re.sub(r'[^ ㄱ-ㅣ가-힣A-Za-z]', '', text) # 특수기호 제거
    stopwords = get_stopwords() # 불용어
    
    return [token for token in okt.nouns(text)
            if len(token) > 1 and token not in stopwords]
```

<br>

전체 소스코드는 [✨여기서✨](https://github.com/2E2I/donation-crawler/blob/main/extract.py) 확인 하실 수 있습니다:)
<br>
