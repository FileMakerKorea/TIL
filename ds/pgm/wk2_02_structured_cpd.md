# Sructured CPDs

## Overview

지금까지 우리는 전체적인 분포의 구조에 대해서만 살펴보았다. 
전체 그래프를 factorize 해서 각각의 변수에 대응되는 factor들의 곱으로 표현할 수 있다는 것을 확인했다. 
하지만 그와 다른 유형의 문제들도 존재하며, 실제 문제를 해결하는데 중요하게 사용된다.

### Tabular Representations

* 확률 분포를 테이블의 형태로 표현할 수 있다
* G에 대한 확률분포를 표로 나타내보면 다음과 같다
    * row에는 G의 부모노드들이 가질 수 있는 모든 값의 조합을 나열한다
    * col에는 G가 가질 수 있는 값들을 나열한다
    * 각각의 칸에는 그에 맞는 확률값을 나타낸다
* binary 형태인 부모 변수가 k개 있다면 2^k 개의 행(변수들의 조합)이 필요하다
* 따라서 현실의 많은 문제들은 테이블 형태로 표현하는 것이 적합하지 않다

### General CPD

* 다행스럽게도 베이지안 네트워크에서는 모든 조건들에 대한 표를 작성할 필요가 없다
* 대신에 다음과 같은 확률분포를 정의해야 한다
    * `CPD P(X | y1, ... , yk)`
    * => y1, ... , yk 각각의 조건에 대한 X의 분포를 나타내는 CPD
* Factor를 구성하기 위해 어떤 함수를 사용해도 된다
    * y1, ... , yk 에 대한 모든 값의 합이 1이 되는 함수면 된다

### Many Models

* 다음과 같이 다양한 형태로 CPD를 정의하여 사용한다
    * Deterministic CPDs
    * Tree-structured CPDs
    * Logistic CPDs & Generalizations
    * Noisy OR / AND
    * Linear Gaussians & Generalizations

### Context-Specific Independence

* Context-Specific Independence 는 독립의 한 가지 종류이다
    * `P ㅑ (X ㅗc Y | Z, c)`
        * c는 C 변수에 특정한 값을 지정했다는 것을 의미한다
    * 변수 C를 특정한 값으로 조건을 부여했을 때 독립인 경우
* 다음과 같이 정의할 수 있다
    * `P(X, Y | Z, c) = P(X | Z, c) P(Y | Z, c)`
    * `P(X | Y, Z, c) = P(X | Z, c)`
    * `P(Y | X, Z, c) = P(Y | Z, c)`
* Y -> X 이고 X값은 y1과 y2의 OR 연산을 통해 결정된다고 할 때, context-specific 독립이 성립하는 것은 언제일까??
    * y2 = FALSE 라면 X는 y1과 같다 => 독립이 아니다
    * y2 = TRUE 라면 X는 y1과 상관없이 TRUE => 독립이다
    * X = FALSE 라면 y1, y2 모두 FALSE 여야 한다 => 독립이다
        * 각각의 y값들이 서로의 영향을 받아 결정된 것이 아니기 때문에 독립이라고 볼 수 있다
    * X = TRUE 라면 독립이 아니다
        * y1과 y2 중에 어느 쪽이 TRUE 일지 알 수 없다


## Tree-Structured CPDs

앞에서 테이블 형태의 CPD에는 부모변수의 개수가 기하급수적으로 증가한다는 문제점이 있다는 것을 살펴보았다. 구조화된 CPD 중에서 유용하게 사용할 수 있는 것은 부모-자식 관계를 가지고 CPD를 구성하는 것이다. 그 중에서 트리 구조 CPD에 대해서 살펴보자.

### Tree CPD

* 어떤 학생이 직장을 구하고 있다고 생각해보자
* 취업할 수 있는 가능성은 세 가지 변수에 의존하고 있다
    * Letter : 추천서
    * Apply : 지원 여부
    * SAT : SAT 점수
* CPD를 트리 구조를 통해 표현할 수 있다
    * 지원서를 제출하지 않았다면 어떻게 될까?
        * 지원서를 넣지 않더라도 취업할 수 있는 확률이 0은 아니겠지만, 이 경우에는 SAT 점수나 추천서와는 연관이 없을 것이다
    * 지원서를 제출했다면 어떻게 될까?
        * 여기서는 채용담당자가 SAT 점수를 우선시한다고 가정했다 (추천서는 그다지 믿지 않는다)
        * 그렇다면 먼저 SAT 점수부터 볼 것이다
        * SAT 점수가 높다면 굳이 추천서까지 보지 않더라도 합격할 확률이 크게 올라간다
        * SAT 점수가 높지 않다면 추천서를 보고 결정을 하게 될 것이다
