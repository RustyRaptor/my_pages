# Problem 1: Shared Categories
1. Design a function `shared_categories(item_categories, k)` such that given a string `item_categories` and an integer `k` return the number of possible string split pairs such that the set of letters present in both the left and right substrings is greater than `k`

## Example 
```python
shared_categories("abbccab", 1) -> 4
```

Why?

|  prefix   |   suffix   |  shared letters  |  length of shared letters |  greater than k?  |
|-----------|------------|------------------|---------------------------|-------------------|
| a         | bbccab     | {a}              | 1                         | False             |
| ab        | bccab      | {a, b}           | 2                         | True              |
| abb       | ccab       | {a, b}           | 2                         | True              |
| abbc      | cab        | {a, b, c}        | 3                         | True              |
| abbcc     | ab         | {a, b}           | 2                         | True              |
| abbcca    | b          | {b}              | 1                         | False             |

There are 4 possible splits where there's more than 1 shared letter.



```python
# basic solution
def shared_categories(item_categories: str, k: int):
        greater_than_k = 0
        for index in range(1, len(item_categories)):
                prefix = set(item_categories[:index])
                suffix = set(item_categories[index:])
                
                shared_categories = prefix & suffix
                
                if len(shared_categories) > k:
                        greater_than_k += 1
        
        return greater_than_k
                
```


```python
shared_categories("abbccab", 1)
```




    4



## Performance improvements
1. We can exit early if we know the number of unique letters in `item_categories` is less than `k`
2. Lets compare performance when we pass a gigantic string of `"a"`s with k = 1


```python
def shared_categories_2(item_categories: str, k: int):
        if len(set(item_categories)) <= k:
                return 0
        greater_than_k = 0
        for index in range(1, len(item_categories)):
                prefix = item_categories[:index]
                suffix = item_categories[index:]
                
                shared_categories = set(prefix).intersection(set(suffix))
                
                if len(shared_categories) > k:
                        greater_than_k += 1
        
        return greater_than_k
```


```python
import timeit

item_categories = "abbccab"
k = 1
print("Time taken for shared_categories:", timeit.timeit(lambda: shared_categories(item_categories, k), number=10000))
print("Time taken for shared_categories_2:", timeit.timeit(lambda: shared_categories_2(item_categories, k), number=10000))


```

## Performance Improvements (2)
1. What if instead of computing the prefix and suffix every time, we iteratively build the prefix and then only compute the suffix each time?


```python
# basic solution
def shared_categories_3(item_categories: str, k: int):
        if len(set(item_categories)) <= k:
                return 0
        greater_than_k = 0
        prefix, suffix = set(), set(item_categories)
        for index, category in enumerate(item_categories):
                prefix.add(category)
                suffix = set(item_categories[index:])
                
                shared_categories = prefix.intersection(suffix)
                
                if len(shared_categories) > k:
                        greater_than_k += 1
        
        return greater_than_k
```


```python
import timeit
import random
import string

# Example: Compare shared_categories_2 and shared_categories_3 on test_str and k=1
k = 1
input_string = ''.join(random.choices(string.ascii_lowercase, k=10000))

time_shared_categories_2 = timeit.timeit(
        stmt="shared_categories_2(input_string, k)",
        globals=globals(),
        number=35
)
print(f"shared_categories_2 average time: {time_shared_categories_2 / 5:.6f}s")

time_shared_categories_3 = timeit.timeit(
        stmt="shared_categories_3(input_string, k)",
        globals=globals(),
        number=5
)
print(f"shared_categories_3 average time: {time_shared_categories_3 / 5:.6f}s")
```

    shared_categories_2 average time: 6.488943s
    shared_categories_3 average time: 0.435434s


# Problem 2: Book Buying

## Description

1. Design a function `optimized_cost(costs, pair_price, k) -> int`
2. Given `costs`, a fixed list of prices corresponding to a fixed ordered list of books.
3. Given `pair_price`, a fixed discount cost of buying the current leftmost and rightmost book in `costs`
4. Given `k`, the number of times we are allowed to apply `pair_price`
5. One step at a time, the customer may do one of the following actions.
    1. Purchase the leftmost book in the list, removing it from the list.
    2. Purchase the rightmost book in the list, removing it from the list.
    3. If `k > 0`, and `len(costs) >= 2`, purchase the rightmost AND leftmost books at the same time at the `pair_price` cost, removing them from the list and decreasing `k` by `1`.
