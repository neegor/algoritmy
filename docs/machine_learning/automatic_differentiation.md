# Автоматическая дифференциация 

В математике и компьютерной алгебре автоматическое дифференцирование, также известное как алгоритмическое дифференцирование, вычислительное дифференцирование или арифметическое дифференцирование, представляет собой набор методов, предназначенных для вычисления частной производной функции, заданной компьютерной программой.

Автоматическое дифференцирование — это сложный и важный инструмент, который позволяет автоматизировать процесс одновременного вычисления как числовых значений сложных функций, так и их производных. При этом нет необходимости в символическом представлении производной — требуется только функциональное правило или алгоритм его вычисления.

Таким образом, автоматическое дифференцирование не является ни чисто числовым, ни чисто символическим методом, а представляет собой нечто среднее между ними. Оно также превосходит обычные численные методы: в отличие от традиционных методов, основанных на конечных разностях, автоматическое дифференцирование обладает "теоретической" точностью, а по сравнению с символьными алгоритмами требует меньших вычислительных затрат.

Автоматическое дифференцирование основано на том факте, что каждое компьютерное вычисление, независимо от его сложности, состоит из последовательности элементарных арифметических операций, таких как сложение, вычитание, умножение и деление, а также элементарных функций, таких как экспонента, логарифм, синус и косинус. При многократном применении правила цепочки к этим операциям частные производные произвольного порядка могут быть вычислены автоматически с рабочей точностью, используя при этом не больше небольшого постоянного множителя арифметических операций, чем в исходной программе.

## Реализация

```python title="python"
from __future__ import annotations

from collections import defaultdict
from enum import Enum
from types import TracebackType
from typing import Any

import numpy as np
from typing_extensions import Self  # noqa: UP035


class OpType(Enum):

    ADD = 0
    SUB = 1
    MUL = 2
    DIV = 3
    MATMUL = 4
    POWER = 5
    NOOP = 6


class Variable:

    def __init__(self, value: Any) -> None:
        self.value = np.array(value)
        self.param_to: list[Operation] = []
        self.result_of: Operation = Operation(OpType.NOOP)

    def __repr__(self) -> str:
        return f"Variable({self.value})"

    def to_ndarray(self) -> np.ndarray:
        return self.value

    def __add__(self, other: Variable) -> Variable:
        result = Variable(self.value + other.value)

        with GradientTracker() as tracker:
            if tracker.enabled:
                tracker.append(OpType.ADD, params=[self, other], output=result)
        return result

    def __sub__(self, other: Variable) -> Variable:
        result = Variable(self.value - other.value)

        with GradientTracker() as tracker:
            if tracker.enabled:
                tracker.append(OpType.SUB, params=[self, other], output=result)
        return result

    def __mul__(self, other: Variable) -> Variable:
        result = Variable(self.value * other.value)

        with GradientTracker() as tracker:
            if tracker.enabled:
                tracker.append(OpType.MUL, params=[self, other], output=result)
        return result

    def __truediv__(self, other: Variable) -> Variable:
        result = Variable(self.value / other.value)

        with GradientTracker() as tracker:
            if tracker.enabled:
                tracker.append(OpType.DIV, params=[self, other], output=result)
        return result

    def __matmul__(self, other: Variable) -> Variable:
        result = Variable(self.value @ other.value)

        with GradientTracker() as tracker:
            if tracker.enabled:
                tracker.append(OpType.MATMUL, params=[self, other], output=result)
        return result

    def __pow__(self, power: int) -> Variable:
        result = Variable(self.value**power)

        with GradientTracker() as tracker:
            if tracker.enabled:
                tracker.append(
                    OpType.POWER,
                    params=[self],
                    output=result,
                    other_params={"power": power},
                )
        return result

    def add_param_to(self, param_to: Operation) -> None:
        self.param_to.append(param_to)

    def add_result_of(self, result_of: Operation) -> None:
        self.result_of = result_of


class Operation:

    def __init__(
        self,
        op_type: OpType,
        other_params: dict | None = None,
    ) -> None:
        self.op_type = op_type
        self.other_params = {} if other_params is None else other_params

    def add_params(self, params: list[Variable]) -> None:
        self.params = params

    def add_output(self, output: Variable) -> None:
        self.output = output

    def __eq__(self, value) -> bool:
        return self.op_type == value if isinstance(value, OpType) else False


class GradientTracker:

    instance = None

    def __new__(cls) -> Self:
        if cls.instance is None:
            cls.instance = super().__new__(cls)
        return cls.instance

    def __init__(self) -> None:
        self.enabled = False

    def __enter__(self) -> Self:
        self.enabled = True
        return self

    def __exit__(
        self,
        exc_type: type[BaseException] | None,
        exc: BaseException | None,
        traceback: TracebackType | None,
    ) -> None:
        self.enabled = False

    def append(
        self,
        op_type: OpType,
        params: list[Variable],
        output: Variable,
        other_params: dict | None = None,
    ) -> None:

        operation = Operation(op_type, other_params=other_params)
        param_nodes = []
        for param in params:
            param.add_param_to(operation)
            param_nodes.append(param)
        output.add_result_of(operation)

        operation.add_params(param_nodes)
        operation.add_output(output)

    def gradient(self, target: Variable, source: Variable) -> np.ndarray | None:

        partial_deriv = defaultdict(lambda: 0)
        partial_deriv[target] = np.ones_like(target.to_ndarray())

        operation_queue = [target.result_of]
        while len(operation_queue) > 0:
            operation = operation_queue.pop()
            for param in operation.params:
                dparam_doutput = self.derivative(param, operation)
                dparam_dtarget = dparam_doutput * partial_deriv[operation.output]
                partial_deriv[param] += dparam_dtarget

                if param.result_of and param.result_of != OpType.NOOP:
                    operation_queue.append(param.result_of)

        return partial_deriv.get(source)

    def derivative(self, param: Variable, operation: Operation) -> np.ndarray:

        params = operation.params

        if operation == OpType.ADD:
            return np.ones_like(params[0].to_ndarray(), dtype=np.float64)
        if operation == OpType.SUB:
            if params[0] == param:
                return np.ones_like(params[0].to_ndarray(), dtype=np.float64)
            return -np.ones_like(params[1].to_ndarray(), dtype=np.float64)
        if operation == OpType.MUL:
            return (
                params[1].to_ndarray().T
                if params[0] == param
                else params[0].to_ndarray().T
            )
        if operation == OpType.DIV:
            if params[0] == param:
                return 1 / params[1].to_ndarray()
            return -params[0].to_ndarray() / (params[1].to_ndarray() ** 2)
        if operation == OpType.MATMUL:
            return (
                params[1].to_ndarray().T
                if params[0] == param
                else params[0].to_ndarray().T
            )
        if operation == OpType.POWER:
            power = operation.other_params["power"]
            return power * (params[0].to_ndarray() ** (power - 1))

        err_msg = f"invalid operation type: {operation.op_type}"
        raise ValueError(err_msg)
```
