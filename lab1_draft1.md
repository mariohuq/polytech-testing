# Кондраев Дмитрий 5130201/10101 --- Тестирование на столе {-}

На вход подаются два целых числа $w$, $h$ ($2 ⩽ w, h ⩽ 20$) разделенные пробелом, далее в $h$ строках
по $w$ цифр 0 или 1, разделенные пробелом.
Цифры описывают прямоугольную сетку $w × h$, где 0 обозначает свободное поле, а 1 --- непроходимую стену.

Программа находит длину кратчайшего пути из верхнего левого поля сетки $(0, 0)$ в правое нижнее $(w-1, h-1)$ по свободным полям,
при условии, что можно разрушить не более одной стены по пути (сделав его свободным).
Длина пути --- количество пройденных полей, включая начальное поле и конечное.
Из текущего поля можно пройти в соседнее по горизонтали или вертикали свободное поле, проход по диагонали не допускается.

Если такого пути не существует, то выводится сообщение об ошибке.

Для решения задачи запускаются 2 поиска в ширину: из начального и конечного поля,
причем для каждого поля, включая стены, запоминается длина пути до него.
Минимальная сумма длин путей из начального и конечного поля --- искомое число, за вычетом единицы.

## Код программы {-}

```python
from collections import deque


DIRECTIONS = [
    (1, 0),
    (0, 1),
    (-1, 0),
    (0, -1)
]

Walls = list[list[bool]]
Distances = list[list[int | type(None)]]


def bfs_to_all(start_pos: (int, int), walls: Walls) -> Distances:
    """
    Подсчитывает для каждой клетки поля
    минимальное количество ходов из `start_pos`
    """
    rows = len(walls)
    columns = len(walls[0])
    result = [[None] * columns for _ in range(rows)]
    queue = deque([start_pos])
    start_row, start_col = start_pos
    result[start_row][start_col] = 0
    while queue:
        row, column = queue.popleft()
        if walls[row][column]:
            continue
        neighbours = [(r, c)
            for dx, dy in DIRECTIONS
            if 0 <= (r := row + dx) < rows
            and 0 <= (c := column + dy) < columns
            and result[r][c] is None
        ]
        next_cost = result[row][column] + 1
        for row_next, column_next in neighbours:
            result[row_next][column_next] = next_cost
        queue.extend(neighbours)
    return result


def solution(walls: Walls) -> int:
    start_pos = (0, 0)
    exit_pos = (len(walls) - 1, len(walls[0]) - 1)
    zipped = zip(bfs_to_all(start_pos, walls), bfs_to_all(exit_pos, walls))
    try:
        return 1 + min(a + b
            for rows in zipped
            for a, b in zip(*rows)
            if a is not None and b is not None)
    except ValueError:
        return None


def main():
    w, h = (int(x) for x in input().split())
    walls = []
    for _ in range(h):
        inp = input().split()
        walls.append([int(x) for x in inp][:w])
    answer = solution(walls)
    if answer is None:
        print("no such a way exists")
        return
    print(answer)


if __name__ == '__main__':
    main()
```