6. The goal is to find the best price that a customer can get by applying these steps.

- Note: This is a genuinely very challenging problem and requires careful thought and it helps to know concepts of traditional AI

## First Approach

### State
1. The first approach I will use in this problem is to think of it as a game and define the state of the game.
2. In this problem we have three factors that can change and impact the available next actions. 
    1. `costs` which is the list of books we can currently buy.
    2. `k` which is the number of times remaning that we can apply `pair_price`
    3. `money_spent` we will create this to keep track of how much we have spent on the books so far. 
3. We also have the constant `pair_price` which does not change but does determine next optimal steps. 

#### Lets define the initial state in python

1. We can use a python dictionary to store information about the state
2. You can also do this with a tuple, three variables, or whatever. But dictionaries are easy to pass around and easy to understand.

```python
state = {
    "available_costs": costs,
    "k": k,
    "money_spent": 0
    "pair_price": pair_price,
}
```

### Actions

1. Now that we defined the state we can define the 3 possible actions we can take each turn.
    1. `buy_left(state)`, `buy_right(state)`, and `buy_pair(state)`
2. We will just define functions that take the `state` dict as input and return a new changed `state`
3. We will also add some checks for basic rules of the problem. 





```python
def buy_left(state: dict) -> dict:
        if not state["available_costs"]:
                msg = "No items available to purchase."
                raise ValueError(msg)
        leftmost_cost = state["available_costs"][0]
        costs_after_purchasing = state["available_costs"][1:]

        new_state = {
                "available_costs": costs_after_purchasing,
                "k": state["k"],
                "money_spent": state["money_spent"] + leftmost_cost,
                "pair_price": state["pair_price"],
        }

        print("buy_left | ", end="")

        return new_state


def buy_right(state: dict) -> dict:
        if not state["available_costs"]:
                msg = "No items available to purchase."
                raise ValueError(msg)
        rightmost_cost = state["available_costs"][-1]
        costs_after_purchasing = state["available_costs"][:-1]

        # create a new state with the updated costs and money spent
        new_state = {
                "available_costs": costs_after_purchasing,
                "k": state["k"],
                "money_spent": state["money_spent"] + rightmost_cost,
                "pair_price": state["pair_price"],
        }

        print("buy_right | ", end="")

        return new_state


def buy_pair(state: dict) -> dict:
        if len(state["available_costs"]) < 2:
                msg = "Not enough items to buy a pair. At least 2 items are required."
                raise ValueError(msg)
        if state["k"] < 1:
                msg = "Cannot buy a pair because k is less than 1."
                raise ValueError(msg)

        costs_after_purchasing = state["available_costs"][1:-1]

        new_state = {
                "available_costs": costs_after_purchasing,
                "k": state["k"] - 1,
                "money_spent": state["money_spent"] + state["pair_price"],
                "pair_price": state["pair_price"],
        }

        print("buy_pair | ", end="")

        return new_state

```

### Logic

#### Variables
1. `min_cost`: Starts as INFINITY We want to keep track of the lowest cost we've gotten so far. We can use this to improve performance too.
2. `initial_state`: we want to keep track of the starting state so we can reset it. Starts as we discussed in state.

#### Base cases

1. `state["available_costs"]` is empty
    1. No more steps to take now we can record the total cost and reset the state.
2. For optimizing: `state["money_spent"] >= min_cost`
    1. We can discard the current path and keep the `min_cost`

#### Main logic
1. Used recursion because it's simple to implement.


```python
def find_min_cost(state: dict) -> int:
        if not state["available_costs"]:
                print(state["money_spent"])
                return state["money_spent"]

        if state["k"] < 1 or len(state["available_costs"]) < 2:
                actions = [buy_left, buy_right]
        else:
                actions = [buy_pair, buy_left, buy_right]

        return min(find_min_cost(action(state)) for action in actions)
```

#### The deliverable
1. We can use `find_min_cost` to return the value we need. 


```python
def optimized_cost(costs: list, k: int, pair_price: int) -> int:
        if not costs:
                return 0
        initial_state = {
                "available_costs": costs,
                "k": k,
                "money_spent": 0,
                "pair_price": pair_price,
        }
        return find_min_cost(initial_state)
```


