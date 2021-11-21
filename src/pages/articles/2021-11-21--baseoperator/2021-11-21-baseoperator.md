---
title: Airflow baseoperator를 이해하고 CustomOperator 생성하기
date: "2021-11-21T00:00:00.000Z"
layout: post
draft: false
path: "/posts/baseoperator/"
category: "airflow"
tags:
  - "airflow"
description: airflow baseoperator를 이해하고 이를 기반한 CustomOperator 생성하는 과정을 이해한다.
---

현재 다양한 프로젝트들 중에서 Airflow를 지속적으로 사용하다보니 Operator를 만들려면 어떻게 정의해야할지가 궁금해졌습니다. 
따라서 오늘은 각종 operator들 에서 상속하고 부모 클래스가 되는 Baseoperator에 대해서 이해하고 CustomOperator를 만들려면 어떤것이 필요한지 
글로 정리해보려고 합니다.

각종 Operator들의 Class에서 상속하고 있는 부모 Class를 보면 다양한 operator에서 `Baseoperator`를 공통적으로 상속하는
것을 볼 수 있습니다.

- [AWSAthenaOperator](https://github.com/apache/airflow/blob/main/airflow/providers/amazon/aws/operators/athena.py)
- [BashOperator](https://github.com/apache/airflow/blob/main/airflow/operators/bash.py)
- [DummyOperator](https://github.com/apache/airflow/blob/main/airflow/operators/dummy.py)

아래와 같은 형태로 공통적으로 Operator들의 코드를 보면 아래와 같이 구성되어 있습니다.
```python3
from airflow.models import BaseOperator

class XXXXXXOperator(BaseOperator): PEP 8에 근거하여 Classname은 PascalCase를 사용하는 듯 하다.
    ...
```
그렇다면 Custom Operator를 생성하기위해서는 Baseoperator를 상속받아서 작성을 하면 될 것 같은데 무엇이 필요한지 궁금해져서
해당 [BaseOperator](https://github.com/apache/airflow/blob/main/airflow/models/baseoperator.py)의 docstring을 보면 아래와 같이 설명되어 있습니다.

>Abstract base class for all operators. Since operators create objects that
>become nodes in the dag, BaseOperator contains many recursive methods for
>dag crawling behavior. To derive this class, you are expected to override
>the constructor as well as the 'execute' method.

doctstring에 따르면 Constructor(생성자)와 executor(실행)을 정의하면 Custom Operator를 만들 수 있다는것을 이해할 수 있었고
아래와 같이 CustomOperator를 직접 생성할 수 있었습니다.

이후의 Airflow에서 작성되어 있는 가장 간단한 Operator인 `DummyOperator`를 보고 Constructor(생성자)와 executor(실행)을 이해하고 CustomOperator인 TestOperator를 작성해 볼 수 있었습니다. 

```python
class DummyOperator(BaseOperator):
    """
    Operator that does literally nothing. It can be used to group tasks in a
    DAG.
    The task is evaluated by the scheduler but never processed by the executor.
    """

    ui_color = '#e8f7e4'
    inherits_from_dummy_operator = True

    def __init__(self, **kwargs) -> None: # Constructor에 해당하는 부분
        super().__init__(**kwargs)

    def execute(self, context): # executor에 해당 하는 부분
        pass
```

직접 작성한 TestOperator
```python
from airflow.models.baseoperator import BaseOperator

class TestOperator(BaseOperator):
    def __init__(
            self,
            name: str,
            **kwargs) -> None:
        super().__init__(**kwargs)
        self.name = name

    def execute(self, context):
        a = f"Test Operator {self.name}"
        return a 
```

```python
from test.test_operator import TestOperator

with DAG("test") as dag:
    task = TestOperator(task_id='test-1', name='byeongwoo')
```

그리고 공식 문서의 BaseOperator [설명](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/models/baseoperator/index.html) 에 어떤 Constructor를 정의해야하는지를 설명해 주고 있습니다.

오늘은 BaseOperator를 이해하고 CustomOperator를 정의해보는것에 대해서 이해보았고 Airflow Operator에 대해서 조금 더이해 할 수 있었습니다.

> fyi: 해당 글을 다썼을때 공식문서에 CustomOperator 작성법이 잘 설명되어 있는것을 확인했습니다 해당 글을 읽으시는 독자분께서는 공식문서를 통해 더 자세한 설명을 보실 수 있을 것 같습니다 [custom operator 공식문서 링크](https://airflow.apache.org/docs/apache-airflow/2.0.2/howto/custom-operator.html)


##### 출처
1. [Airflow Documentation](https://airflow.apache.org/docs/apache-airflow/2.0.2/index.html)<br>
2. [Airflwo Github](https://github.com/apache/airflow)