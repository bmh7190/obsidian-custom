---

---

### Domain Types in SQL
---

| 데이터 타입             | 설명                                         |
| ------------------ | ------------------------------------------ |
| `CHAR(n)`          | 고정 길이 문자열 (n보다 짧으면 공백 추가)                  |
| `VARCHAR(n)`       | 가변 길이 문자열 (짧으면 짧게 저장)                      |
| `INT`              | 일반 정수 (기계 의존 크기)                           |
| `SMALLINT`         | 작은 정수 (메모리 절약)                             |
| `NUMERIC(p,d)`     | 고정 소수점 숫자 (`p` = 전체 자릿수, `d` = 소수점 이하 자릿수) |
| `REAL`             | 부동 소수점 숫자 (낮은 정밀도)                         |
| `DOUBLE PRECISION` | 부동 소수점 숫자 (더 높은 정밀도)                       |
| `FLOAT(n)`         | 최소 `n`자리 유효 숫자의 부동 소수점                     |

---
### Create Table Constuct

create table 로 realation을 만들 수 있음
```SQL
create table r (A1 D1, A2 D2, ..., An Dn,
(integrity-constraint1),
...,
(integrity-constraintk))
```



---
### Integrity Constraints in Create Table
- primary key 
- foregin key  - references r
- not null

Example: Declare ID as the primary key for instructor
Declare dept_name as the foreign key referencing department

```SQL
create table instructor (
	ID char(5),
	name varchar(20) not null,
	dept_name varchar(20),
	salary numeric(8,2),
	primary key (ID),
	foreign key (dept_name) references department
)

```

primary key declaration on an attribute automatically ensures not null


foreign key 할 때 입력 값이 모두 있어야 함

---
### Drop and Alter Table Constructs

**1. DROP TABLE**

- 테이블 전체를 삭제할 때 사용
- 테이블 구조와 데이터 모두 제거됨
- 복구 불가능

```SQL
DROP TABLE 테이블명;
```

**2. ALTER TABLE**

테이블의 구조(속성)를 변경할 때 사용

**속성 추가**

```SQL
ALTER TABLE r ADD A,D;
```

테이블 r에 이름이 A인 속성을 도메인 D로 추가한다.  
기존 튜플의 A 속성 값은 두 NULL로 배정된다.

```SQL
ALTER TABLE STUDENT ADD address VARCHAR(100);
```

**속성 삭제**

```SQL
ALTER TABLE r DROP COLUMN A;
```

테이블 r에서 속성 A를 삭제한다.  

A가 외래 키로 설정되어 있거나 제약 조건이 걸려 있을 경우, 삭제 시 오류가 발생할 수 있으며 많은 DBMS에서 제한되거나 삭제가 불가능하다.

```SQL
ALTER TABLE STUDENT DROP COLUMN address;
```


---
### Basic Query Structure

```SQL
select A1, A2, ..., An
from r1, r2, ..., rm
where P
```

algebra에서는 기본적으로 중복을 제거 하고 출력!
하지만 SQL에서는 허용

만약 중복을 제거하고 싶다면?
```SQL
select distinct dept_name
from instructor
```

만약 복제된걸 모두 보고 싶다면?
```SQL
select all dept_name
from instructor
```

---
### Natural Join

주어진 SQL 쿼리:

```sql
SELECT name, course_id
FROM instructor NATURAL JOIN teaches;
```

이 쿼리는 **`instructor`** 테이블과 **`teaches`** 테이블을 **자연 조인(NATURAL JOIN)** 하여, `name`과 `course_id`를 선택하는 SQL입니다.

---

$$R=πname,courseid(instructor⋈teaches)R = \pi_{name, course_id} (\text{instructor} \bowtie \text{teaches})$$

여기서:

- ⋈ : 자연 조인(NATURAL JOIN)
- π_name,courseid : `name`과 `course_id` 속성만 선택하는 **π (projection, 투영 연산)**

---

**자연 조인(NATURAL JOIN)** 은 **두 릴레이션 사이에서 동일한 이름의 속성을 자동으로 찾아서 조인**을 수행함. 이로 인해 **의도하지 않은 속성끼리 조인이 발생할 수 있음**.