```python
# Test case 1: No pair purchases allowed
# Expected: 30 (buy all individually)
print(optimized_cost([5, 10, 15], k=0, pair_price=8))

print("Test case 2: One pair purchase allowed, obvious expensive pair -> 13")
print(optimized_cost([5, 10, 15], k=1, pair_price=8))

print("# Test case 3: as many pairs as you want -> 15")
print(optimized_cost([3, 7, 2, 6, 5, 4], k=9999999999, pair_price=5))

print("# Test case 4: pair_price is more expensive than any pair -> 27")
print(optimized_cost([3, 7, 2, 6, 5, 4], k=9999999999, pair_price=20))

print("# Test case 5: there's 1 worthwhile pair but all others should be\n"
"# purchased individually -> 30")
print(optimized_cost([1, 15, 2, 12, 3, 4], k=999999999, pair_price=20))



```

    buy_left | buy_left | buy_left | 30
    buy_right | 30
    buy_right | buy_left | 30
    buy_right | 30
    buy_right | buy_left | buy_left | 30
    buy_right | 30
    buy_right | buy_left | 30
    buy_right | 30
    30
    Test case 2: One pair purchase allowed, obvious expensive pair -> 13
    buy_pair | buy_left | 18
    buy_right | 18
    buy_left | buy_pair | 13
    buy_left | BAD 15 >= 13
    buy_right | BAD 20 >= 13
    buy_right | BAD 15 >= 13
    13
    # Test case 3: as many pairs as you want -> 15
    buy_pair | buy_pair | buy_pair | 15
    buy_left | buy_left | 18
    buy_right | 18
    buy_right | BAD 16 >= 15
    buy_left | buy_pair | BAD 17 >= 15
    buy_left | buy_pair | 19
    buy_left | BAD 20 >= 15
    buy_right | BAD 19 >= 15
    buy_right | BAD 17 >= 15
    buy_right | buy_pair | BAD 15 >= 15
    buy_left | BAD 17 >= 15
    buy_right | BAD 16 >= 15
    buy_left | buy_pair | buy_pair | buy_left | 19
    buy_right | 19
    buy_left | buy_pair | 15
    buy_left | BAD 16 >= 15
    buy_right | BAD 15 >= 15
    buy_right | buy_pair | 18
    buy_left | BAD 15 >= 15
    buy_right | BAD 19 >= 15
    buy_left | buy_pair | BAD 15 >= 15
    buy_left | buy_pair | BAD 17 >= 15
    buy_left | BAD 18 >= 15
    buy_right | BAD 16 >= 15
    buy_right | buy_pair | BAD 19 >= 15
    buy_left | BAD 16 >= 15
    buy_right | BAD 19 >= 15
    buy_right | buy_pair | buy_pair | 17
    buy_left | buy_left | 20
    buy_right | 20
    buy_right | BAD 18 >= 15
    buy_left | buy_pair | BAD 19 >= 15
    buy_left | BAD 16 >= 15
    buy_right | BAD 19 >= 15
    buy_right | buy_pair | BAD 17 >= 15
    buy_left | BAD 19 >= 15
    buy_right | BAD 18 >= 15
    buy_right | buy_pair | buy_pair | buy_left | 16
    buy_right | 16
    buy_left | BAD 16 >= 15
    buy_right | BAD 15 >= 15
    buy_left | buy_pair | buy_pair | 17
    buy_left | buy_left | 20
    buy_right | 20
    buy_right | BAD 18 >= 15
    buy_left | buy_pair | BAD 19 >= 15
    buy_left | BAD 16 >= 15
    buy_right | BAD 19 >= 15
    buy_right | buy_pair | BAD 17 >= 15
    buy_left | BAD 19 >= 15
    buy_right | BAD 18 >= 15
    buy_right | buy_pair | buy_pair | 19
    buy_left | BAD 21 >= 15
    buy_right | BAD 16 >= 15
    buy_left | buy_pair | BAD 17 >= 15
    buy_left | BAD 19 >= 15
    buy_right | BAD 18 >= 15
    buy_right | BAD 15 >= 15
    15
    # Test case 4: pair_price is more expensive than any pair -> 27
    buy_pair | buy_pair | buy_pair | 60
    buy_left | buy_left | 48
    buy_right | 48
    buy_right | buy_left | 48
    buy_right | 48
    buy_left | buy_pair | buy_left | 53
    buy_right | 53
    buy_left | buy_pair | 49
    buy_left | buy_left | 40
    buy_right | 40
    buy_right | buy_left | 40
    buy_right | 40
    buy_right | buy_pair | 52
    buy_left | buy_left | 40
    buy_right | 40
    buy_right | buy_left | 40
    buy_right | 40
    buy_right | buy_pair | BAD 45 >= 40
    buy_left | buy_pair | 52
    buy_left | buy_left | 40
    buy_right | 40
    buy_right | buy_left | 40
    buy_right | 40
    buy_right | buy_pair | 51
    buy_left | buy_left | 40
    buy_right | 40
    buy_right | buy_left | 40
    buy_right | 40
    buy_left | buy_pair | buy_pair | BAD 43 >= 40
    buy_left | buy_pair | 45
    buy_left | buy_left | 36
    buy_right | 36
    buy_right | buy_left | 36
    buy_right | 36
    buy_right | buy_pair | 48
    buy_left | buy_left | 36
    buy_right | 36
    buy_right | buy_left | 36
    buy_right | 36
    buy_left | buy_pair | buy_pair | 50
    buy_left | BAD 36 >= 36
    buy_right | buy_left | 41
    buy_right | 41
    buy_left | buy_pair | buy_left | 37
    buy_right | 37
    buy_left | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 36
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 34 >= 27
    buy_left | buy_pair | 36
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 27 >= 27
    buy_left | buy_pair | BAD 34 >= 27
    buy_left | buy_pair | 36
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 32 >= 27
    buy_left | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | buy_pair | BAD 44 >= 27
    buy_left | BAD 31 >= 27
    buy_right | BAD 30 >= 27
    buy_left | buy_pair | BAD 27 >= 27
    buy_left | buy_pair | BAD 34 >= 27
    buy_left | buy_pair | 36
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 32 >= 27
    buy_left | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 29 >= 27
    buy_left | buy_pair | BAD 32 >= 27
    buy_left | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 35 >= 27
    buy_left | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 37
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    27
    # Test case 5: there's 1 worthwhile pair but all others should be
    # purchased individually -> 30
    buy_pair | buy_pair | buy_pair | 60
    buy_left | buy_left | 54
    buy_right | 54
    buy_right | buy_left | 54
    buy_right | 54
    buy_left | buy_pair | BAD 55 >= 54
    buy_left | buy_pair | 57
    buy_left | buy_left | 52
    buy_right | 52
    buy_right | buy_left | 52
    buy_right | 52
    buy_right | buy_pair | 58
    buy_left | buy_left | 52
    buy_right | 52
    buy_right | buy_left | 52
    buy_right | 52
    buy_right | buy_pair | buy_left | 45
    buy_right | 45
    buy_left | buy_pair | 58
    buy_left | buy_left | 52
    buy_right | 52
    buy_right | BAD 50 >= 45
    buy_right | buy_pair | 55
    buy_left | BAD 50 >= 45
    buy_right | buy_left | 52
    buy_right | 52
    buy_left | buy_pair | buy_pair | buy_left | 53
    buy_right | 53
    buy_left | buy_pair | 43
    buy_left | buy_left | 38
    buy_right | 38
    buy_right | buy_left | 38
    buy_right | 38
    buy_right | buy_pair | 44
    buy_left | buy_left | 38
    buy_right | 38
    buy_right | buy_left | 38
    buy_right | 38
    buy_left | buy_pair | buy_pair | 56
    buy_left | BAD 48 >= 38
    buy_right | BAD 39 >= 38
    buy_left | buy_pair | BAD 38 >= 38
    buy_left | buy_pair | 50
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 42
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | BAD 40 >= 37
    buy_left | buy_pair | 42
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | buy_pair | 45
    buy_left | buy_left | 39
    buy_right | 39
    buy_right | BAD 37 >= 37
    buy_left | buy_pair | BAD 40 >= 37
    buy_left | buy_pair | 42
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | buy_left | 30
    buy_right | 30
    buy_left | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 35 >= 30
    buy_right | buy_pair | 40
    buy_left | BAD 35 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | buy_pair | BAD 44 >= 30
    buy_left | BAD 39 >= 30
    buy_right | BAD 36 >= 30
    buy_left | buy_pair | buy_pair | 45
    buy_left | buy_left | 39
    buy_right | 39
    buy_right | BAD 37 >= 30
    buy_left | buy_pair | BAD 40 >= 30
    buy_left | buy_pair | 42
    buy_left | BAD 34 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 35 >= 30
    buy_right | buy_pair | buy_left | 30
    buy_right | 30
    buy_left | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 35 >= 30
    buy_right | buy_pair | 40
    buy_left | BAD 35 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | buy_pair | 47
    buy_left | BAD 42 >= 30
    buy_right | buy_left | 44
    buy_right | 44
    buy_left | buy_pair | buy_left | 30
    buy_right | 30
    buy_left | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 35 >= 30
    buy_right | buy_pair | 40
    buy_left | BAD 35 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | BAD 39 >= 30
    buy_left | buy_pair | 40
    buy_left | BAD 35 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 41
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 36 >= 30
    30


