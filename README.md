### 중요한 지식과 내생각과 경험을 메모함.



-----


프롬프트에 여러 기법 먼저 적용하고
토큰이 많이 나오고 정확도가 계속 떨어지는 시점이 온다면
파인튜닝 고민하자

-----






## LLM Application이란?

- 기존 애플리케이션: 코드를 통해 결정적인(Determinstic) Rule Base기반으로 동작함.
- LLM 애플리케이션: 비결정성이 포함된(지멋대로의) LLM의 추론 결과를 바탕으로 동작함
예를들어 100개의 똑같은 질문을 했을때 대답이 다 똑같지가 않음. 개발자는 머리아픔.

> [!NOTE]
> 실제로 책추천 서비스(오늘의책)을 개발할 때 앨런 에이전트에게 똑같은 질문을 여러번했을때 대답이 너무 달라서 고민을 많이 했음. 당시는 gemini로 변경하고 질문을 여러번했을때 대답이 동일하게 나와서 일단락 해결하긴 했음..

- LLM은 사람 수준의 추론을 가능하게 해줘서 기존에 가지고 있던 문제를 쉽게 풀어줄 수 있음 (추천, 분류, 챗봇, AI 어시스턴스 등을 활용한 다양한 어플리케이션이 나오고 있다.)



----



## 기본적인 LLM Application 간단 플로우

1. 클라이언트의 입력이 들어온다.
2. 미리 설정한 프롬프트에 질문을 넣어서 LLM에 요청한다.
  - 필요한 경우 질문에 필요한 데이터를 포함시킨다.(예를들어 테슬라가 어제 얼마였어? chatGPT는 21년9월까지밖에 데이터가 없기 때문에 어제에 대한 가격 정보를 알기위해서는 외부의 정보를 가져와야함. 그런 데이터)
  - 외부 API, 데이터베이스, VectoerDB 등
3. 요청 결과를 바탕으로 새로운 프롬프트, 함수를 실행한다.(CHaining)
4. 최종 결과가 나오면 답변을 반환한다.

> [!NOTE]
> 아... 그래서 앨런 에이전트가 chatGPT 기반임에도 최신 데이터를 답변할 수 있었던거구나...!

- LLM Application은 대부분 결국 Private나 Public 데이터를 기반으로 대답을 해주는 방식으로 개발중 (고객센터 챗봇, 볍률 판례 검색, 주식 리포트 생성)
- 이는 RAG(Retrieval Augmented Generation)이라고 하는데, 사용자 input에 수집된 데이터를 함께 프롬프트에 담아 질문하는 방식임.
- 결국 본질은 1. 질문에 필요한 데이터를 최대한 잘 뽑아서 2. LLM이 잘 대답해줄 수 있도록 하는 것


