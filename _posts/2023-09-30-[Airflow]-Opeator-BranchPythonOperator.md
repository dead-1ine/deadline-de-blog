---
layout: post
comments: true
date: 2023-09-30
title: "[Airflow 0] BranchPythonOperator의 간단 사용법"
description: "Apache Airflow의 Operator 중 BranchPythonOperator 사용법에 대해 설명합니다."
subject: blog
category: "Apache-Airflow"
tags: [ "Apache-Airflow" ]
---
# Apache Airflow - Operator - BranchPythonOperator
 
안녕하세요. 
이번 글은 Airflow에서 특정 조건에 맞는 Task를 실행하기 위해 Python 함수와 함께 사용되는 `BranchPythonOperator` 사용법에 대해 설명합니다.

- 선수 지식
  - Airflow PythonOperator
    BranchPythonOperator의 키워드 인자 중 `python_callable`에 대한 설명을 생략하였습니다.
  - Task Dependency
    BranchPythonOperator를 사용하여 생성한 Task가 Downstream Task로 연결되는 방법에 대한 설명을 생략하였습니다.
 
Airflow 공식 문서에서 정의하는 `BranchPythonOperator`는 다음과 같습니다.
 
```python
class airflow.operators.python.BranchPythonOperator
```
 
> It derives the PythonOperator and expects a Python function that returns a single task_id or list of task_ids to follow. The task_id(s) returned should point to a task directly downstream from {self}. All other “branches” or directly downstream tasks are marked with a state of skipped so that these paths can’t move forward. The skipped states are propagated downstream to allow for the DAG state to fill up and the DAG run’s state to be inferred.
 
간단하게 위 Operator를 간단하게 설명하면, Python 함수를 python_callable 키워드 인자로 사용하여 해당 함수가 반환하는 문자열과 같은 task_id(s)에 해당하는 Downstream Task(s)가 실행되도록 합니다. 
task_id(s)와 Downstream Task(s)로 표현한 이유는 해당 Operator를 사용하여 생성한 Task가 List로 묶인 task_id들을 반환하면 Downstream으로 해당하는 여러 개의 Task들이 실행되도록 합니다. 
해당 함수에서 반환되지 않은 Downstream Task(s)는 state가 `skipped`로 처리됩니다.
 

## Code Example(Apache-Airflow 2.2.2)
---
- Code를 example dag로 변경하고 이에 대해 설명하는 것으로 변경한다.
- 또한 다른 example들도 마찬가지로 풀어서 설명할 수 있도록 하자.
 
 
```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator
from airflow.operators.python import PythonBranchOperator


def branching_function(condition):
    if condition == 1:
        return 'branch_a'
    else:
        return 'branch_b'


with DAG(
    ...
) as dag:

    branching_task = PythonBranchOperator(
        task_id='branching',
        python_callable=branching_function,
        op_kwargs={
            'condition': 1,
        }
    )

    a_task = DummyOperator(
        task_id='branch_a'
    )

    b_task = DummyOperator(
        task_id='branch_b'
    )

    branching_task >> [a_task, b_task]
```
 
위 코드는 BranchPythonOperator를 사용하여 실행할 DownStream Task를 선택하는 예제입니다.
 