#

#### Optimization

1. Now we know our implementation is correct. But you might notice from the outputs that we are doing a lot more checks than we need to. 
2. We can use a method called pruning to avoid this. We just need to restructure the code so that we are tracking min_cost after each full branch run. 


```python
def find_min_cost_pruned(state: dict, min_cost: int = float("inf")) -> int:
        if not state["available_costs"]:
                print(state["money_spent"])
                return min(min_cost, state["money_spent"])
        if state["money_spent"] >= min_cost:
                print(f"BAD {state['money_spent']} >= {min_cost}")
                return min_cost

        if state["k"] < 1 or len(state["available_costs"]) < 2:
                actions = [buy_left, buy_right]
        else:
                actions = [buy_pair, buy_left, buy_right]
        for action in actions:
                min_cost = find_min_cost_pruned(action(state), min_cost)
        return min_cost

```

##### Now we can plug that pruning function into our deliverable. 


```python
def optimized_cost(costs: list, k: int, pair_price: int) -> int:
        if not costs:
                return 0
        initial_state = {
                "available_costs": costs,
                "k": k,
                "money_spent": 0,
                "pair_price": pair_price,
        }
        return find_min_cost_pruned(initial_state)
```


```python
print(
        "# Test case 1: No pair purchases allowed"
        "# Expected: 30 (buy all individually)"
)
print(optimized_cost([5, 10, 15], k=0, pair_price=8))

print("Test case 2: One pair purchase allowed, obvious expensive pair -> 13")
print(optimized_cost([5, 10, 15], k=1, pair_price=8))

print("# Test case 3: as many pairs as you want -> 15")
print(optimized_cost([3, 7, 2, 6, 5, 4], k=9999999999, pair_price=5))

print("# Test case 4: pair_price is more expensive than any pair -> 27")
print(optimized_cost([3, 7, 2, 6, 5, 4], k=9999999999, pair_price=20))

print(
        "# Test case 5: there's 1 worthwhile pair but all others should be\n"
        "# purchased individually -> 30"
)
print(optimized_cost([1, 15, 2, 12, 3, 4], k=999999999, pair_price=20))

```

    buy_left | buy_left | buy_left | 30
    buy_right | 30
    buy_right | buy_left | 30
    buy_right | 30
    buy_right | buy_left | buy_left | 30
    buy_right | 30
    buy_right | buy_left | 30
    buy_right | 30
    30
    Test case 2: One pair purchase allowed, obvious expensive pair -> 13
    buy_pair | buy_left | 18
    buy_right | 18
    buy_left | buy_pair | 13
    buy_left | BAD 15 >= 13
    buy_right | BAD 20 >= 13
    buy_right | BAD 15 >= 13
    13
    # Test case 3: as many pairs as you want -> 15
    buy_pair | buy_pair | buy_pair | 15
    buy_left | buy_left | 18
    buy_right | 18
    buy_right | BAD 16 >= 15
    buy_left | buy_pair | BAD 17 >= 15
    buy_left | buy_pair | 19
    buy_left | BAD 20 >= 15
    buy_right | BAD 19 >= 15
    buy_right | BAD 17 >= 15
    buy_right | buy_pair | BAD 15 >= 15
    buy_left | BAD 17 >= 15
    buy_right | BAD 16 >= 15
    buy_left | buy_pair | buy_pair | buy_left | 19
    buy_right | 19
    buy_left | buy_pair | 15
    buy_left | BAD 16 >= 15
    buy_right | BAD 15 >= 15
    buy_right | buy_pair | 18
    buy_left | BAD 15 >= 15
    buy_right | BAD 19 >= 15
    buy_left | buy_pair | BAD 15 >= 15
    buy_left | buy_pair | BAD 17 >= 15
    buy_left | BAD 18 >= 15
    buy_right | BAD 16 >= 15
    buy_right | buy_pair | BAD 19 >= 15
    buy_left | BAD 16 >= 15
    buy_right | BAD 19 >= 15
    buy_right | buy_pair | buy_pair | 17
    buy_left | buy_left | 20
    buy_right | 20
    buy_right | BAD 18 >= 15
    buy_left | buy_pair | BAD 19 >= 15
    buy_left | BAD 16 >= 15
    buy_right | BAD 19 >= 15
    buy_right | buy_pair | BAD 17 >= 15
    buy_left | BAD 19 >= 15
    buy_right | BAD 18 >= 15
    buy_right | buy_pair | buy_pair | buy_left | 16
    buy_right | 16
    buy_left | BAD 16 >= 15
    buy_right | BAD 15 >= 15
    buy_left | buy_pair | buy_pair | 17
    buy_left | buy_left | 20
    buy_right | 20
    buy_right | BAD 18 >= 15
    buy_left | buy_pair | BAD 19 >= 15
    buy_left | BAD 16 >= 15
    buy_right | BAD 19 >= 15
    buy_right | buy_pair | BAD 17 >= 15
    buy_left | BAD 19 >= 15
    buy_right | BAD 18 >= 15
    buy_right | buy_pair | buy_pair | 19
    buy_left | BAD 21 >= 15
    buy_right | BAD 16 >= 15
    buy_left | buy_pair | BAD 17 >= 15
    buy_left | BAD 19 >= 15
    buy_right | BAD 18 >= 15
    buy_right | BAD 15 >= 15
    15
    # Test case 4: pair_price is more expensive than any pair -> 27
    buy_pair | buy_pair | buy_pair | 60
    buy_left | buy_left | 48
    buy_right | 48
    buy_right | buy_left | 48
    buy_right | 48
    buy_left | buy_pair | buy_left | 53
    buy_right | 53
    buy_left | buy_pair | 49
    buy_left | buy_left | 40
    buy_right | 40
    buy_right | buy_left | 40
    buy_right | 40
    buy_right | buy_pair | 52
    buy_left | buy_left | 40
    buy_right | 40
    buy_right | buy_left | 40
    buy_right | 40
    buy_right | buy_pair | BAD 45 >= 40
    buy_left | buy_pair | 52
    buy_left | buy_left | 40
    buy_right | 40
    buy_right | buy_left | 40
    buy_right | 40
    buy_right | buy_pair | 51
    buy_left | buy_left | 40
    buy_right | 40
    buy_right | buy_left | 40
    buy_right | 40
    buy_left | buy_pair | buy_pair | BAD 43 >= 40
    buy_left | buy_pair | 45
    buy_left | buy_left | 36
    buy_right | 36
    buy_right | buy_left | 36
    buy_right | 36
    buy_right | buy_pair | 48
    buy_left | buy_left | 36
    buy_right | 36
    buy_right | buy_left | 36
    buy_right | 36
    buy_left | buy_pair | buy_pair | 50
    buy_left | BAD 36 >= 36
    buy_right | buy_left | 41
    buy_right | 41
    buy_left | buy_pair | buy_left | 37
    buy_right | 37
    buy_left | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 36
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 34 >= 27
    buy_left | buy_pair | 36
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 27 >= 27
    buy_left | buy_pair | BAD 34 >= 27
    buy_left | buy_pair | 36
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 32 >= 27
    buy_left | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | buy_pair | BAD 44 >= 27
    buy_left | BAD 31 >= 27
    buy_right | BAD 30 >= 27
    buy_left | buy_pair | BAD 27 >= 27
    buy_left | buy_pair | BAD 34 >= 27
    buy_left | buy_pair | 36
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 32 >= 27
    buy_left | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 29 >= 27
    buy_left | buy_pair | BAD 32 >= 27
    buy_left | buy_pair | 39
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | BAD 35 >= 27
    buy_left | buy_pair | 38
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    buy_right | buy_pair | 37
    buy_left | buy_left | 27
    buy_right | 27
    buy_right | buy_left | 27
    buy_right | 27
    27
    # Test case 5: there's 1 worthwhile pair but all others should be
    # purchased individually -> 30
    buy_pair | buy_pair | buy_pair | 60
    buy_left | buy_left | 54
    buy_right | 54
    buy_right | buy_left | 54
    buy_right | 54
    buy_left | buy_pair | BAD 55 >= 54
    buy_left | buy_pair | 57
    buy_left | buy_left | 52
    buy_right | 52
    buy_right | buy_left | 52
    buy_right | 52
    buy_right | buy_pair | 58
    buy_left | buy_left | 52
    buy_right | 52
    buy_right | buy_left | 52
    buy_right | 52
    buy_right | buy_pair | buy_left | 45
    buy_right | 45
    buy_left | buy_pair | 58
    buy_left | buy_left | 52
    buy_right | 52
    buy_right | BAD 50 >= 45
    buy_right | buy_pair | 55
    buy_left | BAD 50 >= 45
    buy_right | buy_left | 52
    buy_right | 52
    buy_left | buy_pair | buy_pair | buy_left | 53
    buy_right | 53
    buy_left | buy_pair | 43
    buy_left | buy_left | 38
    buy_right | 38
    buy_right | buy_left | 38
    buy_right | 38
    buy_right | buy_pair | 44
    buy_left | buy_left | 38
    buy_right | 38
    buy_right | buy_left | 38
    buy_right | 38
    buy_left | buy_pair | buy_pair | 56
    buy_left | BAD 48 >= 38
    buy_right | BAD 39 >= 38
    buy_left | buy_pair | BAD 38 >= 38
    buy_left | buy_pair | 50
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 42
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | BAD 40 >= 37
    buy_left | buy_pair | 42
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | buy_pair | 45
    buy_left | buy_left | 39
    buy_right | 39
    buy_right | BAD 37 >= 37
    buy_left | buy_pair | BAD 40 >= 37
    buy_left | buy_pair | 42
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | buy_left | 30
    buy_right | 30
    buy_left | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 35 >= 30
    buy_right | buy_pair | 40
    buy_left | BAD 35 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | buy_pair | BAD 44 >= 30
    buy_left | BAD 39 >= 30
    buy_right | BAD 36 >= 30
    buy_left | buy_pair | buy_pair | 45
    buy_left | buy_left | 39
    buy_right | 39
    buy_right | BAD 37 >= 30
    buy_left | buy_pair | BAD 40 >= 30
    buy_left | buy_pair | 42
    buy_left | BAD 34 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 35 >= 30
    buy_right | buy_pair | buy_left | 30
    buy_right | 30
    buy_left | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 35 >= 30
    buy_right | buy_pair | 40
    buy_left | BAD 35 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | buy_pair | 47
    buy_left | BAD 42 >= 30
    buy_right | buy_left | 44
    buy_right | 44
    buy_left | buy_pair | buy_left | 30
    buy_right | 30
    buy_left | buy_pair | 43
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 35 >= 30
    buy_right | buy_pair | 40
    buy_left | BAD 35 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | BAD 39 >= 30
    buy_left | buy_pair | 40
    buy_left | BAD 35 >= 30
    buy_right | buy_left | 37
    buy_right | 37
    buy_right | buy_pair | 41
    buy_left | buy_left | 37
    buy_right | 37
    buy_right | BAD 36 >= 30
    30


