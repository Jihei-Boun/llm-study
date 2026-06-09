# 04. Transformer

<br>

## 한 줄 설명

Transformer = Attention을 여러 층으로 쌓아서, 문장의 맥락을 점점 더 깊게 이해하는 모델

<br>

## 왜 필요한가?

Attention 노트에서 정리한 것처럼, 옛날 방식(RNN)은:

* 왼쪽→오른쪽 **순서대로** 읽고
* **기억상자 하나**에 압축해서 저장해서
* 문장이 길어지면 **앞부분을 잊기 쉬웠음**

Self-Attention은 "멀리 있는 단어도 바로 참고"할 수 있게 해줬지만, **한 번만** 하면 아직 부족함.

예를 들어:

```text
"철수는 어제 공원에서 친구를 만났고, 그는 기분이 좋아 보였다"
```

* 1번째 Attention: "그"가 "철수"를 볼 수 있음
* 2번째 Attention: "기분이 좋아"가 "만났다"와 연결됨
* 3번째 Attention: 문장 전체 의미가 더 정리됨

**한 번 보는 것보다, 여러 번 보면서 이해가 깊어지는 것**이 Transformer의 핵심.

<br>

✅ 정리

> Self-Attention 한 번 = 문장을 한 번 훑어보기  
> Transformer = 그걸 여러 층 반복해서 깊게 이해하기

<br>

## 비유: 회의를 여러 라운드 돌리기

Attention 노트의 회의실 비유를 이어가면:

```text
[나] [철수] [영희] [민수] [희준]
```

**1라운드 (1층 Transformer)**
* 각자 고개를 돌려 누구 말이 중요한지 파악
* "예산" 이야기 → 희준에게 귀 기울임

**2라운드 (2층 Transformer)**
* 1라운드에서 얻은 정보를 바탕으로 **다시** 고개를 돌림
* "예산 + 일정"이 같이 얽혀 있다는 걸 더 잘 이해

**3라운드, 4라운드...**
* 점점 더 복잡한 관계까지 파악

Transformer = **같은 회의실에서 고개 돌리기(Attention)를 여러 층 반복**하는 것

<br>

## Transformer 안에 뭐가 들어있나?

한 층(Layer)은 대략 이렇게 생김:

```text
입력 (단어들의 Embedding)
   ↓
① Self-Attention  ← 문장 안 단어들끼리 서로 참고
   ↓
② Feed Forward    ← 각 단어를 혼자서 한 번 더 가공 (작은 신경망)
   ↓
③ Layer Norm      ← 값이 너무 커지지 않게 정리
   ↓
출력 (더 똑똑해진 단어 표현)
```

이걸 **6층, 12층, 96층...** 쌓음. (GPT-3는 96층)

<br>

### 각 부품 쉬운 설명

| 부품 | 쉬운 뜻 |
|------|---------|
| **Self-Attention** | 같은 문장 안 단어들끼리 서로 참고 (03 Attention에서 배움) |
| **Multi-Head Attention** | Attention을 여러 관점에서 동시에 함 (주어/목적어, 시간/장소 등) |
| **Feed Forward (FFN)** | Attention으로 모은 정보를 각 단어별로 한 번 더 정리 |
| **Layer Norm** | 숫자가 폭발하지 않게 안정화 |
| **Positional Encoding** | "순서" 정보 추가 (Attention은 순서를 모르니까) |
| **Residual Connection** | 층을 지나도 원래 정보를 잃지 않게 이어줌 (`x + 층(x)`) |

<br>

## Multi-Head Attention이란?

Self-Attention을 **한 번**만 하는 게 아니라, **여러 관점에서 동시에** 함.

```text
문장: "고양이가 쥐를 잡았다"

Head 1 (누가?):     잡았다 → 고양이 (많이)
Head 2 (뭘?):       잡았다 → 쥐 (많이)
Head 3 (문법?):     잡았다 → 가, 를 (조금)
```

한 사람이 한 방향만 보는 게 아니라, **여러 사람이 각자 다른 각도에서** 문장을 보는 것과 비슷함.

<br>

✅ 정리

> Multi-Head = Self-Attention을 여러 "관점"에서 동시에 돌리는 것

<br>

## 원래 Transformer vs 지금 LLM (GPT)

2017년 논문 *"Attention Is All You Need"*의 원래 구조:

```text
[Encoder]  ← 입력 문장 이해 (Self-Attention)
     ↓
[Decoder]  ← 출력 문장 생성 (Self-Attention + Cross-Attention)
```