이 쿼리는 일반적으로 **Instructor(강사)** 와 **Course(과목)** 테이블이 있을 때, **강사가 어떤 과목을 가르치는지**를 조회하는 것입니다.

#### Incorrect version
```SQL
SELECT name, title
FROM instructor
NATURAL JOIN teaches
NATURAL JOIN course;
```

모든 테이블에 natural join 적용
테이블 간에 **이름은 같지만 의미는 다른 속성**이 있다면 **의도치 않은 조인**이 발생할 수 있음.


#### Correct version
```SQL
SELECT name, title
FROM instructor NATURAL JOIN teaches, course
WHERE teaches.course_id = course.course_id;
```


`instructor NATURAL JOIN teaches` 는 먼저 수행됨 (`id` 기준)
그 결과와 `course` 는 **FROM 절에 그냥 병렬 나열**, `WHERE` 절에서 **명시적으로 조인 조건 작성**

`teaches.course_id = course.course_id` 라고 **명확하게 조인 기준을 지정**하기 때문에 더 안전함
**NATURAL JOIN이 course 테이블에는 적용되지 않음** → 예기치 않은 속성 충돌 방지

---
# The Rename Operation

SQL은 `as` 를 사용하여 relation들과 attribute에 새로 이름을 짓는 것이 가능하다.

```SQL
old-name as new-name
```

```SQL
SELECT ID, name, salary/12 as monthly_salary
FROM instructor
```

`Comp.Sci.`의 교수들보다 더 높은 봉급을 받는 교수들의 이름을 출력하라

```SQL
SELECT distinct T.name
FROM instructor as T, instructor as S
where T.salary > S.salary and S.dept_name = 'Comp.Sci.'
```

**keyword `AS`는 선택 사항이며 생략할 수 있다.**
``
```SQL
instructor as T = instructor T
```

---
# String Operations

SQL은 문자열의 비교를 할 수 있도록 `string-matching` 기능을 제공한다.

**`LIKE` 연산자는 두 개의 특수 문자를 사용하여 패턴을 표현한다.**
- **`percent (%)`** : `%` 문자는 임의의 0개 이상의 문자**와 매칭된다. (즉, **어떤 문자열이든** 매칭됨)
- **`underscore (_)`** : `_` 문자는 임의의 한 개 문자**와 매칭된다. (즉, **정확히 한 글자**만 대체)

"dar" 을 포함하는 교수님 이름을 찾고 싶을 때는!
```SQL
SELECT name
FROM instructor
WHERE name LIKE '%dar%'
```

**SQL은 문자열 처리에 대해 다음과 같은 연산자와 함수를 제공한다.**

- `"||"` 연산자를 이용한 문자열 연결(concatenation)
- 대문자를 소문자로, 또는 소문자를 대문자로 변환
- 문자열의 길이를 구하거나, 부분 문자열(substring)을 추출

---
# Ordering the Display of Tuples

알파벳 순서로 이름을 정렬한다!

```SQL
SELECT DISTINCT name
FROM instructor
ORDER BY name
```

내림차순 정렬을 할 때는 `desc`을 사용하고 오름차순 정렬을 할 때는 `asc`를 사용한다. 
각 속성들에 대해서 오름차 순 정렬이 기본 값이다. 

```SQL
ORDER BY name DESC
```

여러 개의 속성들로 정렬할 수도 있다.

```SQL
ORDER BY dept_name, name
```

---
# Where Clause Predicates 


**SQL은 `BETWEEN` 연산자를 지원한다.**  
다음 예제는 **봉급이 90,000 이상 100,000 이하인 교수들의 이름**을 출력한다.

```sql
SELECT name
FROM instructor
WHERE salary BETWEEN 90000 AND 100000;
```

---

**튜플 비교 (Tuple Comparison)**

SQL에서는 **튜플 단위로 값 비교**가 가능하다.  
다음 예제는 `instructor`와 `teaches` 테이블에서 **ID가 같고, `dept_name`이 'Biology'인 경우**를 찾는다.

```sql
SELECT name, course_id
FROM instructor, teaches
WHERE (instructor.ID, dept_name) = (teaches.ID, 'Biology');
```

---

