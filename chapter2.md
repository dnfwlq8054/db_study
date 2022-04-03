네트워크 모델은 코다실 이라 불리는 위원회에서 표준화 했다.
네트워크 모델에서 레코드는 다중 부모가 있을 수 있다.
때문에 코다실 모델은 다대일, 다대다 관계를 모델링할 수 있다.
레코디에 접근하는 방법은 최상위 레코드에서부터 연속된 연결 경로를 따야 한다.

이 모델은 매우 제한된 하드웨어 성능을 가장 효율적으로 사용할 수 있지만, 데이터 베이스 질의와 갱신을 위한 코드가 복잡하고 유연하지 못한 문제가 있다.

관계형 모델
대조적으로 관계형 모델이 하는 일은 알려진 모든 데이터를 배치하는 것이다. 관계형 데이터베이스에서 질의 최적화기는 질의의 어느 부분을 어떤 순서로 실행할지 결정하고 사용할 색인을 자동으로 결정한다.
때문에 데이터 변경이 비교적 쉽다.

관계형 데이터베이스에서 쓰기 스키마와 읽기 스키마가 있다.
쓰기 스키마는 정적 타입 확인과 비슷하고 읽기 스키마는 동적 타입 확인과 비슷하다.

데이터의 추가가 있을 때, 혹은 타입을 변경하고자 할 때 이 특징이 뚜렷이 나타난다.
읽기 스키마에서는 애플리케이션에서 수정해주면 된다.
```
if (user && user.name && !user.first_name) {
  // 2013년 12월 8일 이전에 쓴 문서에는 first_name이 없음
  user.first_name = user.naem.split(" ")[0];
}
 
반면 쓰기 스키마에서는 마이그레이션을 수행한다.
```
ALTER TABLE users ADD COLUMN first_name text;
UPDATE users set first_name split_part(name, ' ', 1); //postgresql
UPDATE users set first_name substring_index(name, ' ', 1); //mysql

대부분 관계형 데이터베이스 시스템은 ALTER TABLE문을 수 밀리초 안에 수행한다.
mysql은 ALTER TABLE 시에 전체 테이블을 복사한다.
때문에 큰 테이블을 변경할 대 수 분에서 수 시간까지 중단시간이 발생할 수 있다.
이런 방법을 해결하기 위해선 미리 데이터 값을 만들어 놓은 후 NULL로 초기화 하고, 필요할 때 채워 넣으면 된다.

문서 데이터 베이스는 보통 json, xml로 부호화된 단일 연속 문자열 (몽고 db같은)로 저장된다.
웹 페이지 상에 문서를 보여주는 동작처럼 애플리케이션이 자주 전체 문서에 접근해야 할 때 저장소 지역성(storage locality)을 활용하면 성능 이점이 있다.
지역성의 이점은 한번에 해당 문서의 많은 부분을 필요로 하는 경우에만 적용된다.

데이터베이스는 작은 문서에 접근해도 전체 문서를 적재해야 하기 때문에 낭비일 수 있다.
이런 이유로 문서를 아주 작게 유지하면서 무서의크기가 증가하는 쓰기를 피하라고 권장한다.

# 데이터를 위한 질의 언어
선언형 질의언어는 다음과 같다.
`SELECT * FROM animals WHERE family = 'Sharks';`

SQL이나 관계 대수 같은 선언형 질의 언어에는 목표를 달성하기 위한 방법이 아니라 알고자 하는 데이터의 패턴, 
즉 결과가 충족해야 하는 조건과 데이터를 어떻게 변환할지를 지정하기만 하면 된다.
질의가 어떤 순서로 실행할지를 결정하는 일은 데이터베이스 시스템의 질의 최적화기가 할 일이다.

선언형 언어는 병렬 실행에 적합하다. 데이터베이스 시스템의 질의 최적화가 실행 순서를 알아서 정해주기 때문이다.

# 맵리듀스 질의
몽고db와 카우치db를 포함한 일부 NSQL데이터 저장소는 제한된 형태의 맵리듀스를 지원한다.
맵리듀스 기능은 다음과 같이 표현할 수 있다.
postgresql
```
SELECT data_trunc('month',observation_timesteamp) AS observation_month,
  sum(num_animals) AS total_animals
FROM observations
WHERE family = 'Sharks'
GROUP BY observation_month;
```

mongoDB
```
db.observations.mapReduce(
  function map() {
    var year = this.observationTimestamp.getFullyear();
    var month = this.observationTimestamp.getMonth() + 1;
    emit(year + "-" month, this.numAnimals);
  },
  function reduce(key, values) {
    return Array.sum(values)
  },
  {
    query: { family: "Sharks" },
    out: "monthlySharkReport"
  }
};
```

이런 식으로 사용이 가능하며, 문자열을 파싱하고 라이브러리 함수를 호출하고 계산을 실행하는 등의
작업을 map과 reduce 함수에서 할 수 있다.

몽고 DB 2.2에는 집계 파이프라인 이라 부르는 선언형 질의 언어 지원을 추가했다.

    
