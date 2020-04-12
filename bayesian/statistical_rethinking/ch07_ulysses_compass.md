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