* 원래라면 J 변수에 대해 8가지 서로 다른 확률분포를 구해야 했겠지만, 트리 구조를 통해 4가지로 표현할 수 있었다
    * 연관된 변수들의 CPD만 구한다


* 위와 같은 CPD 구조에서 찾아낼 수 있는 context-specific independencies 는 어떤 것들이 있을까??
    * (O) `(J ㅗc L | a1, s1)`
        * a1과 s1을 가정하면 채용담당자가 추천서를 볼 일도 없다
        * 따라서 이러한 문맥에서는 J가 L과 독립이다
    * (X) `(J ㅗc L | a1)`
        * 두 가지 시나리오가 있다
        * S = s1
        * S = s0 : 이 경우에는 추천서를 보고 결정한다. 따라서 독립이 아니다
    * (O) `(J ㅗc L, S | a0)`
        * a0 인 경우에는 L, S 모두 의존성이 없다
    * (O) `(J ㅗc L | s1, A)`
        * context-specific / non-context-specific independency 가 섞인 케이스다
        * S = s1 일 때, 그리고 A의 모든 값에 대해서 독립인지 확인해야 한다
        * 두 가지 statement로 나누어야 한다
            * `(J ㅗc L | s1, a1)` : 이건 위에서 확인했다 (첫 번째 케이스)
            * `(J ㅗc L | s1, a0)`
                * 세 번째 케이스(`J ㅗc L, S | a0`)의 특수한 경우이다
                * 따라서 이 경우에도 독립이다

### Tree CPD 2

* 2가지 추천서가 있고, 어떤 추천서를 제출할지 선택할 수 있다고 해보자
    * 취업이 결정될 확률은 제출한 추천서의 질에 의해 결정될 것이다
    * 어떤 추천서를 제출할지 결정하는 Choice 변수에 따라 CPD가 변한다
    * `C = c1` 인 경우에는 L1 변수만, `C = c2` 인 경우에는 L2 변수에만 의존한다
* L1과 L2는 J와 C가 주어져 있을 경우 독립이다 `(L1 ㅗ L2 | J, C)`
    * Influence Flow를 살펴보면, Choice의 영향을 받은 Job 변수가 Letter1과 Letter2 사이에서 v-structure를 구성한다
* 조금 더 상세한 케이스에 대해 살펴보자
    * `(L1 ㅗc L2 | J, c1)`
        * C = c1 의 경우 더 이상 L2와는 의존성이 생기지 않는다 ( 담당자가 추천서를 받지 못한다 )
    * `(L1 ㅗc L2 | J, c2)`
        * 여기도 마찬가지
    * 두 경우 모두 active trail이 사라진다. 따라서 독립이라는 것을 알 수 있다

### Multiplexer CPD

위에서 살펴본 것과 같은 상황을 조금 더 일반화시키면, 이러한 구조를 Multiplexer CPD 라고 한다

* 다음과 같은 경우를 Multiplexer CPD 라고 한다
    * Z1, ... , Zk 의 확률 변수를 가지고 있다
    * 변수 Y는 Zi 중 하나의 사본이다
    * 변수 A는 multiplexer 라고 하는 선택 변수이다. {1, ... , k} 중에서 하나를 선택한다
    * Zi가 주어져있고 A가 결정되어 있다면 Y도 결정된다 (deterministic dependencies)
* selector A와 {Z1, ... , Zk} 가 주어져 있을 때, Y의 CPD는 어떻게 될까?
    * `P(Y | A, Z1, ... , Zk)`
        * If Y = Za : 1
        * otherwise : 0

### Microsoft Troubleshooters

* MS의 프린터를 위한 문제해결 도구를 살펴보자
* MS 운영체제의 문제해결 도구들은 베이지안 네트워크를 기반으로 작성되어 있다
    * 프린터가 출력을 수행하고 있는지 여부를 노드로 나타낸다
    * 많은 변수와 의존관계가 있지만, 프린터의 입력이 어디서 오는지에 따라 영향을 받는다
        * Local Transport / Network Transport
        * 입력의 위치에 따라 각기 다른 종류의 오류가 존재한다
    * 트리 구조를 사용함으로써 파라미터 개수를 145개에서 55개로 줄일 수 있었다

### Summary

* 트리 구조를 통해 context에 따른 의존관계를 포함하면서 CPD를 간결하게 표현할 수 있다
* 다양한 분야에서 활용되고 있다


## Independence of Causal Influence