# Doge's Version of Problem 2


```python
from collections.abc import Callable
from typing import Annotated, NamedTuple


class State(NamedTuple):
        available_costs: tuple[int, ...]
        k: Annotated[int, "Number of pair_price discounts available for use"]
        money_spent: int

def buy_left(state: State) -> State:
        if not state.available_costs:
                msg = "No items available to purchase."
                raise ValueError(msg)
        print("buy_left | ", end="")
        return State(
                available_costs=state.available_costs[1:],
                k=state.k,
                money_spent=state.money_spent + state.available_costs[0],
        )

def buy_right(state: State) -> State:
        if not state.available_costs:
                msg = "No items available to purchase."
                raise ValueError(msg)
        print("buy_right | ", end="")
        return State(
                available_costs=state.available_costs[:-1],
                k=state.k,
                money_spent=state.money_spent + state.available_costs[-1],
        )

def buy_pair(state: State, pair_price: int) -> State:
        if len(state.available_costs) < 2:
                msg = (
                        "Not enough items to buy a pair."
                        "At least 2 items are required."
                )
                raise ValueError(msg)
        
        if state.k < 1:
                msg = "Cannot buy a pair because k is less than 1."
                raise ValueError(msg)
        
        print("buy_pair | ", end="")
        
        return State(
                available_costs=state.available_costs[1:-1],
                k=state.k - 1,
                money_spent=state.money_spent + pair_price,
        )

def next_actions(
                state: State,
                pair_price: int
) -> tuple[Callable[[State], State], ...]:
        if state.k < 1 or len(state.available_costs) < 2:
                return (buy_left, buy_right)
        
        return (lambda s: buy_pair(s, pair_price), buy_left, buy_right)

def find_min_cost(
                state: State,
                pair_price: int,
                min_cost: int = float("inf")
) -> int:
        if not state.available_costs:
                print(state.money_spent)
                return min(min_cost, state.money_spent)
        
        if state.money_spent >= min_cost:
                print(f"BAD {state.money_spent} >= {min_cost}")
                return min_cost
        
        for action in next_actions(state, pair_price):
                min_cost = find_min_cost(action(state), pair_price, min_cost)
        return min_cost

```

