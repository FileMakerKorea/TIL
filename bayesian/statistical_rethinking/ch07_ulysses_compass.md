# 7. Ulysses' Compass

학자들은 더 간결한 이론을 선호하는 경우가 많다. 이 "선호"라는 것이 조금 애매하기는 해서, 단순히 아름답기 때문에 간결한 것을 좋아할 때도 있다. 
또는 단순한 이론일수록 다루기 쉽기 때문에 선호하는 경우도 있다. 
과학자들은 "가정이 더 적은 모형을 선호한다"는 **오캄의 면도날(Ockham's Razor)** 원칙을 자주 사용한다. 
그런데 정확도가 같은 두 모형이 있다면 단순한 모형이 더 좋다는 결론을 내릴 수 있겠지만, 일반적으로는 정확도와 단순함을 모두 고려하여 모형을 평가하게 된다. 
이 두 가지 기준을 어떻게 절충하여 사용할 수 있을까?

이번 장에서는 그러한 트레이드 오프를 다루기 위해 사용하는 도구들에 대해서 다룬다. 
이 도구들은 단순성에 대한 부분은 기능을 포함하고 있기 때문에 오캄의 면도날과 비교될 수 있다. 
하지만 명시적으로 단순함과 정확도 사이의 트레이드-오프를 이야기하는 오캄의 면도날과는 달리, 정확도를 높이기 위해서 사용하는 것이다.

따라서 여기서는 오캄의 면도날 대신 **율리시스의 나침반** 을 생각해보자. 율리시스는 호메로스의 오디세이에 등장하는 영웅이다. 
여행 중에 좁은 해협을 지나게 되었는데, 스킬라(Scylla) 라고 하는 머리가 여러 개 달린 괴물이 절벽 위에서 공격한다. 
바다에서는 카립디스(Charybdis) 라는 괴물이 배와 선원들을 물 아래로 끌어내리려 한다. 이 괴물들에 가까이 다가가면 공격당하기 때문에 잘 피해서 해협을 통과해야만 한다. 
과학적인 모형에 빗대어 보면 각각의 괴물들이 다음과 같은 두 가지 통계적인 오류를 나타낸다고 볼 수 있다.

1. **Overfitting** : 데이터를 너무 많이 학습하는 바람에 정확도가 낮아지는 경우
2. **Underfitting** : 데이터를 너무 적게 학습하는 바람에 정확도가 낮아지는 경우

그리고 세 번째 괴물로 바로 이전 장에서 다루었던 confounding 이 있다. 이번 장에서는 교란된 모형이 정상적인 모형보다 높은 정확도가 나올 수 있다는 것을 보여 줄 것이다.
따라서 특정한 통계 모형을 설계할 때는 정확도만 사용하는 것이 아니라 원인까지 고려해야 한다는 점을 확인해보자. 
인과 관계를 파악하기 위한 모형과 예측을 위한 모형은 다르기 때문에 애초에 다른 종류의 모형을 사용해야 할 수 있다. 
하지만 인과 관계를 파악하기 위해서라도 과적합(Overfitting) 을 다룰 수 있어야 한다. 

우리가 해야 될 작업은 이러한 괴물들을 조심스럽게 파악하는 것이다. 크게 두 가지 접근 방식이 있다. 
두 방식 모두 자연/사회과학 분야에서 널리 사용되고 있으며, 필요하다면 두 방식 모두 적용하기도 한다.

1. **Regularizing Prior** : 모형이 데이터에 너무 큰 영향을 받지 않도록 조절한다 
2. **Scoring** (Information Criteria, Cross Validation) : 예측 정확도를 수치로 표현한다

# 7.1 The problem with parameters

이전 장에서는 믿을만한 인과 모형이 없을 경우, 변수를 추가하는 것이 오히려 독이 되는 상황을 살펴보았다. 그런데 가끔은 인과 관계가 아니라 그냥 예측을 잘 하기 위한 모형을 만들어야 할 때가 있다. 이런 경우에는 그냥 변수를 추가하기만 하면 좋은 성능을 낼 수 있을까?

정답은 "아니오" 다. 변수를 추가하는 데는 두 가지 문제가 있다. 

1. 파라미터를 추가하면 거의 항상 모형이 데이터에 더 적합(fit)해진다
    - 여기서 적합(fit)하다는 것은 모형을 통해 학습에 사용한 데이터를 얼마나 원본에 근접하게 예측할 수 있는지를 의미한다
    - R-squared 가 이런 종류에 해당하는 대표적인 지표다
2. 모형이 복잡해져서 데이터를 더 잘 설명하게 되면, 새로운 데이터를 잘 예측하지 못하게 되는 경우가 생긴다
    - 파라미터가 많은 복잡한 모형은 단순한 모형에 비해 과적합되기 쉽다

간단한 예제를 통해 두 가지 케이스를 살펴보자.

## 7.1.1 More parameters (almost) always improve fit

과적합(Overfitting)은 모형이 샘플 데이터를 너무 과하게 학습할 때 발생한다. 

영장류 7가지의 평균 두뇌 크기와 중량 데이터를 살펴보자. 두 변수를 연결하는 가장 단순한 방법은 선형 관계를 가정하는 것이다. 

```r
library(rethinking)
library(tidyverse)

brain_mass_raw <- tibble(
    species = c("afarensis", "africanus", "habilis", "boisei", "rudolfensis", "ergaster", "sapiens"),
    brain = c(438, 452, 612, 521, 752, 871, 1350),
    mass = c(37.0, 35.5, 34.5, 41.5, 55.5, 61.0, 53.5)
)

# 변수를 표준화한다
# * brain 변수는 0을 기준점으로 삼기 위해 표준화하지 않는다
brain_mass <- brain_mass_raw %>% 
    mutate(mass_std = (mass - mean(mass)) / sd(mass),
            brain_std = brain / max(brain))

# 선형 모형을 구성한다
m_bm_01 <- quap(
    alist(
        brain_std ~ dnorm(mu, exp(log_sigma)),
        mu <- a + b * mass_std,
        # Prior for alpha : 말도 안되게 넓은 prior로 설정
        # * 가능한 값이 [0, 1]인데 89% 신용구간이 [-1, 2] 정도가 나옴
        a ~ dnorm(0.5, 1),
        # Prior for beta : flat prior
        b ~ dnorm(0, 10),
        log_sigma ~ dnorm(0, 1)
    ),
    data = brain_mass
)
```

그래프를 그리기 전에, R-squared에 집중해보자. 이 값은 전체 분산 중에서 모형을 통해 설명할 수 있는 비율을 의미한다. 
위에서 작성한 선형 모형의 R-squared 값을 구해보자.

```r
R2_is_bad <- function(data, quap_fit) {
    # 각 관측치에 대한 Posterior predictive distribution 을 구한다
    s <- sim(quap_fit, refresh = 0)
    # 모형의 잔차를 구한다
    r <- apply(s, 2, mean) - data$brain_std
    # 기존 var 함수는 frequentist 의 추정치이기 때문에 분모가 다르다
    # * 따라서 전통적인 방식으로 계산할 수 있도록 rethinking 라이브러리의 var2 함수를 쓴다
    # * var2 = mean(x^2) - mean(x)^2
    # R^2 = 1 - (Residual Variance / Outcome Variance)
    1 - rethinking::var2(r) / rethinking::var2(data$brain_std)
}

R2_is_bad(brain_mass, m_bm_01)
# [1] 0.4826186
```

이번에는 더 복잡한 모형들을 학습해 보자. 
파라미터의 수가 늘어날수록 R-squared가 증가하다가, 항이 6개가 되는 순간 R-squared의 최대치인 1이 되는 것을 볼 수 있다.

```r
#### 파라미터 2개로 학습 ####
m_bm_02 <- quap(
    alist(
        brain_std ~ dnorm(mu, exp(log_sigma)),
        mu <- a + b[1] * mass_std + b[2] * mass_std^2,
        a ~ dnorm(0.5, 1),
        b ~ dnorm(0, 10),
        log_sigma ~ dnorm(0, 1)
    ),
    data = brain_mass,
    start = list(b = rep(0, 2))
)

R2_is_bad(brain_mass, m_bm_02)
# [1] 0.5257412

#### 파라미터 6개로 학습 ####
m_bm_06 <- quap(
    alist(
        # 여기서 sd를 상수 0.001로 고정하지 않으면 모형이 작동하지 않는다
        brain_std ~ dnorm(mu, 0.001),
        mu <- a + b[1] * mass_std + b[2] * mass_std^2 +
            b[3] * mass_std^3 + b[4] * mass_std^4 +
            b[5] * mass_std^5 + b[6] * mass_std^6,
        a ~ dnorm(0.5, 1),
        b ~ dnorm(0, 10)
    ),
    data = brain_mass, 
    start = list(b = rep(0, 6))
)

R2_is_bad(brain_mass, m_bm_06)
# [1] 1
```

왜 6차 다항함수를 사용하면 완전히 적합해버리는 것일까? 왜냐하면 각 데이터 포인트 각각에 정확히 할당할 수 있는 파라미터 개수를 가지게 되었기 때문이다. 
6차 다항함수는 7개의 파라미터를 가지고, 우리는 7개의 데이터 포인트가 있다. 파라미터 수가 넉넉하면 모형은 데이터에 완전히 적합할 수 있다. 
하지만 그런 모형으로는 관측하지 못했던 부분에서 오히려 이상한 예측을 하게 된다.

## 7.1.2 Too few parameters hurts, too

모형이 과소적합(Underfitting)될 경우, 샘플에 있든 없든 부정확한 결과가 나온다. 너무 학습이 조금 되어서 샘플로부터 특징을 뽑아내는데 실패한 상태다. 또는 모형이 데이터에 너무 조금 반응하는 경우를 나타내기도 한다. 

# 7.2 Entropy and accuracy

과적합과 과소적합을 피해가려면 어떻게 해야 할까? **정규화(Regularization)** 또는 **정보 기준(Information Criteria)** 중 하나를 선택하더라도 
가장 먼저 해야 할 일은 모형을 평가하기 위한 기준을 정하는 것이다. 이 평가 기준을 **target** 이라고 해보자. 
이번에는 정보 이론이 어떻게 유용한 타겟을 제공하는지 살펴볼 것이다.

그렇지만 out-of-sample deviance를 구하는 것은 어렵다. 이를 위해서는 다음과 같은 작업들이 필요하다.

1. 완벽한 정확성과 얼마나 거리가 있는지 측정하는 척도를 세운다
2. 완벽한 정확성과의 상대적인 거리를 추정한 편차(Deviance)를 정의한다
3. 우리가 알아야 하는 것은 샘플을 벗어난 편차라는 것을 확인해야 한다

## 7.2.1 Firing the weatherperson

정확도는 평가 기준을 어떻게 정의하는지에 따라 달라진다. 평가 기준을 정의할 때는 크게 다음과 같은 요소를 고민해야 한다.

1. 틀렸을 때 얼마만큼의 비용이 발생하는가?
2. "정확도"의 맥락
    - 우리는 결합 확률(Joint Probability)을 측정해야 한다
    - 발생할 수 있는 사건들의 상대적인 비율을 측정하는 도구이기 때문이다
    - 모형이 실제 확률 분포와 같을 때 평균 확률 또는 결합 확률이 높아진다
    - 보통 결합 확률에 로그를 취해서 정확도의 척도로 사용하곤 한다 : **Log Scoring Rule**

## 7.2.2 Information and uncertainty

데이터의 로그 확률을 정확도의 기준으로 삼기로 했다. 그렇다면 완벽한 정확도와 얼마나 차이가 나는지 측정하는 것은 어떻게 해야 할까? 
이를 위해서는 예측이 목표로부터 얼마나 떨어져 있는지 거리를 측정할 수 있어야 한다. 여기에 어떤 "거리"를 사용해야 할까?

기본적인 컨셉은 다음과 같은 질문에서 시작한다. *결과를 하나 알게 된다면 불확실성이 얼마나 감소할까?* 이것을 바탕으로 정보를 다시 정의할 수 있다.

> 정보 (Information) : 결과를 알게 되었을 때 감소하는 불확실성의 정도

그리고 불확실성을 측정하기 위해 Information Entropy 를 사용한다. 다음과 같이 간단하게 정리할 수 있다. 
만약 비가 올 확률이 0.3, 맑을 확률이 0.7 이라면 `H(p) = -(0.3*log(0.3) + 0.7*log(0.7)) = 0.61` 이 된다.

```
H(p) = -E(log(p_i)) = - SUM_{i=1}^{n} p_i*log(p_i)
-> 확률분포의 불확실성은 사건의 로그 확률에 평균을 취하는 방식으로 구할 수 있다

* n : 가능한 사건의 수
* i : 각 사건
* p_i : 각 사건이 발생할 확률
```

엔트로피 만으로는 아직 무엇을 할 수 있는지 와닿지 않을 것이다. 엔트로피를 통해 정확도를 측정할 방법을 구해보자.

## 7.2.3 From entropy to accuracy

H를 통해 불확실성을 측정하는 방법을 알게 되었다. 정확하게는 목표를 맞히기가 얼마나 어려운지를 측정할 수 있게 되었다. 
그렇다면 모형이 목표로부터 얼마나 떨어져 있는지 측정하려면 어떻게 해야 할까? 발산(Divergence)을 통해 살펴보자.

> 발산 (Divergence) : 한 분포의 확률을 통해 다른 분포를 설명하고자 할 때 늘어나는 불확실성의 정도

이 방식을 제안한 사람의 이름을 따서 쿨백-라이블러 발산 (Kullback-Leibler Divergence) 또는 간단하게 KL Divergence 라고 한다.

실제 사건의 확률 분포가 `p1=0.3, p2=0.7` 이라고 해보자. 만약 우리가 확률이 `q1=0.25, q2=0.75` 일 것이라고 생각하고 있다면, 
q를 통해 p를 근사하면서 발생하는 불확실성이 얼마나 될까? 이에 대한 답은 H를 통해 계산할 수 있다.

```
D_KL(p, q) = SUM_{i}[ p_i * (log(p_i) - log(q_i)) ]
            = SUM_{i}[ p_i * log(p_i / q_i) ]

* Divergence는 target p와 모형 q 사이의 로그 확률의 차이로 평균을 구한 값을 말한다
* p=q 라면 D_KL(p, q) = 0 이다
```

발산(Divergence)을 통해 다양한 분포 중에서 목표와 가장 근사한 분포를 찾아낼 수 있다. 이를 통해 예측 모형의 정확도를 계산할 수 있다.

## 7.2.4 Estimating divergence

KL Divergence를 사용해서 모형을 비교하려면 목표로 하는 확률 분포(p)를 알아야 한다. 그 동안 예제에서는 p를 알고 있다고 가정했다. 
그런데 실제와 가장 비슷한 모형 q를 찾고자 할 때는 p를 직접 알 수 있는 방법이 없다. p를 알고 있다면 애초에 통계적 추론을 할 필요가 없을 것이다.

그런데 이런 문제를 피해갈 수 있는 방법이 있다. 우리가 하려는 것은 서로 다른 모형 후보 q와 r을 비교하는 것이다. 
이 경우에 p와 관련된 항이 대부분 소거된다. 따라서 p를 알지 못하더라도 q와 r이 얼마나 떨어져 있는지, 그리고 어떤 것이 목표와 더 가까운지 알 수 있다. 
이 계산을 위해서는 각 모형의 평균 로그 확률이 필요하다. 각 관측치에 대한 로그 확률을 더하기만 하면 `E[log(q_i)]` 의 근사치로 사용할 수 있다. 
하지만 그 수치만으로는 어떤 해석도 할 수 없다. 다른 모형의 값과 차이를 구했을 때만 각 모형이 목표 확률 분포인 p와 얼마나 떨어져 있는지 알고자 하는 목적으로 사용할 수 있다. 

실제로 사용할 때는 보통 모든 관측치에 대해 더해서 특정 모형에 대한 스코어를 구하는 경우가 많다. 
이런 스코어를 로그 확률 점수라고 하고, 서로 다른 모형의 예측 정확도를 구하기 위해 많이 사용한다.

```
S(q) = SUM_{i}[ log(q_i) ]

* 모형 q에 대한 Score값
```

베이지안 모형을 위해 스코어를 구하기 위해서는 모든 Posterior 분포를 활용해야 한다. 
실제 계산을 할 때는 몇 가지 미묘한 부분이 있기 때문에, rethinking 라이브러리에서는 lppd (Log Pointwise Predictive Density) 함수를 제공한다. 

```r
set.seed(123)
lppd(m_bm_01, n=1e4)
# [1]  0.6100859  0.6489015  0.5486945  0.6256770  0.4697209  0.4371187 -0.8548447
```

각각의 값은 관측치별 로그 확률 점수를 의미한다 (데이터에는 7개의 관측치가 있었다). 이 값을 모두 더하면 모형과 데이터에 대한 로그 확률 점수가 된다. 
이 값이 높을수록 더 높은 정확도를 가지고 있다고 볼 수 있다. Deviance(편차) 를 사용하는 경우도 있는데, lppd 에 -2를 곱한 값을 말한다. 
이 경우에는 값이 작을수록 정확한 모형이다.

## 7.2.5 Scoring the right data

로그 확률 점수는 목표 분포와 얼마나 떨어져있는지 거리를 재기 위한 가장 원칙적인 접근 방법이다. 
하지만 이 스코어는 R-squared와 동일한 문제가 있다. 모형이 복잡해지면 무조건 점수가 높아진다. 

```r
set.seed(123)
map_dbl(list(m_bm_01, m_bm_02, m_bm_06), ~ sum(lppd(.x)))
# [1]  2.495427  2.649391 39.481466
```

우리가 정말 알고자 하는 것은 새로운 데이터에서 나올 점수다. 어떻게 하면 새로운 데이터에서 점수를 높일 수 있을지 고민해보기 전에, 
샘플 안과 바깥 모두에서 점수를 구해보는 시뮬레이션을 해보자. 우리는 학습용 샘플에서 모형을 학습하고, 테스트용 샘플에서 결과를 측정할 것이다. 
이 사고 실험의 전체 프로세스를 정리해보면 다음과 같다.

1. 크기 N의 학습용 샘플이 있다고 가정한다
2. 학습용 샘플에서 모형을 작성하고 Posterior 분포를 구한다. 그리고 학습용 샘플에 대해 스코어를 구한다
    - 이 값을 `D_train` 이라고 한다
3. 다른 크기 N의 샘플(테스트용 샘플)에 대해 동일한 작업을 수행한다
4. 학습용 샘플을 통해 학습한 Posterior 분포를 평가용 샘플에 적용하여 스코어를 구한다
    - 이 값을 `D_test` 라고 한다

10000번 시뮬레이션 후에 파라미터 개수별 편차(Deviance, `-2 * lppd` , 따라서 작을 수록 높은 정확도를 나타낸다)를 비교해보자. 
학습용 샘플에서는 파라미터 수가 늘어날수록 편차가 작아지지만, 테스트용 샘플에서는 특정 파라미터 개수를 넘어가면 다시 편차가 커지기 시작한다. 

참고로 실제 데이터 생성 프로세스에서 테스트용 샘플 편차가 가장 작아질 것이라는 보장이 없다는 점을 주의해야 한다. 
편차(Deviance)는 모형의 정확도를 평가하기 위한 수단이지, 진실 여부를 판단하기 위한 값이 아니다.

# 7.3 Golem Taming : Regularization

과적합(Overfitting)은 학습에 사용한 샘플 데이터에 모형이 과도하게 반응하여 발생한다. 
Prior가 flat에 가까워지면 모형은 모든 파라미터에 동일한 가치를 부여한다. 따라서 이 경우 학습 데이터를 최대한 반영한 Posterior를 얻게 된다.

과적합을 막기 위한 방법 중 하나는 회의적인 Prior를 사용하는 것이다. 여기서 "회의적" 이라는 것은 샘플 데이터에서 배우는 속도를 늦추는 것을 말한다. 
회의적인 Prior 중에서 가장 일반적인 것은 정규화(Regularizing) Prior다. 

이전 장에서는 Prior Predictive 분포가 합리적인 결과를 낼 때까지 모형을 계속 수정해야 했다. 샘플 데이터의 수가 적을 때는 이러한 작업이 매우 도움이 된다. 

회의적인 Prior를 어느 정도의 강도로 설정할지는 데이터와 모형에 따라 다르다. 5개의 모형에 3가지 Prior를 각각 적용해서 1만번씩 시뮬레이션 해보자. 
샘플 사이즈 N=20 일 때는 학습 데이터에서 파라미터 수가 늘어날수록 Deviance가 계속 줄어드는 것을 볼 수 있다. 
특히 가장 회의적인 Prior를 썼을 때는 편차가 가장 커진다. 그런데 테스트용 데이터에서는 가장 좋은 성능을 보인다. 
**Prior가 점점 더 회의적일수록 모형이 복잡해지면서 발생하는 악영향을 덜 받게된다. 샘플 사이즈가 N=100 일 때는 Prior의 영향이 전체적으로 줄어든다.** 
증거가 많아졌기 때문이다. Prior가 도움을 주긴 하지만, 데이터가 Prior을 압도할 정도로 풍부한 정보를 얻을 수 있다. 
Prior를 정규화하는 것은 과적합을 억제하는데 도움이 되지만, 과할 경우에는 모형이 데이터로부터 학습하는 것을 방해할 수 있다.

# 7.4 Predicting predictive accuracy

앞서 논의했던 과소적합과 과적합에 대한 내용은 동일한 접근 방식을 취하고 있다. 모형을 샘플 바깥의 데이터에서 평가해야 한다는 것이다. 
하지만 말 그대로 샘플 바깥의 데이터이니, 실제로 우리가 사용할 수 있는 값은 아니다. 
이러한 경우에 모형을 평가하기 위해 보통 두 가지 접근 방법을 사용한다. 

1. Cross Validation
2. Information Criteria

이 방법들은 새로운 데이터에 모형을 적용했을 때 평균적으로 얼마나 성능을 낼 수 있을지 추정한다. 수학적인 디테일은 많이 다르지만, 상당히 비슷한 결과를 낸다.

## 7.4.1 Cross validation

이 방식은 일부 관측치를 제외하고 학습한 뒤에, 미리 빼둔 관측치를 통해 모형을 평가한다. 
각각의 관측치 덩어리(fold 라고 한다)에 대해 반복하면 평균적인 성능을 구할 수 있다. 
관측치 덩어리(fold)는 가장 적을 경우 2개, 가장 많을 경우 관측치 개수만큼 존재할 수 있다. 
`quap` 으로 학습한 모형에 대해 cv를 적용하려면 rethinking 라이브러리의 `cv_quap` 함수를 사용해보자. 

fold 개수는 몇 개로 결정하는 것이 제일 좋을까? 많은 연구에서 fold 개수는 적당한 것이 제일 좋다고 이야기한다. 
하지만 시뮬레이션을 바탕으로 한 연구는 적당한 값을 찾기가 어렵기 때문에, 최대한의 fold 개수를 설정하는 경우가 흔하다. 
다시 말해 각 fold는 관측치 한 개씩만을 빼둔다. 이러한 방식을 **Leave-One-Out Cross Validation** ( 줄여서 **LOOCV** )라고 한다.

LOOCV의 문제는 관측치가 많아질수록 계산하는데 시간이 오래 걸린다는 점이다. 그래서 모형을 매번 계산하지 않고 CV 점수를 근사하는 방법이 있다. 
한 가지 방법은 각 관측치가 Posterior 분포에서 차지하는 **중요도(Importance)** 를 계산하는 것이다. 
예상하지 못했던 관측치가 예상할 수 있는 관측치보다 더 중요하다는 생각을 바탕으로 한다. 
이 중요도를 가중치(weight)로 두고 모형이 샘플 데이터 바깥에 적용되었을 때 나타났을 성능을 추정한다. 

이 전략을 통해 CV 점수를 근사적으로 계산할 수 있는 방법을 얻을 수 있다. 
이 근사법을 **Pareto-Smoothed Importance Sampling Cross-Validation** ( 줄여서 **PSIS** ) 이라고 한다. 
PSIS의 좋은 점 중 하나는 모형의 신뢰성에 대한 정보를 제공한다는 것이다. 자세한 내용은 이 장의 남은 내용들과 책 전반에 걸친 예제를 통해 살펴볼 수 있을 것이다.

## 7.4.2 Information Criteria

두 번째 방식인 Information Criteria 는 샘플 바깥과의 상대적인 KL divergence 을 추정하는 방식이다. 
일반적인 회귀 모형에 flat prior를 사용할 경우, 예상되는 과적합 페널티는 파라미터 개수의 약 2배가 된다. 
이러한 현상이 Information Criteria의 이론적 바탕이 된다. 이 중에서 가장 많이 알려진 것은 **AIC(Akaike Information Criterion)** 다.

```
AIC = D_train + 2p 
    = -2 * lppd + 2p

* p : Posterior 분포의 자유로운 파라미터 개수
```

AIC는 다음과 같은 조건이 성립할 때만 신뢰할 수 있는 근사치다.

1. flat한 prior를 사용하거나 데이터로 인한 영향이 압도적이다
2. Posterior가 근사적으로 다변량 정규분포를 따른다
3. 샘플 사이즈 N이 파라미터 개수 k 보다 훨씬 크다

flat prior는 보통 최적의 prior가 아니기 때문에, 훨씬 일반화된 지표가 필요하다. 
따라서 여기에서는 **WAIC(Widely Applicable Information Criterion)** 를 사용한다. 
WAIC는 posterior의 형태와 상관없이 사용할 수 있다. 일반화를 시키다보니 어쩔 수 없이 수식은 많이 복잡해지게 되었다. 
하지만 크게 두 부분으로 나눌 수 있고, 둘 다 Posterior 분포에서 샘플링을 통해 구할 수 있다. 

```
WAIC = -2 * (lppd - Penalty Term)

* Penalty Term : 각 관측치의 로그 확률에서 분산을 구하고, 이 값을 전부 합한 총 페널티 점수
                    쉽게 생각하면 "과적합 페널티 점수" 라고 볼 수 있다
```

PSIS 처럼 WAIC 도 관측치 각각에 대해 계산해야 한다. 어떤 관측치들은 예측이 더 어려울 수도 있고, 불확실성의 크기가 서로 다를 수 있기 때문에 이런 특성은 유용하게 사용할 수 있다. 
유용한 활용법 중 하나는 샘플 바깥의 편차를 예측할 때 그 표준 오차를 구하는 것이다.

Cross Validation 처럼 WAIC 도 데이터를 독립적인 관측치 덩어리로 나누는 것을 허용한다. 그래서 가끔은 정의하기 어려워지는 경우가 있다. 
예를 들면 시계열 데이터의 경우 이전의 관측치들이 이후에 영향을 미치기 때문에 데이터가 서로 독립이라고 보기 어렵다. 
값을 계산할 수는 있지만 그 값이 의미하는 바가 명확하지 않다.

R 코드를 통해 WAIC 계산 과정을 살펴보자.

```r
# 간단한 회귀 모형을 만든다
data(cars)
model <- quap(
    alist(
        dist ~ dnorm(mu, sigma),
        mu <- a + b * speed,
        a ~ dnorm(0, 100),
        b ~ dnorm(0, 10),
        sigma ~ dexp(1)
    ),
    data = cars
)

get_waic <- function(data, model, n_samples = 1000, seed = 123) {
    set.seed(seed)
    n_cases <- nrow(data)
    # 모형에서 Posterior의 샘플을 추출한다
    post <- extract.samples(model, n = n_samples)
    # 각 샘플 s에서 관측치별 log likelihood를 구한다
    logprob <- seq_len(n_samples) %>%
        sapply(function(s) {
            mu <- post$a[s] + post$b[s] * data$speed
            dnorm(data$dist, mu, post$sigma[s], log = TRUE)
        })
    # lppd를 구한다
    # * 각 관측치에 대해 log likelihood 값을 평균한다
    # * 정확도를 위해 log 스케일로 변한하여 계산한다
    lppd <- seq_len(n_cases) %>%
        sapply(function(i) rethinking::log_sum_exp(logprob[i, ]) - log(n_samples)) %>%
        sum()
    # 페널티 항을 계산한다
    # * 각 관측치에 대해 샘플 간의 분산을 구한다
    penalty_waic <- seq_len(n_cases) %>%
        sapply(function(i) var(logprob[i, ])) %>% 
        sum()

    # WAIC 값을 반환한다
    -2 * (lppd - penalty_waic)
}

get_waic(cars, model)
# [1] 422.4667
```

## 7.4.3 Comparing CV, PSIS, and WAIC

CV, PSIS, WAIC가 과적합의 위험을 잘 파악할 수 있을까? 2가지 Prior, 2가지 샘플 크기에 대해 파라미터 수가 1~5개인 모형을 각각 1000번씩 시뮬레이션하여 비교해보자. 

- 세 방식 모두 샘플 바깥의 편차를 잘 예측한다
- WAIC 는 편차를 조금 더 잘 예측한다
- CV, PSIS는 샘플 수가 적을 때는 오차가 커지지만 샘플 수가 많아지면서 차이가 줄어든다

PSIS와 WAIC 모두 일반적인 선형 회귀 모형에서는 비슷하게 동작한다. 하지만 Posterior 분포가 정규분포가 아닐 때는 큰 차이가 발생한다. 
CV나 PSIS는 KL Divergence의 추정치로 사용할 경우에 분산이 커지는 경향이 있다. 
따라서 이러한 경우에는 WAIC가 더 나을 수 있지만, 실제로 WAIC를 사용해서 얻는 이득이 크지 않다. 또한 모형의 순위만 매기면 되는 상황에서는 PSIS가 WAIC보다 못할 이유가 없다. 
그리고 PSIS의 각 관측치에서 구하는 k값을 통해 어느 데이터에서 PSIS 값이 불안정한지 확인할 수 있다.

# 7.5 Using cross-validation and information criteria

다시 원래의 문제로 돌아가보자. 데이터를 설명할 수 있는 모형 후보들이 여러 개 있을 때, 각 모형의 정확도를 어떻게 비교할 수 있을까? 
모형이 샘플 데이터에 얼마나 적합한지 보거나, information divergence 를 사용하게 되면 복잡한 모형을 선호하는 경향이 있다. 
따라서 우리는 샘플 바깥에 모형을 적용했을 때를 추정해야 한다. 2가지 중요한 포인트가 있다.

1. Flat prior 를 사용하면 예측 결과가 좋지 않다. Prior를 정규화 하면 샘플에 대한 적합도는 떨어지지만, 예측 정확도를 높일 수 있다.
2. CV, PSIS, WAIC 를 사용하면 예측 정확도를 잘 추정할 수 있으며, Prior 정규화와 서로 상호 보완적인 역할을 한다.

이제 실제로 분석을 해보자. Prior 정규화와 CV/PSIS/WAIC 같은 지표를 어떻게 사용하면 좋을까? 
흔히 **모형 선택(Model Selection)** , 다시 말해 점수를 바탕으로 하나의 모형만 선택하는 방식을 사용한다. 하지만 이런 방식은 사용하지 않는 것이 좋다. 

1. 모형 간의 상대적인 정확도 차이 정보를 버리게 되기 때문이다
    - 파라미터에 대한 신뢰도
    - 모형에 대한 신뢰도
2. 정확도가 높은 모형이라고 해서 인과 관계를 잘 설명할 수 있는 것은 아니기 때문이다 

따라서 우리는 모형을 선택하는 것이 아니라, 모형을 비교하는데 중점을 둘 것이다. 

## 7.5.1 Model mis-selection

인과 관계를 추론하는 것과 예측을 수행하는 것은 전혀 다른 작업이라는 것을 명심해야 한다. CV, PSIS, WAIC 는 좋은 예측을 하는 모형을 찾기 위한 도구다. 
높은 예측 정확도를 가지는 모형을 선택하다보면 교란된 모형을 선택하게 될 가능성이 높아진다. 
Backdoor path는 데이터에 숨겨진 상관관계 정보를 제공하기 때문에 우리가 개입하지 않았을 경우에 발생할 미래를 잘 예상할 수 있다. 
**하지만 우리가 정의한 인과관계라는 것은 그런게 아니다. 우리가 개입했을 때 어떻게 변할지 예상할 수 있어야 한다.** 
따라서 PSIS나 WAIC 점수가 높다고 해서 인과 관계를 잘 설명할 수 있을 거라고 기대하면 안된다.

WAIC를 계산하기 위해 rethinking 라이브러리에서 제공하는 함수를 사용해보자.

```r
#### (1) 특정 모형의 WAIC를 계산한다 ####
set.seed(123)
rethinking::WAIC(m_bm_06)
#        WAIC     lppd  penalty   std_err
# 1 -72.42255 39.50894 3.297659 0.4268529

# WAIC    : out-of-sample deviance 에 대한 추정값
# lppd    : Log Pointwise Predictive Density
# penalty : pWAIC (the effective number of parameters)
# std_err : WAIC 값의 standard error

# (2) 여러 모형을 비교한다
# - 각 모형의 WAIC를 비교한다
rethinking::compare(m_bm_01, m_bm_02, m_bm_06)
#          WAIC   SE dWAIC  dSE pWAIC weight
# m_bm_06 -71.8 0.34   0.0   NA   3.6      1
# m_bm_01   5.5 8.32  77.4 8.91   5.2      0
# m_bm_02   8.4 7.95  80.2 8.53   6.7      0

# WAIC   : WAIC 값. 작을수록 좋다 (기본적으로는 WAIC 오름차순으로 모형이 정렬된다)
# SE     : WAIC의 표준 편차. 
# dWAIC  : 가장 낮은 WAIC를 가지는 모형과 얼마나 차이가 나는지 비교
# dSE    : 모형간 WAIC 차이에 대한 표준 편차
# pWAIC  : penalty term
# weight : 각 모형에 대한 상대적인 지지도 (전부 합하면 1이 된다)

# - WAIC 대신 PSIS를 사용해 비교할 수 있다
rethinking::compare(m_bm_01, m_bm_02, m_bm_06, func = PSIS)
#          PSIS    SE dPSIS   dSE pPSIS weight
# m_bm_06 -66.2  1.94   0.0    NA   6.4      1
# m_bm_01  10.2 12.50  76.4 12.34   7.5      0
# m_bm_02  26.1 12.98  92.3 12.56  15.6      0
```

## 7.5.2 Something about *Cebus*

WAIC 와 LOOIS 의 장점 중 하나는 모형이 어떤 관측치를 어려워하는지 알 수 있다는 것이다. 그리고 모형을 비교할 때 어떤 관측치들을 쉽거나 어려워 하는지 여부는 데이터가 생성되는 프로세스를 파악하는 단서가 된다. 이 장의 마지막 예제에서는 새로운 데이터를 통해 WAIC와 인과관계 추론이 실제 데이터에 어떻게 적용되는지 살펴보자.

```r
library(rethinking)
library(tidyverse)

# rethinking 라이브러리의 영장류 데이터를 메모리로 가져온다
data('Primates301', package = 'rethinking')

# 다음과 같이 세 가지 변수를 사용한다
# - longevity    : 수명 (개월)
# - body mass    : 몸무게 (그램)
# - brain volume : 뇌의 부피 (세제곱센티미터)

# 데이터 전처리에 사용할 표준화 함수를 만든다
log_standardize = function(x, base = exp(1), ...) {
  scale(log(x, base), ...)[,1]
}

# 데이터를 전처리한다
# - 모델링에 사용할 변수를 표준화시킨다
# - L, B, M 변수 모두 존재하는 경우를 추출한다
primates <- Primates301 %>% 
  as_tibble() %>% 
  mutate(log_L = log_standardize(longevity),
         log_B = log_standardize(brain),
         log_M = log_standardize(body)) %>% 
  filter(complete.cases(log_L, log_B, log_M))
```

변수 간의 관계를 DAG로 표현하면 다음과 같다.

```
M -> B -> L
M    ->   L
M <- U -> L
(M: 몸의 질량, B: 뇌의 부피, L: 수명, U: 관측되지 않은 교란변수)

질문 : 몸의 질량과 뇌의 부피가 수명에 영향을 미칠까?
* 몸이 클수록 천적이 줄어 수명이 늘어날 것이다
* 뇌의 부피는 몸의 질량과 상관관계가 높다는 것이 알려져 있다
```

모형을 작성하고 Posterior를 근사해보자. 

```r
# 모형을 학습한다
# * 원래의 스케일에서는 선형 관계이기 어렵지만, 로그 스케일이라면 가능하다
m_primates_01 <- quap(
  alist(
    log_L ~ dnorm(mu, sigma),
    mu <- a + bM * log_M + bB * log_B,
    a ~ dnorm(0, 0.1),
    bM ~ dnorm(0, 0.5),
    bB ~ dnorm(0, 0.5),
    sigma ~ dexp(1)
  ),
  data = primates
)
```

결과를 살펴보기 전에 간단한 모형을 2개 더 학습해보자.

```r
m_primates_02 <- quap(
  alist(
    log_L ~ dnorm(mu, sigma),
    mu <- a + bB * log_B,
    a ~ dnorm(0, 0.1),
    bB ~ dnorm(0, 0.5),
    sigma ~ dexp(1)
  ),
  data = primates
)

m_primates_03 <- quap(
  alist(
    log_L ~ dnorm(mu, sigma),
    mu <- a + bM * log_M,
    a ~ dnorm(0, 0.1),
    bM ~ dnorm(0, 0.5),
    sigma ~ dexp(1)
  ),
  data = primates
)
```

각 모형의 WAIC를 비교해보자.

- B, M을 모두 사용한 1번 모형과 B만 사용한 2번 모형은 WAIC 만으로 우열을 가리기 어렵다
- 1,2번 모형 모두 M만 사용한 3번 모형과는 상당히 큰 차이가 난다

이러한 결과를 보면 뇌의 부피가 몸의 크기보다 수명과 더 연관성이 높아 보일 것이다.

```r
set.seed(123)
compare(m_primates_01, m_primates_02, m_primates_03)
#                WAIC    SE dWAIC  dSE pWAIC weight
# m_primates_01 216.1 14.73   0.0   NA   3.4   0.56
# m_primates_02 216.5 14.87   0.5 1.54   2.6   0.44
# m_primates_03 229.7 16.48  13.6 7.15   2.6   0.00

# 그래프를 통해 살펴보자
set.seed(123)
plot(compare(m_primates_01, m_primates_02, m_primates_03))
```

![](fig/ch7_primate_example_01.png)

하지만 이번에는 Posterior 분포를 살펴보자. 두 변수를 모두 사용한 1번 모형에서 bM과 bB의 Posterior 분포가 더 넓은 것을 확인할 수 있다. 
6장의 다중공선성 문제처럼, `log_B` 와 `log_M` 변수는 서로 높은 상관관계를 가진다고 볼 수 있다. 

```r
plot(coeftab(m_primates_01, m_primates_02, m_primates_03), pars = c('bM', 'bB'))
```

![](fig/ch7_primate_example_02.png)

```r
# 두 변수의 상관관게를 계산해보자
primates %>% with(cor(log_B, log_M))
# [1] 0.9796272
```

두 변수가 서로 높은 상관관계를 가진다는 것은 사실 잘 알려져있다. 하지만 두 변수는 서로 대체재가 아니다. 
2,3번 모형을 보면 M 변수를 추가한 뒤에도 B 변수는 여전히 높은 (하지만 음의 방향으로 바뀐) 상관관계를 가지고 있다. 

- 영장류의 데이터에서만 일어나는 상황일까?
    - 동일한 현상이 포유류 데이터에서도 발생한다
- 데이터를 추가적으로 처리하면 나아지지 않을까?
    - 몸무게 1그램당 뇌의 부피 (BPG) 변수를 만들어서 사용해도 비슷한 현상이 발생한다
- 수명과 뇌의 부피 사이의 선형 모형에서 잔차를 추출해서 분석해보면 어떨까?
    - 잔차는 관측된 값이 아니기 때문에 분포를 갖는다
    - 따라서 잔차를 데이터처럼 생각하게 되면 그 효과를 과대 평가하게 될 우려가 있다

이 주제에 대한 출판된 논의는 명확하지 않기 때문에 여기서 결론을 내진 않을 것이다. 하지만 모형이 데이터를 어떻게 바라보고 있는지를 WAIC로 한 번 살펴보자. 

- x축은 1,2번 모형의 포인트별 WAIC 차이를 의미한다
    - x축이 0보다 작을 경우 1번 모형(B + M)에서 더 정확하게 설명했다
    - x축이 0보다 클 경우 2번 모형(B)에서 더 정확하게 설명했다

```r
waic_m_01 <- WAIC(m_primates_01, pointwise = TRUE)
waic_m_02 <- WAIC(m_primates_02, pointwise = TRUE)

primates %>% 
  mutate(diff = waic_m_01$WAIC - waic_m_02$WAIC,
         name = as.character(name)) %>% 
  ggplot(aes(x = diff, y = log_L)) +
  geom_point(aes(size = log_B - log_M), color = '#1D4E89', alpha = 0.5) +
  geom_vline(xintercept = 0, linetype = 2, color = '#999999') +
  geom_hline(yintercept = 0, linetype = 2, color = '#999999') +
  geom_text(aes(label = if_else(str_detect(name, '^Cebus'), name, '')), hjust = 0.3, size = 3)+
  labs(x = 'pointwise difference in WAIC', 
       y = 'log longevity (std)', 
       size = 'Diff in B and M') +
  theme_minimal()
```

![](fig/ch7_primate_example_03.png)

두 모형은 데이터에 대해 상당히 다른 특성을 보인다. 특히 1번 모형은 *Cebus* 관련하여 특이한 점이 있다. 
현재 우리가 해결하지 못한 점은 다음과 같다. **큰 동물이 오래 산다는 이론적, 경험적 증거가 있는데도 왜 bM Posterior는 음의 상관관계를 보이는 걸까?** 
위 부분까지 포함하여 설명할 수 있는 모형을 만들 수는 없을까?

영장류가 진화하는 과정에 무슨 일이 발생했는지는 잘 모르겠다. 하지만 적어도 모형에서 이런 문제가 발생하는 이유는 **Collider Bias** 때문이 아닐까 생각한다. 
우리가 맨 처음 그렸던 DAG에서는 뇌의 부피가 수명에 영향을 주었다. 하지만 그 반대라면 어떨까?

```
# 원래 DAG
M -> B -> L
M    ->   L
M <- U -> L

# B와 L 사이의 관계를 바꾼 DAG -> Collider가 생성되었다
M -> B <- L
M    ->   L
M <- U -> L
```

B와 L의 순서를 바꾸었더니 Collider가 생겼다. 이런 의심을 하게 된 이유는 데이터에서 나타난 패턴 때문이다. 
Collider의 주요 증상 중 하나는 Collider 변수(여기서는 B)를 추가했을 때 M→L 관계가 왜곡된다는 점이다. 

두 번째 DAG대로 M과 L을 통해 B를 설명하는 모형을 만들어보자.

```r
m_primates_04 <- quap(
  alist(
    log_B ~ dnorm(mu, sigma),
    mu <- a + bM * log_M + bL * log_L,
    a ~ dnorm(0, 0.1),
    bM ~ dnorm(0, 0.5),
    bL ~ dnorm(0, 0.5),
    sigma ~ dexp(1)
  ),
  data = primates
)

precis(m_primates_04)
#        mean   sd  5.5% 94.5%
# a     -0.05 0.02 -0.07 -0.02
# bM     0.94 0.03  0.90  0.98
# bL     0.12 0.03  0.07  0.16
# sigma  0.19 0.01  0.17  0.21
```