* **Encoder**: 원문을 이해 (번역의 영어 문장)
* **Decoder**: 결과를 생성 (번역의 한국어 문장)
* **Cross-Attention**: 디코더가 인코더의 원문을 참고

하지만 **지금 대부분의 LLM (GPT, Claude 등)** 은 **Decoder만** 씀:

```text
[Decoder만] → 이전 글자들을 보고 다음 글자 예측
```

GPT는 "번역"이 아니라 "다음 단어 맞히기"가 목적이라, Encoder 없이 Decoder만으로 충분함.

<br>

## 전체 흐름에서의 위치

```text
"고양이가 쥐를 잡았다"
        ↓
① Tokenization    → [고양이, 가, 쥐를, 잡았다]
        ↓
② Embedding       → 각 단어를 숫자 벡터(좌표)로
        ↓
③ Positional Enc. → 순서 정보 붙이기 (1번째, 2번째...)
        ↓
④ Transformer Layer 1  → Self-Attention + FFN
        ↓
⑤ Transformer Layer 2  → 또 Self-Attention + FFN
        ↓
        ... (수십 층 반복)
        ↓
⑥ 마지막 층 출력   → 각 단어가 "문맥이 반영된" 똑똑한 벡터
        ↓
⑦ 다음 단어 예측   → "다" 다음에 뭐가 올까?
```

<br>

✅ 정리

> Embedding이 각 단어의 지도 좌표를 만든다면,  
> Attention은 그 좌표를 해석할 때 어떤 다른 좌표를 볼지 정하고,  
> Transformer는 그 과정을 여러 층 반복해서 문장 전체를 깊게 이해함

<br>

## RNN vs Transformer 한눈에

| | RNN (옛날) | Transformer |
|--|-----------|-------------|
| 읽는 방식 | 순서대로 하나씩 | **모든 단어 동시에** |
| 기억 | 작은 상자 1개 | Attention으로 필요한 것만 직접 참고 |
| 깊이 | 층이 깊어도 순차 처리 | **층을 쌓아** 이해를 깊게 |
| 병렬 처리 | 느림 (순서 의존) | **빠름** (동시 계산 가능) |
| 긴 문장 | 앞부분 잊기 쉬움 | 멀리 있어도 바로 연결 |

<br>

## 핵심 정리

1. **Transformer = Attention을 여러 층 쌓은 것**
2. **한 층 = Self-Attention + Feed Forward + 정리(Layer Norm)**
3. **Multi-Head = 여러 관점에서 동시에 Attention**
4. **층이 많을수록** 문장의 복잡한 관계를 더 잘 이해
5. **GPT 같은 LLM**은 Decoder만 써서 "다음 단어 예측"

<br>

## 다음 단계

Attention 노트 흐름대로라면, 다음은 **Next Token Prediction**.

Transformer가 문장을 깊게 이해한 뒤, 마지막에 **"다음에 올 단어는 뭘까?"** 를 확률로 예측하는 단계로 넘어감. ChatGPT가 한 글자씩 답을 쓰는 원리가 바로 그것.

<br>

## 내가 이해한 내용  
  
Transformer = Attention을 여러 층 쌓아서 문장의 맥락을 더 깊이있게 이해하는 방식  
  
Self-Attention은 문장 안 단어들끼리 동시에 비교하며, 순서를 하나씩 읽지 않기 때문에 스스로 몇 번째 단어인지 알 수 없다.  
-> Embedding + Positional Encoding  
* Positional Encoding은 **매 층마다**가 아니라 **입력 준비 단계에서 한 번**  
  
  
Transformer이 층을 지날 때마다 Self-Attention + FFN을 진행한 다음 마지막 층을 출력.  
  
<br>

## 아직 헷갈리는 부분

```text
Layer의 실제 구조

입력 x
   ↓
┌─ Multi-Head Self-Attention ─┐
│         + Residual           │  ← x + Attention(x)
│         Layer Norm           │
└─────────────────────────────┘
   ↓
┌─ FFN (Feed Forward) ────────┐
│         + Residual           │  ← x + FFN(x)
│         Layer Norm           │
└─────────────────────────────┘
   ↓
다음 층으로
```

### 비유  
  
회의실 비유로:  
| 단계 | 역할 |
|------|------|
| Self-Attention | 다른 사람 말 듣기 (고개 돌리기) |
| Layer Norm | 정리 ("너무 excited하지 않게") |
| FFN | 혼자 메모 정리하기 |
| Layer Norm | 또 한번 정리 |