# Important Resources and Information
1. General Stuff
    1. Required for above problems
        1. Python Dicts
            1. RealPython: https://realpython.com/python-dicts/ 
            2. El Pythonista: https://elpythonista.com/diccionarios-en-python-dict
            3. Spanish Docs: https://docs.python.org/es/3.12/tutorial/datastructures.html#dictionaries
        2. Python Iteration
            1. RealPython: https://realpython.com/python-iterators-iterables/
            2. ElLibroDePython: https://ellibrodepython.com/iterator-python
    2. Not required for above but highly recommended reading. 
        1. Functional Programing
            1. ElLibroDePython: https://ellibrodepython.com/programacion-funcional-python
            2. RealPython: https://realpython.com/python-functional-programming/ 
            3. My Note: It's a way of thinking and communicating about software problems. I use it whenever I can.
        2. Generators
            1. ElLibroDePython: https://ellibrodepython.com/yield-python
        3. Type annotations
            1. ElLibroDePython: https://ellibrodepython.com/function-annotations 
2. Problem 1
    1. Sets in python: 
        1. RealPython https://realpython.com/python-sets/
        2. Spanish Docs: https://docs.python.org/es/3.13/library/stdtypes.html#set-types-set-frozenset
        3. El Pythonista: https://elpythonista.com/conjuntos-en-python-set
    2. For loops in python:
        1. RealPython: https://realpython.com/python-for-loop/
        2. ElLibroDePython: https://ellibrodepython.com/for-python
        3. j2logo: https://j2logo.com/bucle-for-en-python/
    3. Python `enumerate()` (used with for loop)
        1. RealPython: https://realpython.com/ref/builtin-functions/enumerate/
        2. Spanish Docs: https://docs.python.org/es/3.13/library/functions.html#enumerate
        3. ElLibroDePython: https://ellibrodepython.com/enumerate-python