![image](https://github.com/BaxDailyGit/BaxDailyGit/assets/99312529/ee64a44b-f878-4264-abb2-88b264f52d4b)



-----


## 2. LLM Application > Data Retrieval

결국 데이터가 제일 중요하다. 답변을 할 수 있도록 하기 위해서 데이터를 잘 끌어오는게 중요하다.  

이때 우저 질문(input)에 답변할 때 필요한 정보를 가져오기 위해서 여러가지 방법이 있다.  

![image](https://github.com/BaxDailyGit/BaxDailyGit/assets/99312529/99432250-55bc-437c-ad2e-139def3a9998)
(예시를 들기 위해 llama Index 이미지를 사용한것)  

보면 db들도 있고, Vector Store들도 있고  
API(슬랙, 노션, 외부 Saas api)들도 있고, Raw file들(csv, pdf)도 존재한다. 이것들을 최대한 잘 활용해서 가지고 있는게 중효하다.  


----


### 2.1. 외부 api 활용
- search api (google search, bing, ...)
- web scraper
- SaaS api (slack, notion, ...)
- document loading (pdf, csv, ...)

보통 search api를 활용해서 답변의 신뢰도를 높이는 경우가 많음. (하지만 비쌈)
Langchain, LlamaIndex, Unstructured 같은 오픈소스들에서 많이 지원해줌)

search api는 예를들어, 테슬라 어제 얼마였어? 외부에서 데이터를 가져와야하는데 야후 파이넨스나 해외 주식 정보를 제공하는 api나 구글 서치 api를 사용해서 결과를 반환받을수 있다. (다만 비쌈;;)

web scraper는 웹에서 html을 뽑아서 사용하는것

Langchain, LlamaIndex, Unstructured 같은 오픈소들에서 많이 지원해줌
외부 데이터 소스를 어떻게 가져오냐? 그것을 쉽게 해주는게 오픈소스들


----

### 2.2. Structured 데이터베이스 활용
- data warehouse(bigquery, snowflake, ...), RDBMS

- 질문을 엔진에 질의할 수 있는 SQL 형태로 변경 필요(LLM에게 요청하기)
  ㄴ 예를들어
1. Question to LLM: 1주일 전 테슬라 가격 알려줘.
2. LLM Answer: SELECT ticker, price FROM stock_price WHERE created_at = "2024-06-11 00:00:00"
3. Action: RDBMS에 쿼리

----

### 2.3. 키워드(keyword)기반 혹은 전문 (Full Text) 검색
- Elastic search, solr, openSearch 등
- 기본적으로 많이 알려진 검색 엔진들 활용하기
- Saas(Algolia)형태로 제공해주는 엔진을 사용하는 것도 빠른 시도에 도움이 될듯

----



### 2.4. 유사도 기반 검색 (Similarity Search) (요새 워낙 핫함함)
- 두 벡터의 거리를 기반으로 유사도를 측정함
    - 일반적으로 텍스트를 임베딩(Embedding)시킨 벡터로 변환하여 거리 기반 유사도를 검사
- 임베딩 모델을 통해 텍스트를 벡터로 임베딩하게 됨.
- 임베딩된 벡터들이 들어간 데이터베이스를 vectorDB라고 함
    - VectorDB는 벡터간 유사도 검색을 내부에서 지원해줌
    - 사용자는 그냥 API로 손쉽게 검색만 하면됨.


----

VectorDB는 현재 크게 게임 체인저는 없음. 다 비슷비슷함.
- Chroma, FAISS (테스트 환경, Evaluate 등의 로컬 환경에서 돌릴 때 유용)
- Pinecone, Milvus, Weaviate 같이 요새 핫한 친구들도 나옴
- Elastic Search, Redis 처럼 범용 데이터베이스에서 지원해주기도함

결국 유사도 검색의 핵심은 질문의 벡터와 답변의 벡터가 잘 연결될 수 있도록 Embedding 하는 방식이 중요
- Embedding API로 openAI에서 제공하는 Embedding API 많이 사용
    - (최근 업데이트) openAI Embedding API의 가격이 75% 저렴해짐
- Embedding Model도 계속 발전하고 있으니 계속 지켜봅시다.

> [!NOTE]
> 스프링부트에서 벡터db와 mysql같은 rdbms를 함께 사용할수 있도록 세팅 해봐야겠다.

----


그랩님은 Pinecone을 사용한다고 하심.
선택한 가장 큰 이유는 
- 직관성 + 쉬운 사용 (free trial 제공하면서 넘어옴)
- hybrid search(metadata filter) + upsert도 지원 (다만 saas 형태로만 지원하고 self hosting이 안되는 건 아쉬움)

----

### 4.1. TIP 

생각보다 vectorDB의 유사도 검색이 정확하게 잘 가져오지 못한다. 

사실 검색 결과를 높이기 위해선 Semantic Search(vectorDB) + Keyword Search를 함께 적용하는 게 좋음.

일반적으로 검색을 통해 다양한 후보군을 가져온 후 ReRanking하는 작업 진행함(구글도 그렇다고 함)

Cohere가 요새 뜬다는데 쓰는 것도 고려중(api 비용이 막 저렴한 건 아니라서 고민중)

![image](https://github.com/BaxDailyGit/BaxDailyGit/assets/99312529/13a89528-b37e-4bfb-8138-efd0b4c016d5)
(Cohere이미지)

----

### 4.2. vectorDB의 매칭 확률을 높이기 위해서 여러가지 시도를 해보는 것도 의미가 있음

예를들어) 텍스트 데이터를 어떤 방식으로 넣을 건지 고민해보자

- 어떻게 쪼개서 넣을거니? (어떻게 잘 넣어야지 유사도 검색을 잘 할 수 있을까)
    - Chunk Size, overlap 여러개로 테스트해보기(예를들어 텍스트로 된 긴 기사가 있을때 그것을 어느정도의 SIZE로 어느정도의 중복도를 가지게해서 쪼갤것인가 -> 테스트를 많이 해보기)
    - pinecone은 chunksize를 256~512 token size로 추천함(openai embedding model)

- input을 어떻게 넣을 것인가?
    - 질문이 복잡하다면 쪼개보자!(예를들어 질문 자체를 벡터화 시킬때 그 과정에서 어떻게 가공할것인지)
    - input을 답변처럼 가공해서 쿼리하는 HYDE 방식도 있음


----

### 4.3. Embedding 방식을 바꿔보는 것도 테스트 해봐도 좋음
- 실제 벤치마크 점수 기준으로 openai의 text-embedding-add-002를 이기는 임베딩 모델들 나옴
- 하지만 편하게 쓸 수 있는 건 아직까지 openAI Embedding이 짱(cohere는 어떤지 궁금)

----

### 4.4. 다양하게 vectorDB 인덱스를 구성하면서 테스트를 해보는 게 중요

뒷단에서 데이터를 빠르게 넣고 + 멱등하게 관리할 수 있는 Data Engineering Infra 필요


----


## 3. LLM Application > LLM Library

주로 랭체인, 라마인덱스, 시맨틱 커널 등 오픈소스들이 나오고 있다.

LLM Application을 구성하기 위해서 필요한 게 생각보다 있다.
- prompt 템플릿 + 변수 관리
- 다양한 외부 데이터 접근(api)
- vectorDB Embedding + Search
- (이전 답변에 컨텍스트가 필요하다면) Short Term memory
- (복잡한 태스크 수행이 필요하다면) Agent
- ...

----


### 3.1. 렝체인

- 랭체인이 llm application 개발에서 가장 많이 쓰임
- llm과 통신하며 수행하는 작업 단위를 chain으로 만들어서 관리
- vectorDB 관련 인터페이스가 가장 직관적이고 깔끔
- Python, javascript 모두 지원.

> [!NOTE]
> 자바도 가능하다! (langchain4j이라고 찾았다ㅋㅋ)(https://github.com/langchain4j/langchain4j)

- 랭체인 생태계 + 커뮤니티가 가장 활발하다
- 근데 다큐먼트 솔직히 넘 불친절하다.
    - Pinecone에서 런북 만든게 설명이 더 나은듯 (https://www.pinecone.io/learn/series/langchain/ 참고)

- 랭체인에 뭐가 많기는 하지만 결국 유즈케이스에 맞게 직접 커스터마이징 하게 됨
- 랭체인이 좋은건 뛰어난 확장성 때문
    - 우리는 우리 용도에 맞게 커스터마이징 많이 해서 사용중
    - 여러 용도에 맞게 custom chain들 만들어서 사용
    - conversational context를 유지하기 위해 custom memory(서비스db와 통신)를 만들어 구현
    - llm 결과 모니터링을 위해 custom callback 구현해서 메타데이터, 지표 모니터링

----

### 3.2. 라마인덱스

라마인덱스는 넘어감. 일단 랭체인 먼저 사용해보고 필요하면 공부하기로!

----

### 3.3. 복잡한 유즈케이스

- 하나의 프롬프트에 많은 역할과 추론 과정을 요구할 때, 원하는 대로 말을 안들을 때가 많음.
    - 만약 이게 가능한 모델이 있다하더라도(gpt-3.5는 우선 아님) 좋은 prompt를 만지는 데 시간을 꽤 투자해야함
- 미국주식 q&a 챗봇 예시(복잡한 유즈케이스 예시)
    - 1. 질문 유형에 따른 다양한 요구사항이 있음
        - 시황을 물어볼 경우 시의성 중요함(어제 왜 떨어졌어? 어제에 대한 정보로 대답을 해줘야 하는데 옛날 데이터로 가져와서 대답하면 안됌.)(최신 데이터에 대한 질문을 한달 전 데이터에 대한 답변을 하면 안됌.)
        - 특정 주식의 fundamental 정보를 물어볼 경우 해당 정보에 접근하는 api 활용
    - 2. 질문이 여러 문맥을 포함하고 있을 수 있음
        - 최근 3개월 간 가장 수익률이 놓은 주식의 PER은 몇이야?(이런 고급스러운 질문은 아직 하면 안된다.)(금지!!)

- 해결방법 (두 방식을 보통 같이 활용하곤 합니다.)
    - 1. 프롬프트의 역할을 명확하게 해서 쪼갠 후 Chaining하기
    - 2. Retrospective하게 추론 & 실행을 반복하는 Agent 활용하기

----

#### 3.3.1. 복잡한 유즈케이스(Chaining기법)
Chaining을 많이 할것이다.
프롬프트 하나로만 모든게 해결되지 않기 때문에 보통은 프롬프트의 결과를 다시 프롬프트에 넣는 이런 chaining작업들이 있다.

![image](https://github.com/BaxDailyGit/BaxDailyGit/assets/99312529/f1831ada-7e14-4e2a-b2d5-32109acae828)

이미지를 보면 시퀀셜하게 결과에 그대로 넣을수도 있다. 그 결과를 가지고 또하나의 병렬적으로 실행하게 되는 케이스도 있고

ifelse 케이스도 있고
forloop형태로 에이전트 방식등이 있다.

- 프롬프트를 chaining하여, 개별 단계에서 답변의 정확도를 높이기 (우리는 파이프라인이라고 부름)
- 다만, 답변 생성 시간/토큰 비용이 trad-off + 초반 프롬프트에서 답변이 이상하면 downstream으로 에러가 전파될 수 있음.
- 파이프라인을 잘 구성하기 위해서는 엔지니어링 리소스가 많이 들어감
    - 예외처리, 유닛테스트, E2E테스트 모두 진행해봐야 함.
- 그랩님 팀도 여러 파이프라인을 구성하여 답변 퀄리티를 높이기 위해 E2E테스트를 수시로 진행함.

----

#### 3.3.2. 복잡한 유즈케이스(Agent)

chaining하는 기법이 있다면 Agent기법이 있다.  

Agent는 LLM에게 계속 물어보는것이다.  
예를들어 주식 정보를 접근할 수 있는 API가 있고, 최근 뉴스를 가져올 수 있는 API가 있는데 너는 무슨 API를 쓸래?라고 물어보는것   

그러면 LLM이 반복적으로 그것을 생각하면서 수행을 하고  

그런과정에서 툴들을 계속 사용하는것  

- Agent: 목표와 사용가능한 도구를 주면 스스로 행동하도록 하는 방식 혹은 아키텍처 혹은 코드 구현체
- 외부 리소스(구글 서치, open api, 기타 api 등)을 활용해서 복잡한 태스크를 수행해야 할 때 유용함
- AGI(Artificial General Intelligence)를 구현하는 기본적인 방식임.(예를들어 오토gpt처럼 뭔가 목표와 도구를 주면 알아서 하는 애들이 기본적인 Agent의 기본적인 방식을 따르고 있다.)

![image](https://github.com/BaxDailyGit/BaxDailyGit/assets/99312529/140e96ea-2767-4095-8015-d5d711cd4cc6)


쉽게 설명하면 아래와 같이 동작함  
- 1. LLM에게 미리 정의한 툴(Search, VectorDB, 외부 API등)을 알려주고 질문에 대답하기 위해 툴을 선택하게 한다.
- 2. 선택한 툴을 코드로 실행해서 결과를 얻는다.
- 3. LLM에게 결과를 주고 질문에 충분히 대답할 수 있는지 물어본다. 만약 충분히 대답이 되면 결과 반환 후 종류
- 4. 대답이 안되면 답변에 필요한 새로운 질문을 생성해 1~3을 반복한다.

![image](https://github.com/BaxDailyGit/BaxDailyGit/assets/99312529/63bc0648-8628-41c4-9df9-a32001d42eb0)


- Auto-GPT, BabyAGI, Jarvis(HuggingGPT) 등 다양한 아키텍처이자 구현체가 존재함
    - 다만 위 친구들이 얼마나 활용도가 높은지는 모르겠음.(구현체인 만큼 자유도가 떨어짐)
- Agent를 사용하려면 자유도를 높게 가져갈 수 있는 랭체인을 활용하는 걸 추천함.
    - 랭체인으로 augoGPT 구현한 코드도 있음. (https://python.langchain.com/en/latest/use_cases/autonomous_agents/autogpt.html)(Page Not Found 뜸) (나중에 찾아보기)
- Agent는 Action의 Full Cycle이 LLM의 추론으로 돌아가는 방식임. 따라서 운영시 테스트/디버깅이 힘들 수 있음. 충분히 잘 검토해보고  도입 필요가 있음
- 개인적으로는 예측 가능한 솔루션을 만들려면, 너무 복잡한 작업이 아니라면.. Agent 보다는 Prompt를 Chaining하고 + Rule Base(if-else같은)로 만져서 파이프라인을 구성하는게 더 나아보임. (아직까지 Agent를 신뢰하거나 운영 레벨까지 가져가기에는 비용이 꽤 들어간다고 생각하신다고 함)

----


### 4.1. 운영에서 신경 쓸 것

유저 인터페이스를 어떻게 가져가냐에 따라 기능적 요구사항이 달라짐

- 챗봇 형태인가? 챗봇 형태라면
    - 응답의 latency가 중요 -> Streaming 구현(만약 사용자가 질문을 했는데 30초동안 로딩중이라하면 사람들이 중간에 이탈할것)(그래서 first touch latency라고 보통 이야기 하는데 - 처음에 답을 다 안알려주더라도 한글자 일단 내놓아서 사용자에게 가지말라고 Streaming형식으로 구현을 해야함)

> [!NOTE]
> 해본적 있다!! sse방식으로 앨런의 대답을 조금이라도 빠르게 응답했었다.

    - 이전 대화를 바탕으로 대답할 것인가? -> memory 구현 (이전 문맥으로 사용자가 물어본다면 memory도 구현해야한다.)

> [!NOTE]
> 앨런 같은경우 각 client_id별로 따로에 메모리가 구현되어있었다. 메모리 삭제하는 api가 없어서 이스트소프트 운영진께 해당 client_id에 메모리를 삭제하는 api를 제공부탁한 기억이 있다. 그리고 "이전 데이터 초기화" 버튼을 만들었었다.


- 사용자가 어떻게 사용할 수 있는가?
    - Quota가 따로 없다면 ChatGPT의 경우 Rate Limit을 신경써야 함.
        - 백그라운드에서 Bulk로 돌리는 경우가 많다면 openAI Organization 추가하는게 좋음(예를들어 test환경에서 100개의 질문을 한번에 llm에게 요청을 할거야라고한다면 openAI에서 Organization을 하나 추가해서 api 키 2개를 두고 실행하게 하는게 좀더 나을수 있다.)
    - 다만 인증이 없다면 외부 공격에 취약할 수 있음 (다 돈이다!)

----

### 4.2. 운영에서 신경 쓸 것

LLM Application 운영시 주요 metric
- Token Size
- First touch latency
- Last touch latency

후행 지표로 답변 Quality도 체크해야 함  
- 답변에 대한 Evaluation을 하는 파이프라인을 뒷단에 구성하는 것도 방법


> [!NOTE]
> Token Size하니깐 생각나는게 gemini에게 책을 추천받을때 대답하는 책 1개당 토큰 10개씩 잡아먹는것을 알 수 있었던 경험이 있다.
테스트를 여러번 해 본 결과 대답하는 책 1개를 10개로 잡고 max_token 설정하면 대부분 딱맞았다. (예를들어 "책 10개를 추천해줘"라고 한다면 max_token을 100으로 설정하면 딱 맞음.)
그래도 max_token을 더 여유롭게 설정하고 예외처리를 해두었던것 같다.  

4.3. 운영에서 신경 쓸 것

LLM이 답답한 것이 답변을 항상 잘 주는게 아니고 이상할때도 있고, 프롬프트 조금만 바꿔도 답변이 다르게 나올수도 있다.  

- 그래서 답변 퀄리티에 대해서 사람이 지속적으로 Evaluation하고 Quality Control 필요
- 평가 기준을 세우고 일관된 템플릿으로 답변 결과에 대한 퀄리티 비교 및 제안

----

### 4.4. 운영에서 신경 쓸 것

Data Ingestion Infra 고민도 해봐야함.

- 결국 원본 데이터를 보관하기 위한 Stage(Data WareHous, 운영 DB 등)를 두고 용도에 맞게 ingest 시켜야 함.
- 워크플로우 툴(ex. Airflow)을 적용하는게 추후 좋을 수 있음.

LLM Application하나 개발하는데 들어가는 노력과 비용이 크다. 인프라도 당연히 고민을 해봐야한다. (데이터 엔지니어들이 보통 하는 작업들)  

----

## 5. 그랩님이 LLM Application을 하면서 느낀점

- 해당 애플리케이션을 개발한다는 것은 AI Engineering + Data Engineering(+MLOps) + Beckend
- 생각보다 답변을 제대로 하는 LLM Application을 만들기까지 노력이 꽤 들어감
    - LLM 답변에 대한 퀄리티와 신뢰성을 높이기 위한 작업이 쉽지 않음
    - 결국 답변 퀄리티를 높이기 위한 Ops 환경 구성이 생각보다 비용이 들음(MLOps와 유사)
- 결국 LLM Application의 품질은 답변과 직접적으로 연결되어 있음. 답변 퀄리티를 높이기 위해선 Data Retrieval이 제일 중요한 것 같음
    - 프롬프트를 계속 만지는 것보다 질문에 적합한 Data를 가져오도록 하는게 더 나을수도
    - 더 나은 답변이 나오도록 계속해서 실험해봐야 함. 이를 위한 실험 환경도 구성되어야 함
- 랭체인 빨리 다뤄봐라.















