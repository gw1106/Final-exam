# Final-Exam



## 0. 기말고사 과제 내용

1. 모의 담금질 기법을 이용해 3차함수나 4차함수에서 글로벌 최적점을 찾는 parameter들을 추정한다.

2. 1번에 대한 검증

3. 1번과 2번을 통해 데이터를 수집하고 그 model의 곡선을 가장 잘 나타낼 수 있는 회귀식을 추정하고 결과랑 분석한다.



## 1. SimulatedAnnealing(모의 담금질 기법)

담금질 기법(Simulated Annealing, SA)은 전역 최적화 문제에 대한 일반적인 확률적 메타 알고리즘이다. 이 기법은 광대한 탐색 공간 안에서, 주어진 함수의 전역 최적해에 대한 좋은 근사를 준다. 커크패트릭, 젤라트, 베키가 1983년에 고안했다. 보통 영어를 그냥 읽어서 시뮬레이티드 어닐링이라고 부른다.

담금질 기법이라는 말은 금속 공학의 담금질(quenching)에서 왔는데, 이는 풀림(annealing)의 오역이다. 풀림은 금속재료를 가열한 다음 조금씩 냉각해 결정을 성장시켜 그 결함을 줄이는 작업이다. 열에 의해서 원자는 초기의 위치(내부 에너지가 극소점에 머무르는 상태)로부터 멀어져 에너지가 더욱 높은 상태로 추이된다. 천천히 냉각함으로써 원자는 초기 상태보다 내부 에너지가 한층 더 극소인 상태를 얻을 가능성이 많아진다.

다시말해 Simulated Annealing은 아주 어려운 문제를 풀기 위해 점진적으로 그 해(solution)에 가까운 방향으로 이동하되 적은 확률(0.05~0.5%)로 예상되는 해에 아주 먼 방향으로 이동하는 겁니다. 



## 2. 알고리즘 

```
1. 임의의 후보해 s를 선택한다.
2. 초기 T를 정한다.
3. repeat
4.    for i = 1 to kT {  // kT는 T에서의 for-루프 반복 횟수이다.
5.         s의 이웃해 중에서 랜덤하게 하나의 해 s'를 선택한다.
6.         d = (s'의 값) - (s의 값)
7.         if (d < 0)       // 이웃해인 s'가 더 우수한 경우
8.              s ← s'
9.         else              // s'가 s보다 우수하지 않은 경우
10.           q ← (0,1) 사이에서 랜덤하게 선택한 수
11.           if ( q < p ) s ← s'    // p는 자유롭게 탐색할 확률이다.
         }
12.    T ← T // 1보다 작은 상수  를 T에 곱하여 새로운 T를 계산 
13. until (종료 조건이 만족될 때까지)
14. return s
```



## 3. 모델 설정

![Model](http://www.index.go.kr/rMate/jsp/images/rMateChart_1009011.png)

위 사진은 한국의 시간이 지남에따라 인구성장률 변화이다. 요즘 결혼을 안하려는 청년들이 많다는 뉴스를 보기도 했고 우리나라의 저출산문제에 대해 관심이 있기 때문에 선택을 했다.



## 4. 알고리즘 구현

### Main

```java
package simulatedannealing;

public class Main {

    public static void main(String[] args) {

        // 000000
        // 010000

        SimulatedAnnealing sa = new SimulatedAnnealing(1, 0.0095, 100);
        sa.solve(new Problem() {
            @Override
            public double fit(double x) {
                return 0.0016*x*x*x*x -x*x + 0.037*x + 5;
            }

            @Override
            public boolean isNeighborBetter(double f0, double f1) {
                return f1 > f0;
            }
        }, 0, 31);

        System.out.println(sa.hist);
    
    }
}
```

### SimulatedAnnealing

```java
package simulatedannealing;

import java.util.ArrayList;
import java.util.Random;

public class SimulatedAnnealing {
    private double t;                        // 초기온도
    private double a;                        // 냉각비율
    private int niter;                       // 종료조건
    public ArrayList<Double> hist;

    public SimulatedAnnealing(double t, double a, int niter) {
        this.t = t;
        this.a = a;
        this.niter = niter;
        hist = new ArrayList<>();
    }

    public double solve(Problem p, double lower, double upper) {
        Random r = new Random();
        double x0 = r.nextDouble() * (upper - lower) + lower;    // 초기후보해
        double f0 = p.fit(x0);                                   // 초기후보해의 적합도
        hist.add(f0);

        for(int i=0; i<niter; i++) {                             // REPEAT, 담금질 하는 부분
            int kt = (int) Math.round(t * 20);
            for(int j=0; j<kt; j++) {
                double x1 = r.nextDouble() * (upper - lower) + lower;    // 이웃해
                double f1 = p.fit(x1);
                if(p.isNeighborBetter(f0, f1)) {                         // 이웃해가 더 나을 경우
                    x0 = x1;
                    f0 = f1;
                    hist.add(f0);
                } else {                                                 // 기존해가 더 나을 경우
                    double d = f1 - f0;
                    double p0 = Math.exp(-d/(t*t*t));
                    if(r.nextDouble() < p0) {
                        x0 = x1;
                        f0 = f1;
                        hist.add(f0);
                    }
                }
            }
            t *= a;
        }
        return x0;
    }
}
```

### Problem

```java
package simulatedannealing;

public interface Problem {
    double fit(double x);
    boolean isNeighborBetter(double f0, double f1);
}
```




## 5. 결과 사진
![스크린샷(55)](https://user-images.githubusercontent.com/81409594/121368304-f09b5380-c975-11eb-88e0-8916e65e2c0e.png)


Main메소드에서 냉각률을 0.0095로 감소켜보았다 그 결과는 위 사진과 같다. 결과는 parameter추정에 실패한 것 같다. t값도 더 낮게 해보고 p0값도 낮게 해보았지만 parameter 추정이 쉽지 않았다. 정확하게 parameter의 추정을 어떠한 방식으로 해야하는지 잘 모르겠습니다.