4. Problem 2
    1. Ohhhhh boy this one is a deep rabbit hole.
        1. Solving problems with Search:
            1. As close as I can get to a code summary of what I implemented: https://github.com/aimacode/aima-python/blob/master/search4e.ipynb
            2. Formulating Problems: https://aima.cs.berkeley.edu/4th-ed/pdfs/newchap03.pdf 
                1. Important Things: `initial state`, `actions`, `successor function`, `path cost`, `optimal solution`
            3. Full book: https://annas-archive.org/md5/fb03b9640fac32dd6c301685f9393994 
            4. Geeks4Geeks: https://www.geeksforgeeks.org/artificial-intelligence/how-does-an-agent-formulate-a-problem/#
            5. A Medium Article: https://medium.com/kredo-ai-engineering/search-algorithms-part-1-problem-formulation-and-searching-for-solutions-28f722b7a1a6 
            6. Summary Slides: https://web.pdx.edu/~arhodes/ai7.pdf 
        2. My solution simplifies and customizes these concepts to suit the problem.
        3. Why is this important?
            1. Once you understand this approach you can solve many many problems very quickly and it also makes it easy to communicate about problem solving. I recommend doing 5 problems using this framework to get the hang of it.
    2. I used recursion because its fast to implement especially during a timed interview. But it's not necessary of course.

        

    
