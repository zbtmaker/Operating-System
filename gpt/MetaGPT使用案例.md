# MetaGPT
最近公司一直在推GPT相关的事物，这个文档就当是我了解GPT的第一步骤吧，之前对GPT有一定的，但是我现在对GPT没有任何接入的经验。公司最近一直在提倡AI Agent，我们利用AI Agent做什么。
## what


## why
## how

首先使用命令安装MetaGPT

```bash
pip3 install metagpt
```

我们可以看到这里



2024-03-03 16:56:48.235 | INFO     | __main__:main:127 - write a function that calculates the product of a list
2024-03-03 16:56:48.369 | INFO     | metagpt.team:invest:90 - Investment: $3.0.
2024-03-03 16:56:48.372 | INFO     | metagpt.roles.role:_act:399 - Alice(SimpleCoder): to do SimpleWriteCode(SimpleWriteCode)
```python
def product_of_list(numbers):
    result = 1
    for number in numbers:
        result *= number
    return result

your_code_here = product_of_list
```
2024-03-03 16:56:50.166 | WARNING  | metagpt.utils.cost_manager:update_cost:47 - Model GLM-4 not found in TOKEN_COSTS.
2024-03-03 16:56:50.170 | INFO     | __main__:_act:83 - Bob(SimpleTester): to do SimpleWriteTest(SimpleWriteTest)
```python
import pytest

def test_product_of_list_with_empty_list():
    assert your_code_here([]) == 1

def test_product_of_list_with_single_element():
    assert your_code_here([5]) == 5

def test_product_of_list_with_positive_numbers():
    assert your_code_here([1, 2, 3, 4]) == 24

def test_product_of_list_with_negative_numbers():
    assert your_code_here([-1, -2, -3, -4]) == -24

def test_product_of_list_with_mixed_numbers():
    assert your_code_here([-1, 2, -3, 4]) == 24

your_code_here = product_of_list
```
2024-03-03 16:56:55.326 | WARNING  | metagpt.utils.cost_manager:update_cost:47 - Model GLM-4 not found in TOKEN_COSTS.
2024-03-03 16:56:55.329 | INFO     | metagpt.roles.role:_act:399 - Charlie(SimpleReviewer): to do SimpleWriteReview(SimpleWriteReview)
One critical comment on the test cases provided is that they do not cover the scenario of a list containing zero. The product of any list that includes zero should be zero, as multiplying by zero always results in zero. It would be important to include a test case to ensure that the function correctly handles this situation. For example:

```python
def test_product_of_list_with_zero_in_list():
    assert your_code_here([1, 2, 3, 0]) == 0
```

This test case would verify that the function behaves as expected when a zero is present in the input list.
2024-03-03 16:56:59.944 | WARNING  | metagpt.utils.cost_manager:update_cost:47 - Model GLM-4 not found in TOKEN_COSTS.
2024-03-03 16:56:59.947 | INFO     | __main__:_act:83 - Bob(SimpleTester): to do SimpleWriteTest(SimpleWriteTest)
```python
def test_product_of_list_with_empty_list():
    assert product_of_list([]) == 1

def test_product_of_list_with_single_element():
    assert product_of_list([5]) == 5

def test_product_of_list_with_positive_numbers():
    assert product_of_list([1, 2, 3, 4]) == 24

def test_product_of_list_with_negative_numbers():
    assert product_of_list([-1, -2, -3, -4]) == -24

def test_product_of_list_with_mixed_numbers():
    assert product_of_list([-1, 2, -3, 4]) == 24

def test_product_of_list_with_zero_in_list():
    assert product_of_list([1, 2, 3, 0]) == 0

product_of_list
```
2024-03-03 16:57:06.187 | WARNING  | metagpt.utils.cost_manager:update_cost:47 - Model GLM-4 not found in TOKEN_COSTS.
2024-03-03 16:57:06.190 | INFO     | metagpt.roles.role:_act:399 - Charlie(SimpleReviewer): to do SimpleWriteReview(SimpleWriteReview)
One critical comment on the test cases provided is that while they cover a good range of scenarios, including empty list, single element, positive numbers, negative numbers, mixed numbers, and the presence of zero, they do not test the edge case of a list containing only zeros. It would be important to include a test case to check how the function handles a list where all elements are zero, to ensure that it consistently returns the expected result, which should be zero. For example:

```python
def test_product_of_list_with_all_zeros():
    assert product_of_list([0, 0, 0]) == 0
```

This additional test case would help to verify that the function is robust and can handle multiple zeros in the input list.
2024-03-03 16:57:12.238 | WARNING  | metagpt.utils.cost_manager:update_cost:47 - Model GLM-4 not found in TOKEN_COSTS.