지금까지 변수와 그 부모 변수에 대한 로컬 종속성 구조를 포함하는 CPD 표현 방법에 대해 살펴보았다. 그리고 그 중에서도 각 context마다 다른 변수 변수에 의존할 수 있게 해주는 트리 CPD의 경우에 대해서 이야기했다. 하지만 tree CPD 로는 설명할 수 없는 경우가 있다. 감기 예제를 살펴보면 하나의 특정한 context에 의존하지 않는 경우가 있다. 여러 가지 요인이 존재하고, 모든 요인에 의존한다. 모든 요인들이 감기가 걸리는 확률에 영향을 미친다. 

### Noisy OR CPD

* 그러한 형태의 상호작용을 파악하기 위한 모형 중 하나가 바로 Noisy OR CPD 이다
    * {X1, ... , Xk} 와 Y 변수 사이에 intervening variable {Z1, ... , Zk} 가 존재한다
* 다시 감기 예제를 살펴보자
    * Y는 Cough 변수, {X1, ... , Xk} 는 감기의 원인을 나타낸다
    * 각각의 질병을 여기서는 시끄러운 송신기라고 생각해보자
    * 질병을 가지고 있다면 (X1 = TRUE)
    * Y는 하나라도 TRUE가 있다면 TRUE 값을 가진다 (질병의 원인이 하나라도 있다면 감기에 걸린다)
        * Y는 부모 변수인 {Z0, ... , Zk}에 대해서 deterministic한 변수이다
    * P(Z0 = 1) = lambda0
        * Z0는 leak probability 라고 한다
        * Y가 저절로 TRUE가 될 확률을 의미한다
    * P(Zi = 1 | Xi)
        * = 0 , if Xi = 0
        * = `lambda_i` , if Xi = 1 (`lambda_i` 는 0 또는 1 값을 가진다)
    * P(Y = 0 | X1, ... , Xk) 는 모든 노드가 Y을 TRUE로 만드는데 실패했을 확률을 나타낸다
        * Z0에 의해 자연적으로 발생하지 않아야 한다
        * X1부터 Xk의 모든 경우에 TRUE가 되지 않아야 한다

### Independence of Causal Influence

앞에서 살펴본 모형을 일반화시키면 Causal Influence의 독립성에 대해 정의할 수 있다.

* 원인에 해당하는 다양한 변수가 존재하고, 각각의 변수가 독립적으로 영향을 미친다
* 서로 다른 원인 변수들 간에는 상호작용이 존재하지 않는다
* 각 원인 변수들의 영향이 변수 Z에서 종합되고, 합쳐진 효과를 바탕으로 Y의 값이 결정된다

### Sigmoid CPD

* 위에서 살펴본 것과 같은 구조를 가지는 사례가 바로 Sigmoid CPD이다
    * 이산변수 Xi 마다 연속형 변수 Wi가 존재하고, Zi = Wi * Xi 이다
    * Wi는 각 edge에 대한 파라미터이다
        * Wi = 0 이면 Xi는 아무런 영향이 없다는 것을 의미한다
        * Wi > 0 이면 Y가 참이 되도록 유도한다
        * Wi < 0 이면 Y가 참이 되지 않지 않도록 유도한다
    * 모든 영향력과 bias term을 합쳐서 변수 Z를 구한다
        * Z = W0 + SUM(Wi * Xi)
        * 여기서 W0 는 bias term 이라고 한다
    * 이제 변수 Y에 대한 확률로 변환한다
        * P(y1 | X1, ... , Xk) = sigmoid(Z)
        * Sigmoid Function
            * sigmoid(z) = e^z / (z + e^z)
            * 연속형 변수 Z를 받아서 [0, 1] 구간의 값으로 반환한다

### Behavior of Sigmoid CPD

* Sigmoid CPD가 다양한 파라미터에 따라 어떻게 변하는지 살펴보자
* 기본적으로 TRUE인 부모 변수가 많아질수록 Y가 TRUE일 확률도 높아진다
    * 어떤 W값을 가지더라도 동일하다
* W와 W0 모두 10배로 키워보자
    * 시그모이드 함수의 지수 부분이 극단적으로 커진다는 것을 의미한다
    * 따라서 Y의 결과가 바뀌는 경계면이 훨씬 날카로워진다

### CPCS (Stanford Medical School)

* CPCS는 Stanford Medical School에서 질병을 진단하기 위해 만든 네트워크다
* 500개 가량의 변수가 있기 때문에 factorized form으로 나타낼 경우 1억 3천만개 정도의 파라미터가 필요하다
* Noisy max 모형을 적용하여 8000개의 파라미터로 줄일 수 있었다
