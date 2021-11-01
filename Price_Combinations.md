```python
import pandas as pd
import numpy as np
import collections
```

```python
num_boards = 10
price_csv_file_path = 'price_source_data.csv'
max_price = int(num_boards * 2000)
```

```python
df = pd.read_csv(price_csv_file_path)
df = df[[col for col in df.columns if 'Price' in col or 'Board' in col]].copy()
df = df[~df.isnull().any(axis=1)]
df.columns = [item for sublist in [
    ['board_{}'.format(i), 'price_{}'.format(i)] for i in range(1, num_boards + 1)
    ] for item in sublist]
for col in [c for c in df if 'price' in c]:
    df[col] = df[col].str.slice(1).str.replace(",", ".").astype(float)
    df[col] = np.where(df[col] < num_boards*2, df[col]*1000, df[col]).astype(int)
df
```

```python
class Player:
    def __init__(self, name, price, board):
        self.name = name
        self.price = price
        self.board = board
```

```python
# Set up "graph"
d = {}
for board in range(1, num_boards + 1):
    d[board] = collections.defaultdict(list)
    board_col = 'board_{}'.format(board)
    price_col = 'price_{}'.format(board)
    for ix, row in df[[board_col, price_col]].iterrows():
        # Rules to simplify problem
        if board == 1 or board == 10:
            if row[price_col] < 2600:
                continue
        elif board == 2 or board == 9:
            if row[price_col] < 2200:
                continue
        elif board == 3 or board == 8:
            if row[price_col] < 2100:
                continue
        else:
            if row[price_col] > 2000:
                continue
        d[board][row[price_col]].append(Player(row[board_col], row[price_col], board))
```

```python
# Solve the problem
solutions = []
memo = collections.defaultdict(list)

def dfs(current_solution, board, price):
    # End condition
    if len(current_solution) == num_boards:
        if price == max_price:
            solutions.append(current_solution)
            csum = 0
            for b in range(num_boards - 1):
                csum += current_solution[b]
                memo[(b + 1, csum)].append(current_solution[b + 1:])
            return True
        return False
    
    # Use memo when possible
    if (board, price) in memo:
        for suffix in memo[(board, price)]:
            solutions.append(current_solution + suffix)
        return bool(memo[(board, price)])
    
    # Normal search
    result = False
    for new_price in d[board + 1]:
        sum_price = price + new_price
        if sum_price <= max_price:
            result = dfs(current_solution+[new_price], board + 1, price + new_price) or result
    
    # If search was unsuccessful, remember that
    if not result:
        memo[(board, price)] = []

    return result

dfs([], 0, 0)
```

```python
# Get number of unique solutions
csum = 0
for i, solution in enumerate(solutions):
    unique = 1
    for j, price in enumerate(solution):
        unique *= len(d[j+1][price])
    csum += unique
csum
```

```python
# Get random $20,000 team
random_team = np.random.randint(len(solutions))
print([(d[i + 1][price][0].name, d[i + 1][price][0].price/1000) 
       for i, price in enumerate(solutions[random_team])])
```

```python
# Get total number of possible teams
total = 1
for b in d:
    total *= len(d[b])
total
```

```python

```
